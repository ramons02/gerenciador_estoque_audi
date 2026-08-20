# Data Model: HU-007 - Lançamento Rápido de Venda (Balcão/Entrega)

**HU**: HU-007 | **Feature**: Lançamento Rápido de Venda | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Requisitos vinculados**: RF-020, RF-021, RF-021-A, RF-031, RF-052, RNF-005, RNF-006, RNF-007, RNF-008, RGN-002

Entidades tocadas pela feature (somente as que ela usa ou altera). Nomes canônicos do projeto, consistentes entre as 20 features. A estrutura de venda é a base comum das HU-008 (entrega), HU-009 (troca) e HU-010 (vasilhame novo); cada uma documenta apenas o que adiciona.

---

## tab_venda

Lançamento de venda (cabeçalho).

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGINT | sim | PK | Sequência `seq_venda` |
| data_hora | TIMESTAMP | sim | - | Servidor, fuso Brasília (V7); CT-006 |
| tipo | VARCHAR(20) | sim | - | BALCAO ou ENTREGA (RF-020) |
| forma_pagamento | VARCHAR(20) | sim | - | DINHEIRO, PIX ou CARTAO (RF-021; RGN-002 sem Fiado) |
| taxa_entrega | NUMERIC(12,2) | - | - | Preenchida quando tipo ENTREGA (HU-008) |
| acrescimo_cartao | NUMERIC(12,2) | - | - | Preenchido quando forma CARTAO e carga do produto é Gás (RF-021-A) |
| total | NUMERIC(12,2) | sim | - | Calculado no Service (CT-002) |
| status | VARCHAR(20) | sim | - | ATIVA ou CANCELADA (RNF-007) |
| id_usuario | BIGINT | sim | FK → tab_usuario(id) | Responsável pelo lançamento (RNF-006; CT-006) |
| id_cliente | BIGINT | opcional | FK → tab_cliente(id) | Obrigatório só sem devolução de vazio (HU-010) |
| ativo | BOOLEAN | sim | - | DEFAULT TRUE; nunca DELETE |

*Situação atual (V3/V4)*: `tab_venda` existe com `produto_id`, `quantidade`, `valor_unitario`, `status` default 'CONCLUIDA', `usuario` (VARCHAR) e `vasilhame_novo` (BOOLEAN). Conciliação em V10+: migrar para `tab_venda` + `tab_venda_item` (multi-item), `id_usuario`, status ATIVA/CANCELADA e campos de taxa/acréscimo; `tab_movimentacao_estoque` para o rastro.

---

## tab_venda_item

Itens da venda (nova tabela, migration V10+).

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGINT | sim | PK | Sequência `seq_venda_item` |
| id_venda | BIGINT | sim | FK → tab_venda(id) | |
| id_produto | BIGINT | sim | FK → tab_produto(id) | Produto = carga + vasilhame (RDN-001) |
| quantidade | INTEGER | sim | - | > 0; validada contra cheios (RF-031) |
| preco_unitario | NUMERIC(12,2) | sim | - | Preço de venda vigente, sem acréscimo |
| preco_casco | NUMERIC(12,2) | opcional | - | Preenchido em vasilhame novo (HU-010, RGN-010) |
| tipo_item | VARCHAR(20) | sim | - | CHEIO (troca/balcão) ou CASCO_NOVO (HU-010) |

---

## tab_movimentacao_estoque

Rastro de auditoria (nova tabela, migration V10+; ver [data-model HU-006](HU-006-Chegada-Caminhao/data-model.md) para o modelo completo).

- HU-007 gera um registro SAIDA_CHEIO por item (quantidade = unidades vendidas), com `id_referencia` = id da venda e `id_usuario` do lançamento.
- HU-009 adiciona ENTRADA_VAZIO na mesma transação (troca); HU-010 adiciona EM_RUA (vasilhame novo).

---

## tab_estoque (saldos)

| Campo | Tipo | Obrigatório | Observação |
|---|---|---|---|
| id_produto | BIGINT | sim | PK, FK → tab_produto(id) |
| qtd_cheios | INTEGER | sim | Baixada na venda; nunca negativa (RDN-005, RF-031) |
| qtd_vazios / qtd_em_rua | INTEGER | sim | Não alteradas pela HU-007 (balcão sem troca) |

*Situação atual (V3)*: saldos em `tab_produto` (`estoque_cheios`); ver nota de conciliação em [data-model HU-006](HU-006-Chegada-Caminhao/data-model.md).

---

## tab_configuracao (leitura)

Chaves usadas pela HU-007 (existentes na V9):

| Chave | Valor | Uso |
|---|---|---|
| pagamento_DINHEIRO | true/false | Habilita Dinheiro (RF-052) |
| pagamento_PIX | true/false | Habilita PIX (RF-052) |
| pagamento_CARTAO | true/false | Habilita Cartão (RF-052) |
| acrescimo_cartao | R$ por unidade | Aplicado quando CARTAO e carga do produto é Gás (RF-021-A) |

---

## tab_produto (leitura, lock)

Consultado com `@Lock(PESSIMISTIC_WRITE)` para validar saldo dentro da transação (RNF-008); `preco_venda` é a base do total (RF-002). Estrutura em [data-model HU-006](HU-006-Chegada-Caminhao/data-model.md).

---

## tab_usuario (nova)

| Campo | Tipo | Obrigatório | Observação |
|---|---|---|---|
| id | BIGINT | sim | PK; sequência `seq_usuario` |
| nome | VARCHAR(100) | sim | |
| login | VARCHAR(50) | sim | UNIQUE |
| papel | VARCHAR(20) | sim | VENDEDOR ou ADMIN (RNF-006) |
| senha_hash | VARCHAR(200) | sim | Hash, nunca texto puro |
| ativo | BOOLEAN | sim | |

*Situação atual*: não existe; a venda usa `usuario` VARCHAR. Migration V10+ cria a tabela (RNF-006).

---

## Regras de validação derivadas dos requisitos

- Campos obrigatórios: `id_produto`, `quantidade > 0`, `tipo` (BALCAO/ENTREGA), `forma_pagamento` (CT-001; RF-020/RF-021).
- `forma_pagamento` deve estar habilitada em `tab_configuracao` no momento da confirmação (CT-003; RF-052) e não pode ser FIADO (RGN-002).
- `quantidade <= qtd_cheios` do produto, validada sob lock antes de gravar (CT-004; RF-031; RDN-005; RNF-008). Limite exato permitido (spec.md Edge Cases).
- `total = soma(quantidade × preco_venda)` com acréscimo do cartão por unidade quando CARTAO e carga do produto é Gás (CT-002, CT-003-A; RF-021-A). Produtos de carga Água usam preço normal em qualquer forma de pagamento.
- `id_usuario` sempre preenchido (CT-006; RNF-006); `data_hora` do servidor em fuso Brasília.
- Estoque nunca negativo após a operação (RDN-005); movimentação e saldo no mesmo commit (CONVENTIONS §5.1).
- Regra de negócio sem RF correspondente não pode ser codificada; se necessária, registrar em REQUISITOS.md na mesma entrega (Constituição §X.2).

## Transições de estado

- `tab_venda.status`: ATIVA → CANCELADA (RNF-007; RGN-007). O cancelamento reverte estoque e caixa automaticamente com rastro (quem, quando, motivo - Constituição §XIII.2) e mantém o registro histórico. A HU-007 lança a venda em ATIVA; o cancelamento é comportamento da HU-020.
- `tab_venda_item` e `tab_movimentacao_estoque` não são alterados por cancelamento (rastro imutável); a reversão é registrada como nova movimentação com tipo de estorno.
- Saldo `qtd_cheios`: decrementado na confirmação, sem estado intermediário visível (RNF-005).
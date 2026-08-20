# Data Model: HU-006 - Registro de Chegada de Caminhão (Entrada)

**HU**: HU-006 | **Feature**: Chegada de Caminhão (Carregamento) | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Requisitos vinculados**: RF-010, RF-011, RF-012, RDN-003, RDN-009, RNF-005, RNF-007, RNF-008

Entidades tocadas pela feature (somente as que ela usa ou altera). Nomes canônicos do projeto, consistentes entre as 20 features. Onde o schema atual (migrations V1 a V9) diverge do canônico, há nota de conciliação para a migration V10+.

---

## tab_carregamento

Registro da chegada de caminhão (uma entrada = um caminhão).

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGINT | sim | PK | Sequência `seq_carregamento` |
| id_fornecedor | BIGINT | sim | FK → tab_fornecedor(id) | Distribuidora da carga |
| data_hora | TIMESTAMP | sim | - | Horário de Brasília (V7) |
| ativo | BOOLEAN | sim | - | DEFAULT TRUE; estorno lógico, nunca DELETE |

*Situação atual (V3)*: `tab_carregamento` existe com `fornecedor_id` e `criado_em`. Conciliação em V10+: renomear para `id_fornecedor`/`data_hora` ou manter mapeamento na entidade JPA. Não bloqueia a implementação.

---

## tab_carregamento_item

Item da chegada: um carregamento pode trazer cheios de um ou mais produtos, com devolução de vazios por produto.

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGINT | sim | PK | Sequência `seq_carregamento_item` |
| id_carregamento | BIGINT | sim | FK → tab_carregamento(id) | |
| id_produto | BIGINT | sim | FK → tab_produto(id) | Produto = carga + vasilhame (RDN-001) |
| qtd_cheios_entraram | INTEGER | sim | - | > 0 (divisão por zero do valor unitário) |
| qtd_vazios_sairam | INTEGER | sim | - | >= 0; devolvidos à distribuidora |
| custo_total | NUMERIC(12,2) | sim | - | Custo da carga do item |
| valor_unitario | NUMERIC(12,2) | sim | - | Calculado: custo_total ÷ qtd_cheios_entraram, 2 casas |
| ativo | BOOLEAN | sim | - | DEFAULT TRUE |

*Situação atual (V3)*: `tab_carregamento_item` existe com `quantidade_cheios`, `vazios_devolvidos` e `custo_unitario`. Conciliação em V10+ para os nomes canônicos.

---

## tab_estoque

Saldo por estado do produto (canônico; alvo de conciliação).

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id_produto | BIGINT | sim | PK, FK → tab_produto(id) | Um registro por produto |
| qtd_cheios | INTEGER | sim | - | Prontos para venda; nunca negativo (RDN-005) |
| qtd_vazios | INTEGER | sim | - | Pátio; nunca negativo (RDN-003, RDN-005) |
| qtd_em_rua | INTEGER | sim | - | Com clientes (comodato); não afetada por HU-006 |
| limite_minimo | INTEGER | sim | - | Alerta de estoque baixo (RF-003) |

*Situação atual (V3)*: saldos `estoque_cheios`/`estoque_vazios` e `limite_minimo` vivem em `tab_produto`; "em rua" é derivado de `tab_cliente_vasilhame`. Conciliação em V10+: criar `tab_estoque` e migrar os saldos, ou manter as colunas em `tab_produto` e documentar `tab_estoque` como visão. Decisão a registrar na implementação; HU-006 usa apenas `qtd_cheios` e `qtd_vazios`.

---

## tab_movimentacao_estoque

Rastro de auditoria de toda alteração de saldo (nova tabela, migration V10+).

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGINT | sim | PK | Sequência `seq_movimentacao_estoque` |
| id_produto | BIGINT | sim | FK → tab_produto(id) | |
| tipo | VARCHAR(30) | sim | - | ENTRADA_CHEIO, SAIDA_VAZIO (HU-006); SAIDA_CHEIO, ENTRADA_VAZIO, EM_RUA, BAIXA_EM_RUA (vendas HU-007/009/010) |
| quantidade | INTEGER | sim | - | Sinal implícito no tipo |
| id_referencia | BIGINT | sim | - | Id do documento de origem (carregamento ou venda) |
| data_hora | TIMESTAMP | sim | - | Horário de Brasília |
| id_usuario | BIGINT | opcional | FK → tab_usuario(id) | Quem confirmou (RNF-006) |

---

## tab_produto (leitura)

Consultado para validar saldos e para leitura de custo médio.

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGINT | sim | PK | Sequência `seq_produto` |
| id_carga | BIGINT | sim | FK → tab_carga(id) | Gás ou Água |
| id_vasilhame | BIGINT | sim | FK → tab_vasilhame(id) | P13, P45, Galão 20L |
| preco_custo / preco_venda | NUMERIC(12,2) | sim | - | RF-002 |
| custo_medio | NUMERIC(12,2) | - | - | Recalculado pela HU-006 (RF-012) |
| ativo | BOOLEAN | sim | - | |

*Situação atual (V1)*: `tab_produto` existe com `carga_id`, `vasilhame_id`, `preco_custo`, `preco_venda`, `limite_minimo` e UNIQUE(carga_id, vasilhame_id); `custo_medio` ainda não existe - migration V10+ o adiciona.

---

## tab_configuracao (leitura)

| Campo | Tipo | Obrigatório | Observação |
|---|---|---|---|
| chave | VARCHAR(50) | sim | UNIQUE; HU-006 não a usa diretamente |
| valor | VARCHAR(100) | sim | |

---

## Regras de validação derivadas dos requisitos

- `qtd_cheios_entraram > 0` (evita divisão por zero do valor unitário - spec.md Edge Cases; RF-010).
- `qtd_vazios_sairam >= 0` e `qtd_vazios_sairam <= qtd_vazios` do pátio no momento da confirmação (RDN-003; CT-002). Limite exato permitido (spec.md Edge Cases).
- `custo_total >= 0`; `valor_unitario` sempre igual a `custo_total ÷ qtd_cheios_entraram`, arredondado a 2 casas (CT-001, SC-004).
- `id_fornecedor` e `id_produto` devem existir e estar ativos (RF-010; assumido cadastrado, RF-005).
- Estoque de cheios e vazios nunca negativo após a operação (RDN-005; sob lock - RNF-008).
- Regra de negócio sem RF correspondente não pode ser codificada; se necessária, registrar em REQUISITOS.md na mesma entrega (Constituição §X.2).

## Transições de estado

- `tab_carregamento.ativo` / `tab_carregamento_item.ativo`: ATIVO → INATIVO (estorno) - registrado em HU-006 apenas como edge case (spec.md): "registro confirmado por engano não pode ser apagado, apenas estornado/cancelado com rastro de auditoria" (RNF-007). O estorno completo (reverter estoque e movimentações) é implementado quando o cancelamento for tratado; a estrutura de dados já o suporta via `tab_movimentacao_estoque` com `id_referencia`.
- Saldos de `tab_estoque`: cheios +N e vazios -N imediatamente após a confirmação (RF-011, CT-003), sem estado intermediário visível.
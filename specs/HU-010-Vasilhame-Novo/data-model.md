# Data Model: HU-010 - Venda de Vasilhame Novo (Casco + Carga)

**HU**: HU-010 | **Feature**: Venda de Vasilhame Novo | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Requisitos vinculados**: RF-004, RF-024, RF-026, RF-028, RDN-005, RDN-008, RNF-005, RNF-007, RNF-008, RGN-010

Entidades tocadas pela feature (somente as que ela usa ou altera). Estrutura base de venda em [data-model HU-007](../HU-007-Lancamento-Venda/data-model.md); esta HU documenta apenas o que o vasilhame novo adiciona.

---

## tab_venda (uso)

| Campo | Tipo | Obrigatório | Observação |
|---|---|---|---|
| id_cliente | BIGINT | quando item CASCO_NOVO | FK → tab_cliente(id); obrigatório sem devolução de vazio (CT-004; RF-028) |
| total | NUMERIC(12,2) | sim | Soma dos itens com preço casco + carga (CT-002; RGN-010) |

---

## tab_venda_item (uso)

| Campo | Tipo | Obrigatório | Observação |
|---|---|---|---|
| tipo_item | VARCHAR(20) | sim | CASCO_NOVO para vasilhame novo (RF-024); CHEIO para demais |
| preco_unitario | NUMERIC(12,2) | sim | preco_casco + preco_venda da carga, calculado no Service (CT-002) |
| preco_casco | NUMERIC(12,2) | quando tipo_item CASCO_NOVO | Valor do casco aplicado, gravado para conferência (RGN-010) |

---

## tab_estoque (alteração)

| Campo | Tipo | Obrigatório | Efeito no vasilhame novo (por unidade) |
|---|---|---|---|
| qtd_cheios | INTEGER | sim | -1 (RF-026); nunca negativa (RDN-005) |
| qtd_vazios | INTEGER | sim | Inalterada (sem devolução de vazio) |
| qtd_em_rua | INTEGER | sim | +1 (RF-026; RDN-008) |

*Situação atual (V3)*: saldos em `tab_produto`; ver nota de conciliação em [data-model HU-006](../HU-006-Chegada-Caminhao/data-model.md).

---

## tab_cliente_vasilhame (em rua por cliente)

Tabela existente (V3), usada como controle de comodato (RF-028).

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGINT | sim | PK | Sequência `seq_cliente_vasilhame` |
| cliente_id | BIGINT | sim | FK → tab_cliente(id) | |
| vasilhame_id | BIGINT | sim | FK → tab_vasilhame(id) | Tipo de casco (P13 etc.) |
| quantidade | INTEGER | sim | - | +N na venda de vasilhame novo; -N na devolução (HU-011) |
| ativo | BOOLEAN | sim | - | |
| UNIQUE (cliente_id, vasilhame_id) | - | - | - | Um agregado por cliente e tipo de casco |

---

## tab_vasilhame (leitura)

| Campo | Tipo | Obrigatório | Observação |
|---|---|---|---|
| id | BIGINT | sim | PK |
| nome | VARCHAR(50) | sim | UNIQUE |
| preco_casco | NUMERIC(12,2) | sim | Preço do casco configurado à parte (RGN-010); base do cálculo CASCO_NOVO |

*Situação atual (V3)*: `preco_casco` já existe com DEFAULT 0.

---

## tab_movimentacao_estoque (uso)

O vasilhame novo gera, no mesmo commit, um registro por item (RF-026):

| Registro | Tipo | Quantidade | id_referencia |
|---|---|---|---|
| 1 | SAIDA_CHEIO | N | id da venda |
| 2 | EM_RUA | N | id da venda |

Modelo completo da tabela em [data-model HU-006](../HU-006-Chegada-Caminhao/data-model.md). A baixa do em rua na devolução (RF-028) gera movimentação BAIXA_EM_RUA com referência à devolução (HU-011).

---

## tab_cliente (leitura)

Validação da existência do cliente (RF-004). Estrutura em [data-model HU-004](../HU-004-Cadastro-Clientes/data-model.md) quando existir; campos: id, nome, telefone, endereco, ativo (V3/V6).

---

## Regras de validação derivadas dos requisitos

- Item `tipo_item = CASCO_NOVO` exige `id_cliente` (CT-004; RF-028). Sem cliente: "Informe o cliente para venda de vasilhame novo." (422).
- `preco_unitario = preco_casco + preco_venda`, calculado no Service com os valores vigentes (CT-002; RGN-010). `preco_casco` gravado no item.
- `quantidade <= qtd_cheios`, validada sob lock (RF-031; RDN-005). Limite exato permitido.
- Cliente deve existir e estar ativo (RF-004); cadastro rápido permitido antes de concluir (spec.md Edge Cases).
- Venda de vasilhame novo de N unidades: N cheios baixados e N vasilhames em rua registrados para o cliente (CT-003; spec.md Edge Cases).
- Devolução posterior: baixa no registro do cliente correto (RF-028; spec.md Edge Cases), tratada na HU-011.
- Regra de negócio sem RF correspondente não pode ser codificada; se necessária, registrar em REQUISITOS.md na mesma entrega (Constituição §X.2).

## Transições de estado

- Saldos de `tab_estoque`: `qtd_cheios` -N e `qtd_em_rua` +N aplicados juntos ou nenhum (CT-003; RNF-005). Sem estado parcial visível.
- Vasilhame: estado Cheio → Em rua (RDN-008; RDN-002: todo vasilhame em exatamente um estado).
- `tab_cliente_vasilhame.quantidade`: +N na venda; -N na devolução do vazio (HU-011); nunca negativa (RDN-005).
- `tab_venda.status`: ATIVA → CANCELADA (RNF-007, RGN-007), comportamento da HU-020. A reversão do vasilhame novo é o espelho: +N cheios, -N em rua, com novo rastro.
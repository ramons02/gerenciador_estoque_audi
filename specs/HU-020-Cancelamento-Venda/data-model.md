# Data Model - Fase 1: Cancelamento/Estorno de Venda

**HU**: HU-020 - Cancelamento/Estorno de Venda
**Fase**: 1 - Modelo de dados
**Data**: 2026-08-20

## Visão geral

A feature adiciona os campos de cancelamento em `tab_venda` (migration Flyway `V<N>__adiciona_campos_cancelamento_tab_venda.sql`, CONVENTIONS §8) e grava movimentações de estorno em `tab_movimentacao_estoque`. Nenhuma venda é apagada fisicamente (RNF-007); o cancelamento é uma transição de estado com reversão atômica (RGN-007).

## Tabelas alteradas

### tab_venda (campos novos de cancelamento)

| Coluna | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| id | bigint (PK, seq_venda) | sim | Identificador da venda |
| data_hora | timestamptz | sim | Data/hora do lançamento |
| tipo | varchar(10) | sim | BALCAO ou ENTREGA (RF-020) |
| forma_pagamento | varchar(10) | sim | DINHEIRO, PIX ou CARTAO (RF-021, RGN-002) |
| taxa_entrega | numeric(12,2) | nao | Taxa de entrega (RF-022, RGN-001) |
| acrescimo_cartao | numeric(12,2) | nao | Acréscimo de cartão (RF-021-A, somente carga Gás) |
| total | numeric(12,2) | sim | Total da venda |
| status | varchar(10) | sim | ATIVA ou CANCELADA (RNF-007) |
| id_usuario | bigint (FK tab_usuario) | sim | Usuário do lançamento (RNF-006) |
| id_cliente | bigint (FK tab_cliente) | nao | Cliente, opcional (RF-004) |
| motivo_cancelamento | varchar(200) | sim (no cancelamento) | Motivo do cancelamento (Constituição §XIII.2, FR-004) |
| data_hora_cancelamento | timestamptz | sim (no cancelamento) | Quando o cancelamento ocorreu (FR-004, CT-003) |
| id_usuario_cancelamento | bigint (FK tab_usuario) | sim (no cancelamento) | Quem cancelou (FR-004, CT-003) |

Regras:

- `motivo_cancelamento` e `id_usuario_cancelamento` são obrigatórios quando `status = CANCELADA` (Constituição §XIII.2, RNF-006/007).
- A venda CANCELADA permanece no histórico; nenhum campo do lançamento original é alterado ou apagado (RNF-007).

### tab_venda_item (consultada)

Itens com `id_produto`, `quantidade` e `valor_unitario`: base da reversão de estoque e do estorno de valor (FR-001/FR-002).

### tab_movimentacao_estoque (novos tipos)

| Coluna | Tipo | Descrição |
|---|---|---|
| id | bigint (PK, seq_movimentacao_estoque) | Identificador da movimentação |
| data_hora | timestamptz | Data/hora da movimentação (a do cancelamento) |
| id_produto | bigint (FK tab_produto) | Produto movimentado |
| tipo | varchar(20) | ENTRADA_CHEIO, SAIDA_CHEIO, ENTRADA_VAZIO, SAIDA_VAZIO, ENTRADA_EM_RUA, SAIDA_EM_RUA, ESTORNO_CHEIO, ESTORNO_VAZIO, ESTORNO_EM_RUA |
| quantidade | integer | Quantidade (sempre positiva; o tipo indica o sentido) |
| id_venda_referencia | bigint (FK tab_venda) | Venda de origem, preenchido em vendas e estornos |
| id_carregamento_referencia | bigint (FK tab_carregamento) | Carregamento de origem, quando aplicável |
| id_usuario | bigint (FK tab_usuario) | Usuário responsável (RNF-006) |
| observacao | varchar(200) | Livre, ex.: "Estorno por cancelamento da venda X" |

Regras de reversão por tipo de venda (FR-001, Edge Case do spec):

| Tipo de venda original | Reversão no estoque | Movimentações gravadas |
|---|---|---|
| Venda com troca (RF-023) | cheio volta ao estoque de cheios; vazio sai do pátio | ESTORNO_CHEIO (+), ESTORNO_VAZIO (-) |
| Venda sem devolução de vazio (vasilhame novo/avulsa, RF-024/RF-026) | casco "em rua" volta ao estoque de cheios | ESTORNO_EM_RUA (-), ESTORNO_CHEIO (+) |
| Venda avulsa sem casco em rua (se existir, conforme HU-007) | cheio volta ao estoque de cheios | ESTORNO_CHEIO (+) |

## Regras de validação (derivadas de requisitos)

| Regra | Fonte |
|---|---|
| Reversão de estoque e caixa na MESMA transação do cancelamento | RGN-007, RNF-005, CONVENTIONS §5.1 |
| Estoque nunca negativo em nenhum estado após a reversão | RDN-005, Edge Case do spec |
| Cancelamento de venda já cancelada é recusado | FR-006, Edge Case do spec |
| Motivo obrigatório; quem e quando registrados | FR-004, RNF-006, §XIII.2 |
| Estornos não entram nas colunas (+) Entradas / (-) Vendas do balanço | RF-043, FR-005 |
| Relatórios e dashboard somam apenas vendas ATIVA | CT-004, RGN-007 |
| Forma de pagamento nunca é Fiado | RGN-002 |

## Transições de estado

- Venda: ATIVA -> CANCELADA (irreversível; cancelamento de cancelada é recusado, FR-006).
- Estoque: a reversão move vasilhames entre os três estados sem sair da invariante RDN-002; o saldo materializado em `tab_estoque` é atualizado no mesmo commit.
- Caixa: derivado das vendas ATIVA (RGN-008), o estorno é consequência do status CANCELADA na mesma transação (Decision 4 da research).
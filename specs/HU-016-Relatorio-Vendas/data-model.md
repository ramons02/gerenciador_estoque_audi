# Data Model - Fase 1: Relatório de Vendas (Diário/Mensal)

**HU**: HU-016 - Relatório de Vendas (Diário/Mensal)
**Fase**: 1 - Modelo de dados
**Data**: 2026-08-20

## Visão geral

Feature de LEITURA: não cria tabela nova. Consolida vendas do período (RF-040) a partir das tabelas canônicas `tab_venda`, `tab_venda_item` e `tab_produto`. As colunas do relatório seguem CONVENTIONS §10 (RF-041). Os campos de venda usam os mesmos nomes definidos nas features de venda (HU-007 a HU-010) e no cancelamento (HU-020).

## Tabelas consultadas

### tab_venda (Relatório de Vendas, RF-041)

| Coluna | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| id | bigint (PK, seq_venda) | sim | Identificador da venda |
| data_hora | timestamptz | sim | Data/hora do lançamento (coluna do filtro de período) |
| tipo | varchar(10) | sim | BALCAO ou ENTREGA (RF-020) |
| forma_pagamento | varchar(10) | sim | DINHEIRO, PIX ou CARTAO (RF-021, RGN-002) |
| taxa_entrega | numeric(12,2) | nao | Taxa aplicada quando tipo = ENTREGA (RF-022, RGN-001) |
| acrescimo_cartao | numeric(12,2) | nao | Acréscimo aplicado quando forma_pagamento = CARTAO e carga do produto é Gás (RF-021-A) |
| total | numeric(12,2) | sim | Total da venda com taxa/acréscimo (RF-022; RF-021-A somente carga Gás) |
| status | varchar(10) | sim | ATIVA ou CANCELADA (RNF-007, HU-020) |
| id_usuario | bigint (FK tab_usuario) | sim | Usuário responsável (RNF-006) |
| id_cliente | bigint (FK tab_cliente) | nao | Cliente, opcional (RF-004) |
| motivo_cancelamento | varchar(200) | nao | Preenchido no cancelamento (HU-020) |
| data_hora_cancelamento | timestamptz | nao | Preenchido no cancelamento (HU-020) |
| id_usuario_cancelamento | bigint (FK tab_usuario) | nao | Preenchido no cancelamento (HU-020) |

### tab_venda_item (RF-041)

| Coluna | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| id | bigint (PK, seq_venda_item) | sim | Identificador do item |
| id_venda | bigint (FK tab_venda) | sim | Venda do item |
| id_produto | bigint (FK tab_produto) | sim | Produto (carga + vasilhame, RF-001) |
| quantidade | integer | sim | Qtd de vasilhames da linha |
| valor_unitario | numeric(12,2) | sim | Valor unitário da linha (com acréscimo de cartão quando aplicável: carga Gás com Cartão) |

### tab_produto (consultada)

Nome do produto (carga + vasilhame, RF-001), exibido na coluna Produto.

## Regras de validação (derivadas de requisitos)

| Regra | Fonte |
|---|---|
| Filtro por período via `data_hora` de `tab_venda` (Hoje, 7 dias, Mês, personalizado) | RF-040, CT-002 |
| Somente vendas `status = ATIVA` entram no relatório; CANCELADA excluídas | FR-005, RGN-007 |
| Colunas exatas conforme CONVENTIONS §10 | RF-041, CT-001 |
| Forma de pagamento nunca é Fiado (somente DINHEIRO, PIX, CARTAO) | RGN-002 |
| Cartão é forma única; crédito e débito não se distinguem na venda | RF-021, RF-051 |
| Período personalizado com fim anterior ao início é rejeitado | Edge Case do spec |
| Período sem vendas gera relatório com zero linhas, sem erro | Edge Case do spec, SC-001 |

## Transições de estado

- Relatório é leitura pura: nenhum estado de negócio muda (RF-040 a RF-044).
- O estado dos registros usados (venda ATIVA/CANCELADA) é definido pela feature de cancelamento (HU-020, RGN-007); a venda CANCELADA desaparece do relatório na consulta seguinte.
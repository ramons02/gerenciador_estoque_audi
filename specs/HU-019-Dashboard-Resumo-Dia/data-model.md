# Data Model - Fase 1: Dashboard (Resumo do Dia)

**HU**: HU-019 - Dashboard (Resumo do Dia)
**Fase**: 1 - Modelo de dados
**Data**: 2026-08-20

## Visão geral

Feature de LEITURA: não cria tabela nova. Agrega as vendas do dia (RF-050 a RF-052) a partir de `tab_venda` e `tab_venda_item` e os alertas (RF-053) a partir de `tab_estoque` e `tab_produto`. Nomes consistentes com as features de venda (HU-007 a HU-010), cancelamento (HU-020), painel de estoque (HU-012) e alerta (HU-014).

## Tabelas consultadas

### tab_venda (RF-050, RF-051)

| Coluna | Tipo | Uso no dashboard |
|---|---|---|
| data_hora | timestamptz | Filtro: vendas do dia atual (RF-050) |
| forma_pagamento | varchar(10) | Agrupamento: DINHEIRO, PIX, CARTAO (RF-051, RF-021) |
| total | numeric(12,2) | Soma do total faturado e por forma de pagamento |
| status | varchar(10) | Filtro: somente ATIVA (FR-005, RGN-007) |

### tab_venda_item (RF-052)

| Coluna | Tipo | Uso no dashboard |
|---|---|---|
| id_produto | bigint (FK tab_produto) | Agrupamento por produto (RF-052, CT-003) |
| quantidade | integer | Soma de unidades vendidas por produto e total geral |

### tab_estoque e tab_produto (RF-053)

| Coluna | Tipo | Uso no dashboard |
|---|---|---|
| tab_estoque.qtd_cheios | integer | Saldo de cheios comparado ao limite (RF-032) |
| tab_estoque.limite_minimo | integer | Critério do alerta (RF-003, RF-053) |
| tab_produto | - | Nome do produto (carga + vasilhame, RF-001) |

## Agregações do resumo do dia

| Campo do dashboard | Cálculo | Fonte |
|---|---|---|
| total_faturado | soma de `total` das vendas ATIVA do dia | RF-050, CT-001 |
| total_dinheiro | soma de `total` com forma_pagamento = DINHEIRO | RF-051 |
| total_pix | soma de `total` com forma_pagamento = PIX | RF-051 |
| total_cartao | soma de `total` com forma_pagamento = CARTAO (crédito + débito somados) | RF-051, CT-002 |
| unidades por produto | soma de `quantidade` por id_produto | RF-052, CT-003 |
| total_geral_unidades | soma de `quantidade` do dia | RF-052, CT-003 |
| alertas_estoque | produtos com `qtd_cheios <= limite_minimo` | RF-053, CT-004 |

## Regras de validação (derivadas de requisitos)

| Regra | Fonte |
|---|---|
| Somente vendas ATIVA entram nos totais; CANCELADA fora (status) | FR-005, RGN-007 |
| Cartão é forma única; crédito e débito não se distinguem na venda | RF-021, RF-051, RGN-002 |
| Forma de pagamento sem movimentação no dia: exibida com valor zero | Edge Case do spec |
| Alerta somente para saldo de cheios no limite mínimo configurado ou abaixo | RF-032, RF-053, RGN-004 |
| Dia sem vendas: totais zerados, sem erro | Edge Case do spec |
| Produto sem vendas no dia não aparece nas unidades vendidas | Edge Case do spec |

## Transições de estado

- Dashboard é leitura pura: nenhum estado de negócio muda.
- Venda ATIVA -> CANCELADA (HU-020) retira a venda das somas do dia na consulta seguinte (FR-005, RGN-007); o alerta de estoque é atualizado conforme o saldo reverter (RGN-004).
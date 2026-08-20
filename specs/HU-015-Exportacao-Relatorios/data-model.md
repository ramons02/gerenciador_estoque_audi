# Data Model - Fase 1: Exportação de Relatórios (Período Personalizado)

**HU**: HU-015 - Exportação de Relatórios (Período Personalizado)
**Fase**: 1 - Modelo de dados
**Data**: 2026-08-20

## Visão geral

Feature de LEITURA: não cria tabela nova. Consolida movimentações (RF-040) a partir das tabelas canônicas do projeto: vendas, carregamentos, movimentações de estoque e saldos materializados. As colunas dos relatórios seguem CONVENTIONS §10 (RF-041/042/043).

## Tabelas consultadas

### tab_venda e tab_venda_item (Relatório de Vendas, RF-041)

| Tabela | Campos usados | Descrição |
|---|---|---|
| tab_venda | data_hora, forma_pagamento, tipo, total, status | Data/Hora, Forma de Pagamento, Tipo (Balcão/Entrega), Total |
| tab_venda_item | id_produto, quantidade, valor_unitario | Produto, Qtd, Valor Unitário |

Regras:

- Filtro por período: `data_hora` dentro do intervalo (Hoje, 7 dias, Mês ou personalizado - RF-040).
- Vendem listadas somente com `status = ATIVA` (vendas canceladas não aparecem no total do período; ver transições).
- Valor Unitário e Total (R$) já refletem acréscimo de cartão (RF-021-A, aplicado somente a produtos de carga Gás) e taxa de entrega (RF-022) aplicados na venda.
- Forma de pagamento nunca é Fiado (RGN-002).

### tab_carregamento e tab_carregamento_item (Relatório de Carregamentos, RF-042)

| Tabela | Campos usados | Descrição |
|---|---|---|
| tab_carregamento | id, data_hora, id_fornecedor, custo_total | Data, Fornecedor, Custo Total |
| tab_carregamento_item | id_carregamento, id_produto, qtd_cheios_entraram, qtd_vazios_sairam | Produto, Qtd Cheios Entraram, Qtd Vazios Saíram |
| tab_fornecedor | nome | Nome do fornecedor/distribuidora (RF-005) |

Regras:

- Filtro por período: `data_hora` dentro do intervalo (RF-040).
- `qtd_vazios_sairam` nunca ultrapassa o saldo de vazios do pátio naquele momento (RDN-003).

### tab_movimentacao_estoque e tab_estoque (Balanço, RF-043)

Derivações por produto, para o período:

| Coluna do relatório | Cálculo | Fonte |
|---|---|---|
| Estoque Inicial | saldoAtual - (entradas - saídas no período) | tab_estoque (saldo atual) + tab_movimentacao_estoque (variação) |
| (+) Entradas | soma de ENTRADA_CHEIO no período | tab_movimentacao_estoque.tipo |
| (-) Vendas | soma de SAIDA_CHEIO no período | tab_movimentacao_estoque.tipo |
| Estoque Final | saldoAtual (qtd_cheios) | tab_estoque |
| Vazios em Pátio | qtd_vazios atual | tab_estoque |

Regras:

- Todo vasilhame está em exatamente um estado (RDN-002); o balanço reflete os três estados de forma consistente.
- Estoque nunca negativo (RDN-005); movimentações nunca apagadas (RNF-007), base do cálculo.

### tab_produto (consultada)

Nome do produto (carga + vasilhame, RF-001) para os três relatórios.

## Regras de validação (resumo, derivadas de requisitos)

| Regra | Fonte |
|---|---|
| Período com fim anterior ao início é rejeitado com mensagem | RF-040, Edge Case do spec |
| Período sem movimentações gera relatório com cabeçalhos e zero linhas | Edge Case do spec, SC-004 |
| Colunas exatas conforme CONVENTIONS §10 | RF-041/042/043, CT-004 |
| Vendas canceladas excluídas da consolidação do período | RGN-007 (status cancelada) |

## Transições de estado

- Relatório é leitura pura: nenhum estado de negócio muda (RF-040 a RF-044).
- O estado dos registros usados (venda ATIVA/CANCELADA) é definido pelas features de venda (HU-013) e permanece estável no período exportado.
# Data Model - Fase 1: Fechamento de Caixa e Balanço de Estoque

**HU**: HU-018 - Fechamento de Caixa e Balanço de Estoque
**Fase**: 1 - Modelo de dados
**Data**: 2026-08-20

## Visão geral

A feature cria a tabela `tab_fechamento_caixa` (migration Flyway `V<N>__cria_tab_fechamento_caixa.sql`, CONVENTIONS §8) e consulta as tabelas canônicas `tab_venda`, `tab_movimentacao_estoque`, `tab_estoque` e `tab_produto`. O balanço de estoque (RF-043) é leitura derivada; o fechamento de caixa (RGN-006) é escrita com transação única.

## Tabela nova

### tab_fechamento_caixa (RGN-006, RF-043)

| Coluna | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| id | bigint (PK, seq_fechamento_caixa) | sim | Identificador do fechamento |
| data | date (UNIQUE) | sim | Data do fechamento; uma única por dia (Decision 1) |
| total_dinheiro | numeric(12,2) | sim | Soma das vendas ATIVA do dia em Dinheiro |
| total_pix | numeric(12,2) | sim | Soma das vendas ATIVA do dia em PIX |
| total_cartao | numeric(12,2) | sim | Soma das vendas ATIVA do dia em Cartão (crédito + débito somados) |
| total_geral | numeric(12,2) | sim | Soma dos três totais |
| status | varchar(10) | sim | ABERTO ou FECHADO (inicia ABERTO no resumo do dia) |
| data_hora_fechamento | timestamptz | nao | Preenchido quando status vira FECHADO |
| id_usuario_fechamento | bigint (FK tab_usuario) | nao | Usuário que concluiu o fechamento (RNF-006) |

Regras:

- `total_geral = total_dinheiro + total_pix + total_cartao`, gravado na transação do fechamento (RGN-006).
- Vendas canceladas (status CANCELADA) não entram nos totais (FR-005, RGN-007).
- Forma de pagamento nunca é Fiado (RGN-002); Cartão é forma única (RF-021, RF-051).

## Tabelas consultadas

### tab_venda (fechamento e balanço)

Somas do dia: vendas `status = ATIVA` com `data_hora` no dia, agrupadas por `forma_pagamento` (DINHEIRO, PIX, CARTAO) e somando `total` (RF-021, RF-021-A, RF-022, RGN-008).

### tab_movimentacao_estoque e tab_estoque (Balanço, RF-043)

Derivações por produto, para o período:

| Coluna do relatório | Cálculo | Fonte |
|---|---|---|
| Estoque Inicial | saldoAtual - (entradas - vendas no período) | tab_estoque (saldo atual) + tab_movimentacao_estoque (variação) |
| (+) Entradas | soma de ENTRADA_CHEIO no período (carregamentos) | tab_movimentacao_estoque.tipo |
| (-) Vendas | soma de SAIDA_CHEIO no período (vendas ATIVA) | tab_movimentacao_estoque.tipo |
| Estoque Final | saldoAtual (qtd_cheios) | tab_estoque |
| Vazios em Pátio | qtd_vazios atual | tab_estoque |

Regras:

- Movimentações de estorno (ESTORNO_CHEIO, ESTORNO_VAZIO, ESTORNO_EM_RUA, HU-020) não entram em Entradas nem Vendas; o Estoque Final (saldo materializado) já as reflete.
- Todo vasilhame está em exatamente um estado (RDN-002); estoque nunca negativo (RDN-005); movimentações nunca apagadas (RNF-007).
- Estoque inicial do período calculado com base nas movimentações anteriores ao período (CT-004).

### tab_produto (consultada)

Nome do produto (carga + vasilhame, RF-001) para as linhas do balanço.

## Regras de validação (derivadas de requisitos)

| Regra | Fonte |
|---|---|
| Fechamento só conclui com todas as vendas do dia conciliadas | RGN-006, CT-002 |
| Um único fechamento por data; fechamento duplicado é rejeitado | RGN-006, Decision 1 |
| Colunas exatas do balanço conforme CONVENTIONS §10 | RF-043, CT-001 |
| Vendas canceladas não entram nas vendas (-) do balanço nem nos totais do dia | FR-005, RGN-007 |
| Período sem movimentações: balanço com produtos cadastrados e valores zerados | Edge Case do spec |
| Produto que não existia no início do período: estoque inicial zero | Edge Case do spec |

## Transições de estado

- `tab_fechamento_caixa.status`: ABERTO -> FECHADO, somente após conciliação das vendas do dia (RGN-006). Não há transição de volta (RNF-007: sem apagar histórico).
- Venda ATIVA -> CANCELADA (HU-020) faz a venda sair das somas do dia; o fechamento calculado depois disso reflete apenas as ATIVA (FR-005).
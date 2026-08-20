# Data Model - Fase 1: Relatório de Carregamentos (Entradas)

**HU**: HU-017 - Relatório de Carregamentos (Entradas)
**Fase**: 1 - Modelo de dados
**Data**: 2026-08-20

## Visão geral

Feature de LEITURA: não cria tabela nova. Consolida carregamentos do período (RF-040) a partir das tabelas canônicas `tab_carregamento`, `tab_carregamento_item`, `tab_fornecedor` e `tab_produto`. As colunas do relatório seguem CONVENTIONS §10 (RF-042). Os nomes das tabelas são os mesmos definidos na feature de chegada de caminhão (HU-006).

## Tabelas consultadas

### tab_carregamento (RF-042)

| Coluna | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| id | bigint (PK, seq_carregamento) | sim | Identificador do carregamento |
| data_hora | timestamptz | sim | Data/hora da chegada (coluna do filtro de período) |
| id_fornecedor | bigint (FK tab_fornecedor) | sim | Fornecedor/distribuidora (RF-005) |
| custo_total | numeric(12,2) | sim | Custo total da carga (RF-010) |
| id_usuario | bigint (FK tab_usuario) | sim | Usuário que registrou a chegada (RNF-006) |

### tab_carregamento_item (RF-042)

| Coluna | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| id | bigint (PK, seq_carregamento_item) | sim | Identificador do item |
| id_carregamento | bigint (FK tab_carregamento) | sim | Carregamento do item |
| id_produto | bigint (FK tab_produto) | sim | Produto (carga + vasilhame, RF-001) |
| qtd_cheios_entraram | integer | sim | Cheios recebidos (entrada de estoque, RF-010/RF-011) |
| qtd_vazios_sairam | integer | sim | Vazios devolvidos à distribuidora (saída do pátio, RF-010) |
| valor_unitario | numeric(12,2) | sim | Valor unitário da carga (RF-010) |

### tab_fornecedor e tab_produto (consultadas)

- `tab_fornecedor.nome`: coluna Fornecedor (RF-005).
- `tab_produto` (carga + vasilhame, RF-001): coluna Produto.

## Regras de validação (derivadas de requisitos)

| Regra | Fonte |
|---|---|
| Filtro por período via `data_hora` de `tab_carregamento` (Hoje, 7 dias, Mês, personalizado) | RF-040, CT-002 |
| Somente carregamentos confirmados; registro só existe após confirmação (HU-006) | FR-004 |
| Colunas exatas conforme CONVENTIONS §10 | RF-042, CT-001 |
| `qtd_vazios_sairam` nunca ultrapassa o saldo de vazios do pátio naquele momento (garantido na escrita) | RDN-003 |
| Carregamento envolve dois fluxos opostos: cheios entram e vazios saem | RDN-009 |
| Custo médio recalculado a cada carregamento, na escrita | RF-012, CONVENTIONS §5.5 |
| Período personalizado com fim anterior ao início é rejeitado | Edge Case do spec |
| Período sem carregamentos gera relatório com zero linhas, sem erro | Edge Case do spec, SC-001 |

## Transições de estado

- Relatório é leitura pura: nenhum estado de negócio muda (RF-040 a RF-044).
- O carregamento não possui status de cancelamento no modelo atual: a chegada confirmada não é cancelável por esta feature; o registro permanece íntegro no histórico (RNF-007).
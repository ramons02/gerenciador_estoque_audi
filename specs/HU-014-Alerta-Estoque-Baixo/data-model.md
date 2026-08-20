# Data Model - Fase 1: Alerta de Estoque Baixo

**HU**: HU-014 - Alerta de Estoque Baixo
**Fase**: 1 - Modelo de dados
**Data**: 2026-08-20

## Visão geral

Feature de LEITURA: não cria tabela nova. O alerta é um estado DERIVADO comparando `tab_estoque.qtd_cheios` com `tab_estoque.limite_minimo` (RF-032); a sugestão de reposição usa o histórico em `tab_venda_item` (RGN-009). O limite mínimo é configurado por outra feature (RF-003, HU-003).

## Tabelas

### tab_estoque (consultada)

| Campo | Tipo | Obrigatório | PK/FK | Uso no alerta |
|---|---|---|---|---|
| id_produto | bigint | sim | PK/FK tab_produto.id | Agrupamento por produto |
| qtd_cheios | integer | sim | - | Comparado com o limite: alerta quando qtd_cheios <= limite_minimo (RF-032, CT-001) |
| qtd_vazios | integer | sim | - | Exibido como informação complementar |
| qtd_em_rua | integer | sim | - | Exibido como informação complementar |
| limite_minimo | integer | nao | - | Gatilho do alerta (RF-003); nulo = sem alerta (Edge Case) |

Regras derivadas:

- Alerta ativo: `qtd_cheios <= limite_minimo` E `limite_minimo` não nulo (RF-032, Edge Case).
- Saldo igual ao limite dispara alerta ("atingir ou ficar abaixo", RF-032 e Edge Case do spec).
- Enquanto a condição persistir, o alerta permanece (RGN-004, CT-003); ele some quando uma entrada de cheios eleva `qtd_cheios` acima de `limite_minimo` (CT-003).

### tab_venda_item (consultada para a média de vendas)

| Campo | Tipo | Obrigatório | PK/FK | Uso no alerta |
|---|---|---|---|---|
| id_venda | bigint | sim | FK tab_venda.id | Join para filtrar vendas do período e não canceladas |
| id_produto | bigint | sim | FK tab_produto.id | Agrupamento por produto |
| quantidade | integer | sim | - | Somado para o total vendido (média) |

Cálculo (RGN-009, CT-004):

- Total vendido nos últimos 30 dias = soma de `quantidade` dos itens do produto em vendas `ATIVA` no período.
- `mediaVendasDiarias` = total vendido nos últimos 30 dias / 30, com uma casa decimal.
- `sugestaoReposicao` = max(1, arredondaParaCima(mediaVendasDiarias)).
- Sem histórico de vendas no período: `sugestaoReposicao` = max(1, limiteMinimo - qtdCheios) (Edge Case: produto recém-cadastrado não inviabiliza a sugestão).

### Tabelas consultadas (sem alteração)

- `tab_produto`: nome do produto para exibição do alerta.
- `tab_venda`: filtro de status (`ATIVA`) e período para a média.
- `tab_usuario`: contexto de autenticação (não alterado nesta feature).

## Regras de validação (resumo, derivadas de requisitos)

| Regra | Fonte |
|---|---|
| Alerta quando qtd_cheios <= limite_minimo, limite não nulo | RF-032, CT-001, Edge Case |
| Saldo igual ao limite dispara alerta | RF-032, Edge Case do spec |
| Produto sem limite configurado nunca alerta | Edge Case do spec, SC-003 |
| Alerta persiste até entrada elevar o saldo acima do limite | RGN-004, CT-003 |
| Sugestão de reposição pela média de vendas diárias | RGN-009, CT-004 |

## Transições de estado

Alerta é derivado, então as "transições" são mudanças na condição, causadas por:

| Evento | Efeito no alerta |
|---|---|
| Venda baixa o saldo para <= limite durante o dia | Alerta ativa imediatamente (Edge Case do spec) |
| Entrada de cheios (carregamento) eleva saldo acima do limite | Alerta desativa (RGN-004, CT-003) |
| Entrada eleva saldo apenas até o limite | Alerta permanece ativo (<= inclui igualdade) |
| Alteração do limite mínimo pela HU-003 | Reavaliação na próxima consulta (leitura pura) |
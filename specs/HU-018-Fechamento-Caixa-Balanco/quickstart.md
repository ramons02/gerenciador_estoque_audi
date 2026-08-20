# Quickstart: Fechamento de Caixa e Balanço de Estoque

**HU**: HU-018 - Fechamento de Caixa e Balanço de Estoque
**Fase**: 1 - Validação end-to-end
**Data**: 2026-08-20

## Pré-requisitos

- PostgreSQL rodando, banco `gerenciador_estoque` criado e migrations aplicadas, incluindo `V<N>__cria_tab_fechamento_caixa.sql` (schema conforme `data-model.md`).
- API no ar na porta 8080.
- App no ar na porta 5173.
- Dados de apoio: vendas ATIVA de hoje em Dinheiro, PIX e Cartão em `tab_venda`; pelo menos uma venda CANCELADA de hoje (valida FR-005); carregamentos do período em `tab_carregamento` e movimentações em `tab_movimentacao_estoque`; produtos em `tab_produto` e saldos em `tab_estoque`.

## Setup

```bash
# API (em gerenciador_estoque_api/)
mvn spring-boot:run

# App (em gerenciador_estoque_app/)
npm install
npm run dev
```

## Cenários roteirizados

### CT-001 - Balanço de estoque por período

**Dado** um período com entradas e vendas registradas,
**Quando** o administrador gera o balanço de estoque (GET /api/relatorios/balanco, contrato em `contracts/api.md`),
**Então** cada produto lista Estoque Inicial, (+) Entradas, (-) Vendas, Estoque Final e Vazios em Pátio (RF-043).

**Dado** um período sem movimentações,
**Quando** o administrador gera o balanço,
**Então** o balanço é exibido com os produtos cadastrados e valores zerados, sem erro (Edge Case do spec).

### CT-002 - Fechamento de caixa com conciliação

**Dado** todas as vendas do dia conciliadas,
**Quando** o administrador conclui o fechamento de caixa do dia (POST /api/caixa/fechar),
**Então** o fechamento é concluído com sucesso e o registro fica com status FECHADO, totais por forma de pagamento, data/hora e usuário responsável.

**Dado** uma ou mais vendas do dia em aberto/em edição,
**Quando** o administrador tenta concluir o fechamento de caixa,
**Então** o fechamento é bloqueado e o sistema indica as vendas pendentes de conciliação (RGN-006).

**Dado** um fechamento já concluído no dia,
**Quando** o administrador tenta fechar novamente,
**Então** o sistema recusa com "O caixa deste dia já foi fechado." (unicidade por data).

### CT-003 - Exportação do balanço em Excel/CSV

**Dado** um balanço de estoque gerado com dados no período,
**Quando** o administrador clica no botão de exportação,
**Então** o sistema gera o arquivo CSV com as linhas e colunas do período selecionado (via GET /api/relatorios/balanco com Accept: text/csv, contrato da HU-015).

**Dado** o arquivo exportado,
**Quando** o administrador o abre em Excel, LibreOffice ou Google Sheets,
**Então** os cabeçalhos e valores estão legíveis, com acentos preservados (BOM UTF-8, RNF-009), e coincidem com o balanço em tela.

### CT-004 - Estoque inicial do período

**Dado** movimentações anteriores ao período selecionado,
**Quando** o administrador gera o balanço do período,
**Então** o estoque inicial de cada produto é calculado com base nas movimentações anteriores ao período (RF-043).

## Verificação adicional

- Conferir que o total do dia (GET /api/caixa/fechamento?data=) bate com a soma das vendas ATIVA do dia em `tab_venda` por forma de pagamento (RGN-008); vendas CANCELADA fora da soma (FR-005, RGN-007).
- O estoque final do balanço do dia confere com `tab_estoque.qtd_cheios` e `qtd_vazios` no mesmo instante (SC-003, Constituição §IV.2).
- Fechamento duplicado rejeitado; histórico do dia permanece consultável (RNF-007).
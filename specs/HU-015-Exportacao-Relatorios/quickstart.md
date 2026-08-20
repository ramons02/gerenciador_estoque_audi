# Quickstart: Exportação de Relatórios (Período Personalizado)

**HU**: HU-015 - Exportação de Relatórios (Período Personalizado)
**Fase**: 1 - Validação end-to-end
**Data**: 2026-08-20

## Pré-requisitos

- PostgreSQL rodando, banco `gerenciador_estoque` criado e migrations aplicadas (schema conforme `data-model.md`).
- API no ar na porta 8080.
- App no ar na porta 5173.
- Dados de apoio: vendas em `tab_venda`/`tab_venda_item`, carregamentos em `tab_carregamento`/`tab_carregamento_item` e movimentações em `tab_movimentacao_estoque`, em datas conhecidas (ex.: vendas nos últimos 3 dias e nenhuma ontem) para validar os períodos.

## Setup

```bash
# API (em gerenciador_estoque_api/)
mvn spring-boot:run

# App (em gerenciador_estoque_app/)
npm install
npm run dev
```

## Cenários roteirizados

### CT-001 - Seleção de período

**Dado** o painel de relatórios aberto,
**Quando** o administrador acessa o seletor de período,
**Então** as quatro opções estão disponíveis: Hoje, Últimos 7 dias, Mês Atual e Personalizado (RF-040).

**Dado** a opção Personalizado selecionada,
**Quando** o administrador informa data inicial e final,
**Então** o sistema aceita o intervalo e reconsulta os relatórios; fim anterior ao início é rejeitado com "A data final deve ser posterior ou igual à data inicial." (Edge Case do spec).

### CT-002 - Botão de exportação por relatório

**Dado** um período selecionado,
**Quando** o administrador clica em "Exportar CSV" no Relatório de Vendas,
**Então** o arquivo `relatorio-vendas-<data>.csv` é baixado com os dados consolidados do período (GET /api/relatorios/vendas com Accept: text/csv, contrato em `contracts/api.md`).

**Dado** os relatórios de vendas, carregamentos e balanço disponíveis,
**Quando** o administrador exporta cada um,
**Então** cada seção tem seu próprio botão funcional e os três arquivos são gerados (RF-044).

### CT-003 - Compatibilidade dos arquivos

**Dado** um relatório exportado,
**Quando** o administrador abre o arquivo em Excel, LibreOffice ou Google Sheets,
**Então** o arquivo abre corretamente, com caracteres acentuados preservados (BOM UTF-8, RNF-009).

### CT-004 - Cabeçalhos claros em português

**Dado** um relatório exportado,
**Quando** o administrador visualiza o arquivo,
**Então** as colunas são exatamente as de CONVENTIONS §10: Vendas (Data/Hora; Produto; Qtd; Valor Unitário; Total (R$); Forma de Pagamento; Tipo (Balcão/Entrega)); Carregamentos (Data; Fornecedor; Produto; Qtd Cheios Entraram; Qtd Vazios Saíram; Custo Total); Balanço (Produto; Estoque Inicial; (+) Entradas; (-) Vendas; Estoque Final; Vazios em Pátio).

## Verificação adicional

- Período sem movimentações: o CSV é gerado com cabeçalho e zero linhas de dados, sem erro (Edge Case do spec, SC-004).
- Exportar um relatório não altera o período selecionado dos outros relatórios no mesmo painel (Edge Case do spec).
- Vendas canceladas (status CANCELADA) não aparecem na consolidação (RGN-007).
# Quickstart: Relatório de Carregamentos (Entradas)

**HU**: HU-017 - Relatório de Carregamentos (Entradas)
**Fase**: 1 - Validação end-to-end
**Data**: 2026-08-20

## Pré-requisitos

- PostgreSQL rodando, banco `gerenciador_estoque` criado e migrations aplicadas (schema conforme `data-model.md`).
- API no ar na porta 8080.
- App no ar na porta 5173.
- Dados de apoio: carregamentos em `tab_carregamento`/`tab_carregamento_item` em datas conhecidas (ex.: carregamentos hoje, há 3 dias e há 20 dias), com fornecedores cadastrados em `tab_fornecedor` e produtos em `tab_produto`.

## Setup

```bash
# API (em gerenciador_estoque_api/)
mvn spring-boot:run

# App (em gerenciador_estoque_app/)
npm install
npm run dev
```

## Cenários roteirizados

### CT-001 - Colunas padrão do relatório

**Dado** carregamentos registrados no período,
**Quando** o administrador gera o relatório de carregamentos (GET /api/carregamentos/relatorio, contrato em `contracts/api.md`),
**Então** cada linha lista Data, Fornecedor, Produto, Qtd Cheios Entraram, Qtd Vazios Saíram e Custo Total, com valores coerentes com os carregamentos do período (RF-042).

**Dado** um período sem carregamentos registrados,
**Quando** o administrador gera o relatório,
**Então** o relatório é exibido vazio (nenhuma linha), sem erro (Edge Case do spec).

### CT-002 - Filtro por período

**Dado** carregamentos em datas diferentes,
**Quando** o administrador seleciona o período "Hoje",
**Então** o relatório lista apenas os carregamentos da data atual (RF-040).

**Dado** carregamentos em datas diferentes,
**Quando** o administrador seleciona um período personalizado com data inicial e final,
**Então** o relatório lista apenas os carregamentos dentro do intervalo.

**Dado** uma seleção de período inválida (data inicial posterior à final),
**Quando** o administrador tenta gerar o relatório,
**Então** o sistema recusa a seleção com "A data final deve ser posterior ou igual à data inicial." (Edge Case do spec).

### CT-003 - Exportação em Excel/CSV

**Dado** um relatório de carregamentos gerado com dados no período,
**Quando** o administrador clica no botão de exportação,
**Então** o sistema gera o arquivo CSV com as linhas e colunas do período selecionado (via GET /api/relatorios/carregamentos com Accept: text/csv, contrato da HU-015).

**Dado** o arquivo exportado,
**Quando** o administrador o abre em Excel, LibreOffice ou Google Sheets,
**Então** os cabeçalhos e valores estão legíveis, com acentos preservados (BOM UTF-8, RNF-009), e coincidem com o relatório em tela.

## Verificação adicional

- O relatório consolida apenas carregamentos confirmados (FR-004); validar contra `tab_carregamento` diretamente (Constituição §IV.2).
- Para cada item, `qtd_vazios_sairam` reflete a devolução de vazios registrada na chegada (RDN-003, RDN-009); conferir com `tab_carregamento_item`.
- Período sem carregamentos: relatório vazio e CSV com cabeçalho e zero linhas, sem erro (Edge Case do spec).
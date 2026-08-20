# Quickstart: Relatório de Vendas (Diário/Mensal)

**HU**: HU-016 - Relatório de Vendas (Diário/Mensal)
**Fase**: 1 - Validação end-to-end
**Data**: 2026-08-20

## Pré-requisitos

- PostgreSQL rodando, banco `gerenciador_estoque` criado e migrations aplicadas (schema conforme `data-model.md`).
- API no ar na porta 8080.
- App no ar na porta 5173.
- Dados de apoio: vendas em `tab_venda`/`tab_venda_item` em datas conhecidas (ex.: vendas hoje, há 3 dias e há 20 dias) e pelo menos uma venda com `status = CANCELADA` para validar a exclusão (FR-005). Nenhuma venda com `forma_pagamento` diferente de DINHEIRO, PIX ou CARTAO (RGN-002).

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

**Dado** vendas registradas no período,
**Quando** o administrador gera o relatório de vendas (GET /api/vendas/relatorio, contrato em `contracts/api.md`),
**Então** cada linha lista Data/Hora, Produto, Qtd, Valor Unitário, Total (R$), Forma de Pagamento e Tipo (Balcão/Entrega), com valores coerentes com as vendas do período (RF-041).

**Dado** o relatório de vendas gerado,
**Quando** o administrador verifica as linhas,
**Então** toda linha discrimina a forma de pagamento e o tipo para conferência do caixa físico (RGN-008).

**Dado** um período sem vendas registradas,
**Quando** o administrador gera o relatório,
**Então** o relatório é exibido vazio (nenhuma linha), sem erro (Edge Case do spec).

### CT-002 - Filtro por período

**Dado** vendas em datas diferentes,
**Quando** o administrador seleciona o período "Hoje",
**Então** o relatório lista apenas as vendas da data atual (RF-040).

**Dado** vendas em datas diferentes,
**Quando** o administrador seleciona um período personalizado com data inicial e final,
**Então** o relatório lista apenas as vendas dentro do intervalo.

**Dado** uma seleção de período inválida (data inicial posterior à final),
**Quando** o administrador tenta gerar o relatório,
**Então** o sistema recusa a seleção com "A data final deve ser posterior ou igual à data inicial." (Edge Case do spec).

### CT-003 - Exportação em Excel/CSV

**Dado** um relatório de vendas gerado com dados no período,
**Quando** o administrador clica no botão de exportação,
**Então** o sistema gera o arquivo CSV com as linhas e colunas do período selecionado (via GET /api/relatorios/vendas com Accept: text/csv, contrato da HU-015).

**Dado** o arquivo exportado,
**Quando** o administrador o abre em Excel, LibreOffice ou Google Sheets,
**Então** os cabeçalhos e valores estão legíveis, com acentos preservados (BOM UTF-8, RNF-009), e coincidem com o relatório em tela.

## Verificação adicional

- Vendas canceladas (status CANCELADA) não aparecem nas linhas nem nos totais do relatório (FR-005, RGN-007); validar contra `data-model.md` consultando `tab_venda` diretamente (Constituição §IV.2).
- O Total do período e os totais por forma de pagamento conferem com a soma das vendas ATIVA no banco (RGN-008).
- Período sem vendas: relatório vazio e CSV com cabeçalho e zero linhas, sem erro (Edge Case do spec).
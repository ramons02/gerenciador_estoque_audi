# Feature Specification: Relatório de Carregamentos (Entradas)

**Feature Branch**: `HU-017-Relatorio-Carregamentos`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como administrador, quero gerar o relatório de carregamentos por período, para acompanhar compras de carga e devoluções de vazios."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Gerar o relatório de carregamentos com as colunas padrão (Priority: P1)
O administrador acessa o painel de relatórios, seleciona o período e gera o relatório de carregamentos. Cada linha apresenta Data, Fornecedor, Produto, Qtd Cheios Entraram, Qtd Vazios Saíram e Custo Total, permitindo acompanhar as compras de carga e as devoluções de vazios à distribuidora.

**Why this priority**: É a finalidade central da HU — sem as colunas padrão, o administrador não consegue acompanhar entradas de carga e devoluções de vazios por período.

**Independent Test**: Abrir o relatório de carregamentos de um período com carregamentos registrados e verificar que cada linha exibe Data, Fornecedor, Produto, Qtd Cheios Entraram, Qtd Vazios Saíram e Custo Total, com valores coerentes com os carregamentos do período.

**Acceptance Scenarios**:

1. **Given** um período com carregamentos registrados, **When** o administrador gera o relatório de carregamentos, **Then** cada linha lista Data, Fornecedor, Produto, Qtd Cheios Entraram, Qtd Vazios Saíram e Custo Total.
2. **Given** um período sem carregamentos registrados, **When** o administrador gera o relatório, **Then** o relatório é exibido vazio (nenhuma linha), sem erro.

---

### User Story 2 - Filtrar o relatório por período (Priority: P2)
O administrador escolhe entre os períodos Hoje, Últimos 7 dias, Mês Atual ou um período personalizado. O relatório consolida apenas os carregamentos ocorridos dentro do período selecionado.

**Why this priority**: O filtro por período é necessário para acompanhar compras de carga, mas depende do relatório básico existir (US1) — por isso fica logo atrás.

**Independent Test**: Gerar o relatório para "Hoje" e confirmar que apenas os carregamentos de hoje aparecem; repetir com "Últimos 7 dias", "Mês Atual" e um período personalizado, conferindo os limites de data.

**Acceptance Scenarios**:

1. **Given** carregamentos em datas diferentes, **When** o administrador seleciona o período "Hoje", **Then** o relatório lista apenas os carregamentos da data atual.
2. **Given** carregamentos em datas diferentes, **When** o administrador seleciona um período personalizado com data inicial e final, **Then** o relatório lista apenas os carregamentos dentro desse intervalo.
3. **Given** uma seleção de período inválida (data inicial posterior à final), **When** o administrador tenta gerar o relatório, **Then** o sistema recusa a seleção e solicita um período válido.

---

### User Story 3 - Exportar o relatório em Excel/CSV (Priority: P3)
O administrador exporta o relatório de carregamentos do período selecionado por meio do botão de exportação disponível no painel, obtendo um arquivo Excel/CSV com as mesmas colunas exibidas, para arquivamento ou análise externa.

**Why this priority**: A exportação agrega valor de arquivamento, mas o relatório em tela (US1/US2) já atende ao acompanhamento diário — por isso é a última prioridade.

**Independent Test**: Clicar no botão de exportação de um relatório de carregamentos de um período com dados e abrir o arquivo em planilha, conferindo que as linhas e colunas coincidem com o relatório em tela.

**Acceptance Scenarios**:

1. **Given** um relatório de carregamentos gerado com dados no período selecionado, **When** o administrador clica no botão de exportação, **Then** o sistema gera o arquivo Excel/CSV com as linhas e colunas do período selecionado.
2. **Given** o arquivo exportado, **When** o administrador o abre em Excel, LibreOffice ou Google Sheets, **Then** os cabeçalhos e valores estão legíveis e coincidem com o relatório em tela.

### Edge Cases

- Período sem carregamentos: o relatório é exibido e exportado vazio, sem linhas e sem erro.
- Período personalizado com data inicial maior que a data final é recusado na seleção.
- O relatório reflete apenas carregamentos registrados e confirmados no sistema; carregamentos em edição não aparecem.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O relatório de carregamentos deve listar, por linha, as colunas: Data, Fornecedor, Produto, Qtd Cheios Entraram, Qtd Vazios Saíram e Custo Total.
- **FR-002**: O relatório deve respeitar o período selecionado, consolidando apenas os carregamentos ocorridos dentro de Hoje, Últimos 7 dias, Mês Atual ou período personalizado.
- **FR-003**: O painel deve disponibilizar um botão de exportação que gera o relatório em Excel/CSV com os dados do período selecionado.
- **FR-004**: O relatório deve consolidar somente carregamentos registrados e confirmados no sistema.

### Key Entities *(include if feature involves data)*

- **Carregamento**: registro de chegada de caminhão com data, fornecedor, produto, quantidade de cascos cheios recebidos, quantidade de vazios devolvidos à distribuidora e custo total da carga.
- **Fornecedor**: distribuidora da qual o carregamento foi recebido, exibida no relatório.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: O administrador consegue gerar o relatório de carregamentos de qualquer período suportado (Hoje, 7 dias, Mês Atual, personalizado) sem depender de outro usuário ou sistema.
- **SC-002**: Cada carregamento registrado no período aparece no relatório com as quantidades de cheios entrados e vazios saídos, permitindo conferir devoluções à distribuidora.
- **SC-003**: A exportação em Excel/CSV gera arquivo compatível com Excel, LibreOffice e Google Sheets para todo relatório exibido.

## Assumptions

- Os carregamentos do período já estão registrados e confirmados no sistema com os campos necessários (data, fornecedor, produto, quantidades e custo total).
- O histórico de movimentações é mantido por pelo menos 12 meses, permitindo relatórios mensais completos.
- Cada carregamento envolve dois fluxos opostos (entrada de cheios e saída de vazios), conforme a regra de domínio do carregamento.

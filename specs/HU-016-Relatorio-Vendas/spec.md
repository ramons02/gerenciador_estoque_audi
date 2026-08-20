# Feature Specification: Relatório de Vendas (Diário/Mensal)

**Feature Branch**: `HU-016-Relatorio-Vendas`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como administrador, quero gerar o relatório de vendas por período com as colunas padrão, para conferir as vendas do dia/mês."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Gerar o relatório de vendas com as colunas padrão (Priority: P1)
O administrador acessa o painel de relatórios, seleciona o período desejado e gera o relatório de vendas. Cada linha do relatório apresenta Data/Hora, Produto, Qtd, Valor Unitário, Total (R$), Forma de Pagamento e Tipo (Balcão/Entrega), permitindo a conferência das vendas do dia ou do mês e a conferência do caixa físico.

**Why this priority**: É a finalidade central da HU — sem o relatório completo com as colunas padrão, o administrador não consegue conferir as vendas do dia/mês nem conciliar o caixa físico.

**Independent Test**: Abrir o relatório de vendas de um período com vendas registradas e verificar que todas as linhas exibem as colunas Data/Hora, Produto, Qtd, Valor Unitário, Total (R$), Forma de Pagamento e Tipo (Balcão/Entrega), com valores coerentes com as vendas registradas no período.

**Acceptance Scenarios**:

1. **Given** um período com vendas registradas, **When** o administrador gera o relatório de vendas, **Then** cada linha lista Data/Hora, Produto, Qtd, Valor Unitário, Total (R$), Forma de Pagamento e Tipo (Balcão/Entrega).
2. **Given** um relatório de vendas gerado, **When** o administrador verifica as linhas, **Then** toda linha discrimina a forma de pagamento e o tipo (Balcão/Entrega) para conferência do caixa físico.
3. **Given** um período sem vendas registradas, **When** o administrador gera o relatório, **Then** o relatório é exibido vazio (nenhuma linha), sem erro.

---

### User Story 2 - Filtrar o relatório por período (Priority: P2)
O administrador escolhe entre os períodos Hoje, Últimos 7 dias, Mês Atual ou um período personalizado. O relatório consolida apenas as vendas ocorridas dentro do período selecionado.

**Why this priority**: O filtro por período é essencial para a conferência diária, mas depende do relatório básico existir (US1) — por isso fica logo atrás.

**Independent Test**: Gerar o relatório para "Hoje" e confirmar que apenas as vendas de hoje aparecem; repetir com "Últimos 7 dias", "Mês Atual" e um período personalizado, conferindo os limites de data.

**Acceptance Scenarios**:

1. **Given** vendas em datas diferentes, **When** o administrador seleciona o período "Hoje", **Then** o relatório lista apenas as vendas da data atual.
2. **Given** vendas em datas diferentes, **When** o administrador seleciona um período personalizado com data inicial e final, **Then** o relatório lista apenas as vendas dentro desse intervalo.
3. **Given** uma seleção de período inválida (data inicial posterior à final), **When** o administrador tenta gerar o relatório, **Then** o sistema recusa a seleção e solicita um período válido.

---

### User Story 3 - Exportar o relatório em Excel/CSV (Priority: P3)
O administrador exporta o relatório de vendas do período selecionado por meio do botão de exportação disponível no painel, obtendo um arquivo Excel/CSV com as mesmas colunas exibidas, para arquivamento ou análise externa.

**Why this priority**: A exportação agrega valor de arquivamento, mas o relatório em tela (US1/US2) já atende à conferência diária — por isso é a última prioridade.

**Independent Test**: Clicar no botão de exportação de um relatório de vendas de um período com dados e abrir o arquivo em planilha, conferindo que as linhas e colunas coincidem com o relatório em tela.

**Acceptance Scenarios**:

1. **Given** um relatório de vendas gerado com dados no período selecionado, **When** o administrador clica no botão de exportação, **Then** o sistema gera o arquivo Excel/CSV com as linhas e colunas do período selecionado.
2. **Given** o arquivo exportado, **When** o administrador o abre em Excel, LibreOffice ou Google Sheets, **Then** os cabeçalhos e valores estão legíveis e coincidem com o relatório em tela.

### Edge Cases

- Período sem vendas: o relatório é exibido e exportado vazio, sem linhas e sem erro.
- Período personalizado com data inicial maior que a data final é recusado na seleção.
- Vendas canceladas não entram nas linhas do relatório.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O relatório de vendas deve listar, por linha, as colunas: Data/Hora, Produto, Qtd, Valor Unitário, Total (R$), Forma de Pagamento e Tipo (Balcão/Entrega).
- **FR-002**: O relatório deve respeitar o período selecionado, consolidando apenas as vendas ocorridas dentro de Hoje, Últimos 7 dias, Mês Atual ou período personalizado.
- **FR-003**: O relatório deve discriminar a forma de pagamento e o tipo (Balcão/Entrega) em todas as linhas, para conferência do caixa físico.
- **FR-004**: O painel deve disponibilizar um botão de exportação que gera o relatório em Excel/CSV com os dados do período selecionado.
- **FR-005**: Vendas canceladas não devem aparecer nas linhas do relatório.

### Key Entities *(include if feature involves data)*

- **Venda**: registro de venda com data/hora, forma de pagamento (Dinheiro, PIX, Cartão), tipo (Balcão/Entrega), status (incluindo "cancelado") e usuário responsável.
- **Item de Venda**: linha da venda com produto, quantidade e valor unitário; o total (R$) da linha é derivado da quantidade e do valor unitário.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: O administrador consegue gerar o relatório de vendas de qualquer período suportado (Hoje, 7 dias, Mês Atual, personalizado) sem depender de outro usuário ou sistema.
- **SC-002**: O relatório do dia permite conferir o caixa físico, discriminando forma de pagamento e tipo (Balcão/Entrega) em 100% das linhas.
- **SC-003**: A exportação em Excel/CSV gera arquivo compatível com Excel, LibreOffice e Google Sheets para todo relatório exibido.

## Assumptions

- As vendas do período já estão registradas no sistema com os campos necessários (data/hora, produto, quantidade, valor, forma de pagamento e tipo).
- O histórico de movimentações é mantido por pelo menos 12 meses, permitindo relatórios mensais completos.
- O relatório consolida vendas não canceladas do período; a regra de exclusão de canceladas é definida na HU-020.

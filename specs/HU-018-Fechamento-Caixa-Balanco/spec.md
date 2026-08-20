# Feature Specification: Fechamento de Caixa e Balanço de Estoque

**Feature Branch**: `HU-018-Fechamento-Caixa-Balanco`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como administrador, quero gerar o balanço de estoque e fechamento de caixa por período, para saber o estoque final e conferir o caixa do dia."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Gerar o balanço de estoque por período (Priority: P1)
O administrador gera o balanço de estoque de um período. Para cada produto, o balanço apresenta Estoque Inicial, (+) Entradas, (−) Vendas, Estoque Final e Vazios em Pátio. O estoque inicial do período é calculado com base nas movimentações anteriores ao período, e o estoque final com as movimentações do próprio período.

**Why this priority**: É a finalidade central da HU — sem o balanço com os campos por produto, o administrador não sabe o estoque final do período.

**Independent Test**: Gerar o balanço de um período com entradas e vendas registradas e conferir que cada produto exibe Estoque Inicial, (+) Entradas, (−) Vendas, Estoque Final e Vazios em Pátio, com o estoque inicial coerente com as movimentações anteriores ao período.

**Acceptance Scenarios**:

1. **Given** um período com entradas e vendas registradas, **When** o administrador gera o balanço de estoque, **Then** cada produto lista Estoque Inicial, (+) Entradas, (−) Vendas, Estoque Final e Vazios em Pátio.
2. **Given** um balanço de estoque gerado para um período, **When** o administrador confere o estoque inicial de um produto, **Then** ele é calculado com base nas movimentações anteriores ao período.
3. **Given** um período sem movimentações, **When** o administrador gera o balanço, **Then** o balanço é exibido com os produtos cadastrados e valores zerados, sem erro.

---

### User Story 2 - Fazer o fechamento de caixa do dia (Priority: P1)
O administrador conclui o fechamento de caixa do dia. O fechamento só pode ser concluído quando todas as vendas do dia estiverem conciliadas; se houver venda em aberto/em edição, o fechamento é bloqueado até a conciliação.

**Why this priority**: O fechamento de caixa é a ação que garante a conferência do caixa do dia, exigida para encerrar o dia operacional — sem ele o balanço do dia não tem valor.

**Independent Test**: Tentar concluir o fechamento de caixa com todas as vendas do dia conciliadas e confirmar que conclui; repetir com uma venda em aberto e confirmar que o fechamento é bloqueado com aviso.

**Acceptance Scenarios**:

1. **Given** todas as vendas do dia conciliadas, **When** o administrador conclui o fechamento de caixa do dia, **Then** o fechamento é concluído com sucesso.
2. **Given** uma ou mais vendas do dia em aberto/em edição, **When** o administrador tenta concluir o fechamento de caixa, **Then** o fechamento é bloqueado e o sistema indica as vendas pendentes de conciliação.
3. **Given** um fechamento concluído, **When** o administrador gera o balanço do dia, **Then** o estoque final e os vazios em pátio refletem todas as movimentações do dia.

---

### User Story 3 - Exportar o balanço em Excel/CSV (Priority: P3)
O administrador exporta o balanço de estoque do período selecionado por meio do botão de exportação disponível no painel, obtendo um arquivo Excel/CSV com as mesmas colunas exibidas.

**Why this priority**: A exportação agrega valor de arquivamento, mas o balanço em tela (US1/US2) já atende à conferência diária — por isso é a última prioridade.

**Independent Test**: Clicar no botão de exportação de um balanço de um período com dados e abrir o arquivo em planilha, conferindo que as linhas e colunas coincidem com o balanço em tela.

**Acceptance Scenarios**:

1. **Given** um balanço de estoque gerado com dados no período selecionado, **When** o administrador clica no botão de exportação, **Then** o sistema gera o arquivo Excel/CSV com as linhas e colunas do período selecionado.
2. **Given** o arquivo exportado, **When** o administrador o abre em Excel, LibreOffice ou Google Sheets, **Then** os cabeçalhos e valores estão legíveis e coincidem com o balanço em tela.

### Edge Cases

- Período sem movimentações anteriores: o estoque inicial é zero para todos os produtos.
- Período em que o produto ainda não existia no início: o estoque inicial é zero e as entradas/vendas contam apenas após o cadastro.
- Fechamento de caixa com vendas do dia em edição é bloqueado até a conciliação completa.
- Vendas canceladas não entram nas vendas (−) do balanço.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O balanço de estoque deve listar, por produto, as colunas: Estoque Inicial, (+) Entradas, (−) Vendas, Estoque Final e Vazios em Pátio.
- **FR-002**: O estoque inicial do período deve ser calculado com base nas movimentações anteriores ao período selecionado.
- **FR-003**: O fechamento de caixa do dia só pode ser concluído se todas as vendas do dia estiverem conciliadas (sem vendas em aberto/em edição).
- **FR-004**: O painel deve disponibilizar um botão de exportação que gera o balanço em Excel/CSV com os dados do período selecionado.
- **FR-005**: Vendas canceladas não devem ser contabilizadas nas vendas (−) do balanço.

### Key Entities *(include if feature involves data)*

- **Balanço de Estoque**: consolidação por produto de estoque inicial, entradas, vendas, estoque final e vazios em pátio de um período.
- **Fechamento de Caixa**: encerramento do caixa do dia, com status de conciliação das vendas do dia.
- **Movimentação**: registro de entrada ou saída que baseia os cálculos de estoque inicial e final do balanço.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: O administrador consegue gerar o balanço de estoque de qualquer período suportado (Hoje, 7 dias, Mês Atual, personalizado) sem depender de outro usuário ou sistema.
- **SC-002**: O fechamento de caixa do dia só conclui quando 100% das vendas do dia estiverem conciliadas; nenhum fechamento com vendas em aberto é aceito.
- **SC-003**: O estoque final do balanço confere com o estoque em tempo real do pátio para o mesmo instante.
- **SC-004**: A exportação em Excel/CSV gera arquivo compatível com Excel, LibreOffice e Google Sheets para todo balanço exibido.

## Assumptions

- Todas as movimentações (entradas e vendas) do período estão registradas e consolidadas no sistema.
- O histórico de movimentações é mantido por pelo menos 12 meses, permitindo calcular estoque inicial de períodos retirados.
- A conciliação das vendas do dia é pré-requisito para o fechamento de caixa, conforme a regra de negócio de fechamento.

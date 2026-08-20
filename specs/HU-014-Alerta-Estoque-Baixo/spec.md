# Feature Specification: Alerta de Estoque Baixo

**Feature Branch**: `HU-014-Alerta-Estoque-Baixo`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como administrador, quero ser avisado visualmente quando o estoque cheio de um produto estiver no limite mínimo, para repor a tempo."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Alerta visual no limite mínimo (Priority: P1)
Quando o estoque cheio de um produto atinge ou fica abaixo do limite mínimo configurado, o sistema exibe um alerta visual, avisando o administrador de que o produto precisa de reposição.

**Why this priority**: É o comportamento central da HU (RF-032): o alerta é o mecanismo que permite ao administrador repor a tempo, evitando ruptura de venda.

**Independent Test**: Configurar o limite mínimo de um produto acima do saldo de cheios atual e verificar que o alerta visual é exibido; subir o saldo acima do limite e verificar que o alerta desaparece.

**Acceptance Scenarios**:

1. **Given** um produto com limite mínimo configurado, **When** o saldo de cheios fica igual ou abaixo desse limite, **Then** o sistema exibe alerta visual de estoque baixo (CT-001, RF-032).
2. **Given** um produto com saldo de cheios acima do limite mínimo, **When** o administrador consulta o estoque, **Then** o sistema não exibe alerta para esse produto (CT-001).

---

### User Story 2 - Visibilidade do alerta no dashboard e no painel (Priority: P1)
O alerta de estoque baixo é visível tanto no dashboard (resumo do dia) quanto no painel de estoque, garantindo que o administrador não perca a informação em nenhum dos pontos de acompanhamento.

**Why this priority**: A exibição em ambos os locais (RF-053 + RF-032) garante que o alerta seja percebido independentemente de onde o administrador esteja acompanhando a operação.

**Independent Test**: Com um produto em estoque baixo, abrir o dashboard e o painel de estoque e confirmar que o alerta aparece em ambas as telas.

**Acceptance Scenarios**:

1. **Given** um produto com estoque cheio no/abaixo do limite mínimo, **When** o administrador abre o dashboard, **Then** o alerta de estoque baixo é exibido (CT-002, RF-053).
2. **Given** um produto com estoque cheio no/abaixo do limite mínimo, **When** o administrador abre o painel de estoque, **Then** o produto aparece destacado com o alerta visual (CT-002).

---

### User Story 3 - Persistência do alerta até nova entrada (Priority: P2)
O alerta permanece ativo enquanto o estoque não subir acima do limite mínimo; ele só desaparece quando uma nova entrada eleva o saldo de cheios acima do limite.

**Why this priority**: A persistência (RGN-004) evita que o alerta seja ignorado ou desapareça sem reposição — o administrador precisa ser lembrado até o estoque ser de fato recomposto.

**Independent Test**: Manter um produto abaixo do limite, registrar uma entrada que eleve o saldo acima do limite e verificar que o alerta deixa de ser exibido somente após essa entrada.

**Acceptance Scenarios**:

1. **Given** um produto em alerta de estoque baixo, **When** não há nova entrada de caminhão e o saldo permanece no/abaixo do limite, **Then** o alerta persiste visível (CT-003, RGN-004).
2. **Given** um produto em alerta de estoque baixo, **When** uma nova entrada eleva o saldo de cheios acima do limite mínimo, **Then** o alerta deixa de ser exibido (CT-003, RGN-004).

---

### User Story 4 - Sugestão de reposição pela média de vendas (Priority: P3)
Para cada produto em alerta, o sistema sugere a quantidade de reposição (quantidade a comprar no próximo carregamento) calculada com base na média de vendas diárias do produto.

**Why this priority**: A sugestão (RGN-009) agrega valor ao alerta orientando a decisão de compra, mas é um aprimoramento — o alerta em si já cumpre o objetivo central da HU.

**Independent Test**: Com um produto em alerta, verificar que o sistema exibe uma sugestão de quantidade de reposição consistente com a média de vendas diárias do produto.

**Acceptance Scenarios**:

1. **Given** um produto em alerta de estoque baixo, **When** o administrador visualiza o alerta, **Then** o sistema sugere uma quantidade de reposição calculada pela média de vendas diárias do produto (CT-004, RGN-009).

### Edge Cases

- Produto sem limite mínimo configurado não deve gerar alerta (a configuração do limite é pré-requisito, RF-003).
- Produto recém-cadastrado, sem histórico de vendas, não deve inviabilizar a sugestão de reposição (cálculo deve tratar ausência de média).
- Estoque exatamente igual ao limite mínimo deve disparar o alerta ("atingir ou ficar abaixo", conforme RF-032).
- Uma venda que derrube o saldo para o limite durante o dia deve acionar o alerta imediatamente, sem esperar recarga de tela.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema deve gerar alerta visual de estoque baixo quando o saldo de cheios de um produto atingir ou ficar abaixo do limite mínimo configurado (RF-032, CT-001).
- **FR-002**: O alerta de estoque baixo deve ser exibido no dashboard e no painel de estoque (RF-053, CT-002).
- **FR-003**: O alerta deve persistir visível até que uma nova entrada eleve o saldo de cheios acima do limite mínimo configurado (RGN-004, CT-003).
- **FR-004**: O sistema deve sugerir a quantidade de reposição com base na média de vendas diárias do produto (RGN-009, CT-004).

### Key Entities *(include if feature involves data)*

- **Produto**: item composto por carga/conteúdo + vasilhame/casco (ex.: Gás P13); o alerta é por produto.
- **Limite mínimo de estoque**: parâmetro configurado por produto (RF-003) que define o gatilho do alerta.
- **Saldo de Cheios**: estado de estoque comparado com o limite mínimo para ativar/desativar o alerta.
- **Alerta de Estoque Baixo**: indicador visual ativo enquanto o saldo de cheios estiver no/abaixo do limite.
- **Média de vendas diárias**: métrica calculada por produto, base para a sugestão de reposição.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% dos produtos com saldo de cheios no/abaixo do limite mínimo exibem alerta visual ativo.
- **SC-002**: 100% dos alertas ativos são visíveis tanto no dashboard quanto no painel de estoque.
- **SC-003**: 0 alertas exibidos para produtos com saldo acima do limite mínimo ou sem limite configurado.
- **SC-004**: 100% dos produtos em alerta exibem sugestão de reposição calculada pela média de vendas diárias.

## Assumptions

- O limite mínimo por produto já é configurado por outra feature (HU-003 / RF-003) — esta feature apenas consome o parâmetro.
- "Nova entrada" que desativa o alerta refere-se a entrada de cheios (carregamento/caminhão), conforme RGN-004.
- A sugestão de reposição é orientativa (não obriga o carregamento) e usa dados de vendas já registrados pelo sistema.
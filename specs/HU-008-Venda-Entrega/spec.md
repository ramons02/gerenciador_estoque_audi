# Feature Specification: Venda com Entrega (Taxa de Entrega)

**Feature Branch**: `HU-008-Venda-Entrega`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como vendedor, quero lançar vendas do tipo Entrega com taxa de entrega automática, para que o total cobrado inclua a entrega sem cálculo manual."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Lançar venda do tipo Entrega com taxa automática (Priority: P1)
Ao selecionar o tipo "Entrega" no lançamento de venda, o sistema soma automaticamente a taxa de entrega configurada ao total, sem cálculo manual do vendedor.

**Why this priority**: É o fluxo principal da venda com entrega — garante que o valor cobrado cubra o custo de entrega e que o caixa feche sem depender do vendedor calcular a taxa.

**Independent Test**: Lançar uma venda do tipo Entrega e verificar que o total exibido é quantidade × preço + taxa de entrega configurada.

**Acceptance Scenarios**:

1. **Given** uma taxa de entrega configurada no sistema, **When** o vendedor lança uma venda com tipo Entrega, **Then** a taxa é somada automaticamente ao total.
2. **Given** a mesma venda lançada com tipo Balcão, **When** o total é calculado, **Then** a taxa de entrega não é aplicada.

---

### User Story 2 - Configurar o valor da taxa de entrega (Priority: P2)
O administrador configura o valor da taxa de entrega, que passa a valer para as novas vendas do tipo Entrega.

**Why this priority**: É o requisito de configuração que precede o fluxo de venda com entrega — sem a taxa definida o total da entrega não é calculado corretamente.

**Independent Test**: Alterar o valor da taxa de entrega nas Configurações e lançar uma venda do tipo Entrega para verificar que o novo valor é aplicado.

**Acceptance Scenarios**:

1. **Given** um administrador autenticado, **When** ele altera o valor da taxa de entrega nas Configurações, **Then** o novo valor passa a ser usado nas vendas do tipo Entrega.

---

### User Story 3 - Discriminar tipo e taxa no relatório de vendas (Priority: P3)
O relatório de vendas discrimina o tipo de cada venda (Balcão/Entrega) e exibe o total com a taxa aplicada, permitindo a conferência do caixa físico.

**Why this priority**: É a conferência contábil posterior ao fluxo — necessária para o fechamento de caixa, mas não bloqueia o lançamento do dia.

**Independent Test**: Gerar o relatório de vendas de um período com vendas dos dois tipos e conferir que o tipo e o total com taxa aparecem discriminados.

**Acceptance Scenarios**:

1. **Given** vendas dos tipos Balcão e Entrega registradas no período, **When** o relatório de vendas é gerado, **Then** cada venda aparece com seu tipo e o total com a taxa aplicada quando Entrega.

### Edge Cases

- Taxa de entrega configurada como zero (entrega gratuita) - o total segue a regra, sem acréscimo de entrega (o acréscimo do cartão, se aplicável, é regra da HU-007: somente carga Gás).
- Taxa alterada após vendas do dia já lançadas — apenas as novas vendas usam o novo valor.
- Venda do tipo Entrega lançada sem taxa configurada — o sistema deve exigir a configuração antes de permitir a conclusão.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema deve somar automaticamente a taxa de entrega configurada ao total quando o tipo da venda for Entrega.
- **FR-002**: O sistema deve permitir ao administrador configurar o valor da taxa de entrega.
- **FR-003**: O sistema deve discriminar o tipo (Balcão/Entrega) e o total com taxa no relatório de vendas.

### Key Entities

- **Taxa de Entrega**: valor configurável aplicado automaticamente ao total das vendas do tipo Entrega.
- **Venda**: registro com tipo (Balcão/Entrega) e total que inclui a taxa de entrega quando aplicável.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% das vendas do tipo Entrega exibem o total com a taxa aplicada automaticamente.
- **SC-002**: Alterações no valor da taxa configurada refletem imediatamente nas novas vendas.
- **SC-003**: 100% das vendas no relatório discriminam o tipo e o total com taxa.

## Assumptions

- Os tipos de venda são Balcão e Entrega (RF-020).
- A taxa é um valor por entrega, configurado pelo administrador (RF-022, RGN-001).
# Feature Specification: Bloqueio de Venda sem Estoque

**Feature Branch**: `HU-013-Bloqueio-Sem-Estoque`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como vendedor, quero que o sistema bloqueie a venda quando não houver estoque cheio suficiente, para nunca vender o que não tenho."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Bloqueio de venda por estoque cheio insuficiente (Priority: P1)
Ao lançar uma venda com quantidade maior do que o saldo de cheios disponível, o sistema bloqueia a operação e exibe uma mensagem clara ao vendedor, impedindo a venda de um produto que não existe no estoque.

**Why this priority**: É o comportamento central da HU (RF-031) — o bloqueio garante que o estabelecimento nunca venda o que não tem, protegendo a confiabilidade do estoque e do atendimento.

**Independent Test**: Estando logado como vendedor, tentar confirmar uma venda com quantidade superior ao saldo de cheios e verificar que o sistema bloqueia com mensagem clara, sem baixar estoque.

**Acceptance Scenarios**:

1. **Given** um produto com saldo de cheios igual a X, **When** o vendedor tenta confirmar uma venda de quantidade maior que X, **Then** o sistema bloqueia a venda e exibe mensagem clara de estoque insuficiente (CT-001, RF-031).
2. **Given** um produto com saldo de cheios suficiente, **When** o vendedor confirma a venda, **Then** o sistema permite a venda normalmente, sem bloqueio (CT-001).

---

### User Story 2 - Bloqueio contra estoque negativo em vendas simultâneas (Priority: P1)
Duas vendas do mesmo produto acontecendo ao mesmo tempo não podem, juntas, ultrapassar o saldo de cheios disponível: o sistema garante que nenhuma venda gere estoque negativo.

**Why this priority**: O controle de concorrência (RNF-008) é o que impede o estado proibido de estoque negativo (RDN-005) em cenários reais de portaria com vendedores simultâneos; sem ele, o bloqueio individual seria furável.

**Independent Test**: Executar duas vendas do mesmo produto de forma concorrente cuja soma excede o saldo de cheios e verificar que apenas a(s) que cabe(m) no saldo são aprovadas, sem nenhum estado negativo.

**Acceptance Scenarios**:

1. **Given** um saldo de cheios finito de um produto, **When** duas vendas simultâneas tentam consumir, em conjunto, mais do que o saldo, **Then** o sistema bloqueia a venda que excede o saldo restante e o estoque nunca fica negativo (CT-002, RNF-008, RDN-005).

---

### User Story 3 - Bloqueio em todos os fluxos de venda (Priority: P2)
O bloqueio por falta de estoque cheio vale para todos os fluxos de saída: venda de balcão, entrega, troca de vasilhame e venda de vasilhame novo — nenhum fluxo pode vender sem estoque.

**Why this priority**: A regra precisa ser uniforme em toda a operação (RF-031 aplicada em RF-020, RF-022, RF-023, RF-024); inconsistência entre fluxos criaria brechas para vender o que não se tem.

**Independent Test**: Para cada fluxo (balcão, entrega, troca, vasilhame novo), tentar uma venda acima do saldo de cheios e verificar que o bloqueio ocorre em todos.

**Acceptance Scenarios**:

1. **Given** um produto sem saldo de cheios suficiente, **When** o vendedor tenta uma venda por balcão, **Then** o sistema bloqueia a operação (CT-003).
2. **Given** um produto sem saldo de cheios suficiente, **When** o vendedor tenta uma venda por entrega, **Then** o sistema bloqueia a operação (CT-003).
3. **Given** um produto sem saldo de cheios suficiente, **When** o vendedor tenta uma venda com troca de vasilhame, **Then** o sistema bloqueia a operação (CT-003).
4. **Given** um produto sem saldo de cheios suficiente, **When** o vendedor tenta uma venda de vasilhame novo, **Then** o sistema bloqueia a operação (CT-003).

### Edge Cases

- Quantidade solicitada igual ao saldo de cheios disponível deve ser permitida (bloqueio apenas quando a quantidade é maior que o saldo).
- Venda cancelada deve devolver o estoque ao saldo, permitindo novas vendas dentro do que foi liberado (RGN-007).
- O bloqueio não deve ocorrer por falta de vazios ou de vasilhames "em rua" — apenas pela falta de cheios, conforme a regra definida.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema deve bloquear a venda quando a quantidade solicitada for maior que o saldo de cheios disponível, exibindo mensagem clara ao vendedor (RF-031, CT-001).
- **FR-002**: O sistema deve garantir, sob vendas simultâneas do mesmo produto, que nenhuma venda gere estoque negativo (RNF-008, RDN-005, CT-002).
- **FR-003**: O bloqueio por estoque insuficiente deve ser aplicado em todos os fluxos de venda: balcão, entrega, troca de vasilhame e venda de vasilhame novo (CT-003).
- **FR-004**: O sistema deve liberar a venda quando a quantidade solicitada for igual ou menor que o saldo de cheios disponível (CT-001).

### Key Entities *(include if feature involves data)*

- **Produto**: item composto por carga/conteúdo + vasilhame/casco (ex.: Gás P13); o saldo de cheios é a base para o bloqueio.
- **Saldo de Cheios**: estado de estoque consultado para validar a quantidade solicitada em cada venda.
- **Venda**: operação de saída (balcão, entrega, troca, vasilhame novo) cuja confirmação é condicionada à disponibilidade de cheios.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 0 vendas confirmadas com quantidade maior que o saldo de cheios disponível.
- **SC-002**: 0 ocorrências de estoque negativo em qualquer produto, inclusive sob vendas simultâneas.
- **SC-003**: 100% dos fluxos de venda (balcão, entrega, troca, vasilhame novo) aplicam o bloqueio por estoque insuficiente.

## Assumptions

- O bloqueio considera apenas o saldo de cheios (estado pronto para venda), conforme RF-031.
- A regra de estoque nunca negativo (RDN-005) já é um invariante do domínio que esta feature implementa no momento da venda.
- O estoque é atualizado atomicamente em cada venda (RNF-005), de modo que o saldo consultado para o bloqueio é sempre o saldo atual.
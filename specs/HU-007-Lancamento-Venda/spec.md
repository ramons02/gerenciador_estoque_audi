# Feature Specification: Lançamento Rápido de Venda (Balcão/Entrega)

**Feature Branch**: `HU-007-Lancamento-Venda`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como vendedor, quero lançar uma venda rapidamente com produto, quantidade, tipo e forma de pagamento, para atender o cliente da portaria sem demora."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Lançar uma venda rapidamente na portaria (Priority: P1)
O vendedor seleciona o produto, informa a quantidade, o tipo de venda (Balcão ou Entrega) e a forma de pagamento. O sistema calcula automaticamente o total (quantidade × preço de venda). O lançamento deve ser concluído em poucos segundos para atender o cliente da portaria sem demora.

**Why this priority**: É o fluxo principal de receita da revenda — a portaria não pode parar; um lançamento lento impacta diretamente o atendimento e o faturamento do dia.

**Independent Test**: Lançar uma venda completa de um produto com quantidade, tipo e forma de pagamento e verificar que o total exibido é quantidade × preço de venda e que a venda é registrada com sucesso.

**Acceptance Scenarios**:

1. **Given** um produto com preço de venda cadastrado, **When** o vendedor lança uma venda com quantidade 2 e forma de pagamento Dinheiro, **Then** o total é calculado automaticamente como 2 × preço de venda.
2. **Given** o lançamento de venda aberto, **When** o vendedor tenta confirmar sem informar produto, quantidade, tipo ou forma de pagamento, **Then** o sistema exige os campos obrigatórios antes de concluir.
3. **Given** uma venda lançada, **When** o vendedor confirma o lançamento, **Then** a venda registra data/hora e o usuário responsável.

---

### User Story 2 - Aplicar as formas de pagamento e o acréscimo do cartão (Priority: P2)
Na venda, aparecem apenas as formas de pagamento habilitadas nas Configurações (Dinheiro, PIX, Cartão). Vendas pagas com Cartão somam o acréscimo configurado (R$ por unidade) ao preço; Dinheiro e PIX usam o preço normal.

**Why this priority**: Define o valor efetivamente cobrado por forma de pagamento — erro aqui significa caixa divergente e perda de margem; a forma Fiado não existe no sistema.

**Independent Test**: Configurar as formas aceitas e o acréscimo do cartão, lançar a mesma venda em cada forma e conferir que apenas as habilitadas aparecem e que o Cartão aplica o acréscimo.

**Acceptance Scenarios**:

1. **Given** as Configurações com Dinheiro, PIX e Cartão habilitados, **When** o vendedor abre a lista de formas de pagamento, **Then** apenas as formas habilitadas aparecem.
2. **Given** um acréscimo de cartão configurado em R$ por unidade, **When** a venda é paga com Cartão, **Then** o total soma o acréscimo por unidade vendida.
3. **Given** a mesma venda paga com Dinheiro ou PIX, **When** o total é calculado, **Then** é usado o preço normal, sem acréscimo.

---

### User Story 3 - Bloquear venda sem estoque suficiente (Priority: P3)
O sistema bloqueia a venda quando a quantidade solicitada é maior que o estoque de cheios disponível, evitando venda de produto que não existe fisicamente.

**Why this priority**: Protege a integridade do estoque (nunca negativo) e evita comprometer o cliente com uma venda que não pode ser entregue; é uma validação de proteção sobre o fluxo principal.

**Independent Test**: Tentar lançar uma venda com quantidade maior que o estoque de cheios do produto e verificar que o sistema bloqueia com mensagem clara.

**Acceptance Scenarios**:

1. **Given** um produto com 10 cheios em estoque, **When** o vendedor tenta lançar uma venda de 11 unidades, **Then** o sistema bloqueia o lançamento.
2. **Given** um produto com 10 cheios em estoque, **When** o vendedor lança uma venda de exatamente 10 unidades, **Then** o lançamento é permitido.

### Edge Cases

- Quantidade igual ao estoque disponível (limite exato) — deve ser permitida.
- Produto com estoque zerado — venda bloqueada com mensagem clara.
- Acréscimo do cartão calculado por unidade e multiplicado pela quantidade, antes da soma do total.
- Forma de pagamento desabilitada nas Configurações após a abertura da tela — a lista deve refletir a configuração vigente.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema deve lançar a venda exigindo produto, quantidade, tipo (Balcão/Entrega) e forma de pagamento.
- **FR-002**: O sistema deve calcular automaticamente o total como quantidade × preço de venda.
- **FR-003**: O sistema deve exibir apenas as formas de pagamento habilitadas nas Configurações (Dinheiro, PIX, Cartão).
- **FR-004**: O sistema deve somar o acréscimo configurado (R$ por unidade) ao preço nas vendas pagas com Cartão; Dinheiro e PIX usam o preço normal.
- **FR-005**: O sistema deve bloquear a venda quando a quantidade solicitada exceder o estoque de cheios.
- **FR-006**: O sistema deve registrar data/hora e o usuário responsável em cada venda lançada.

### Key Entities

- **Venda**: lançamento com produto, quantidade, tipo (Balcão/Entrega), forma de pagamento, total calculado, data/hora e usuário responsável.
- **Forma de Pagamento**: Dinheiro, PIX ou Cartão (Crédito/Débito como forma única), com habilitação configurável e acréscimo próprio para Cartão.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% dos lançamentos concluídos em poucos segundos (meta de usabilidade da portaria).
- **SC-002**: 100% dos totais calculados corretamente (quantidade × preço, com acréscimo quando Cartão).
- **SC-003**: Zero vendas confirmadas com estoque negativo ou acima do disponível.
- **SC-004**: 100% das vendas registram data/hora e usuário responsável.

## Assumptions

- As Configurações definem as formas de pagamento aceitas e o acréscimo do cartão (RF-052).
- A forma Fiado não existe no sistema (RGN-002).
- O preço de venda é cadastrado por produto (RF-002).
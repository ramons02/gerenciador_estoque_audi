# Feature Specification: Recebimento de Vasilhames Vazios Avulsos

**Feature Branch**: `HU-011-Recebimento-Vazios`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como vendedor, quero registrar devoluções de vazios fora de venda (cliente devolve vasilhame sem comprar), para controlar o saldo do pátio."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Lançamento de recebimento avulso de vazios (Priority: P1)
O vendedor lança um recebimento de vasilhames vazios avulsos, indicando o produto e, opcionalmente, o cliente que está devolvendo. O sistema aceita o lançamento sem exigir uma venda associada, pois a devolução acontece fora de uma compra.

**Why this priority**: É a jornada principal da HU — sem o lançamento do recebimento avulso não há como controlar o saldo do pátio de vazios, que é o objetivo central da história.

**Independent Test**: Estando logado como vendedor, acessar a tela de recebimento de vazios, informar o produto e a quantidade, confirmar e verificar que o saldo de vazios do produto aumentou na quantidade lançada.

**Acceptance Scenarios**:

1. **Given** um produto cadastrado e o vendedor na tela de recebimento de vazios, **When** o vendedor informa o produto, a quantidade e confirma, **Then** o sistema registra o lançamento e o pátio de vazios do produto aumenta em N (CT-002).
2. **Given** a tela de recebimento de vazios aberta, **When** o vendedor informa apenas o produto e a quantidade sem selecionar cliente, **Then** o sistema aceita o lançamento normalmente (cliente é opcional) (CT-001).
3. **Given** a tela de recebimento de vazios aberta, **When** o vendedor informa o produto, a quantidade e seleciona um cliente opcionalmente, **Then** o sistema registra o lançamento vinculado ao cliente informado (CT-001).

---

### User Story 2 - Baixa do comodato do cliente (Priority: P1)
Quando o cliente devolve vasilhames e ele possui vasilhames "em rua" registrados (comodato), a devolução reduz o saldo em rua dele, além de incrementar o pátio de vazios.

**Why this priority**: A baixa do comodato é o que mantém o saldo "em rua" por cliente fiel à realidade (RF-028); sem ela o controle de vasilhames em poder dos clientes ficaria inflado.

**Independent Test**: Registrar um cliente com vasilhames "em rua" em aberto, lançar a devolução desses vasilhames e verificar que o saldo em rua do cliente baixou na quantidade devolvida.

**Acceptance Scenarios**:

1. **Given** um cliente com vasilhames "em rua" em aberto, **When** o vendedor lança a devolução desses vazios vinculada ao cliente, **Then** o sistema baixa o comodato do cliente (RF-028) e incrementa o pátio de vazios (CT-003).
2. **Given** um cliente sem vasilhames "em rua" em aberto, **When** o vendedor lança a devolução vinculada a esse cliente, **Then** o sistema apenas incrementa o pátio de vazios, sem alterar saldo em rua (que não existe para ele) (CT-003).

---

### User Story 3 - Rastreabilidade do lançamento (Priority: P2)
Todo recebimento de vazios avulsos fica registrado com data/hora e o usuário responsável pelo lançamento, garantindo auditoria das movimentações do pátio.

**Why this priority**: O registro de data/hora e usuário dá rastro de auditoria (alinhado ao RNF-007) e permite identificar quem e quando realizou cada devolução; é essencial, porém secundário ao fluxo de lançamento em si.

**Independent Test**: Realizar um lançamento de devolução e consultar o histórico da movimentação verificando que data/hora e usuário estão preenchidos corretamente.

**Acceptance Scenarios**:

1. **Given** um lançamento de recebimento de vazios confirmado, **When** o vendedor consulta o registro da movimentação, **Then** o sistema exibe a data/hora do lançamento e o usuário responsável (CT-004).

### Edge Cases

- Cliente devolve quantidade maior do que o saldo de vazios "em rua" que possui em aberto — a baixa do comodato não deve gerar saldo em rua negativo; o excedente apenas entra no pátio.
- Lançamento sem cliente informado (cliente opcional) não deve perder a possibilidade de vinculação posterior.
- A devolução de vazios avulsos não deve confundir o estado do produto: o vasilhame devolvido entra no pátio como "vazio", nunca como "cheio".

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema deve permitir lançar recebimento de vasilhames vazios avulsos (devolução fora de venda) informando produto e quantidade, com o cliente como campo opcional (RF-027, CT-001).
- **FR-002**: Ao confirmar o lançamento, o sistema deve incrementar o saldo de vazios (pátio) do produto na quantidade informada (CT-002).
- **FR-003**: Quando o recebimento estiver vinculado a um cliente que possui vasilhames "em rua", o sistema deve baixar o comodato do cliente (RF-028, CT-003).
- **FR-004**: O sistema deve registrar em cada lançamento de recebimento a data/hora e o usuário responsável (CT-004).

### Key Entities *(include if feature involves data)*

- **Produto**: item composto por carga/conteúdo + vasilhame/casco (ex.: Gás P13), sobre o qual o saldo de vazios é controlado.
- **Cliente**: pessoa que devolve os vasilhames; opcional no lançamento; possui controle de vasilhames "em rua" (comodato).
- **Lançamento de Recebimento de Vazio**: registro da devolução avulsa com produto, quantidade, cliente (opcional), data/hora e usuário responsável.
- **Saldo de Vazios (Pátio)**: estado de estoque que recebe o incremento a cada devolução.
- **Saldo Em Rua**: estado por cliente que é baixado quando o vazio é devolvido.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% dos lançamentos de devolução avulsa confirmados refletem imediatamente o incremento no saldo de vazios do produto.
- **SC-002**: 100% das devoluções vinculadas a clientes com comodato em aberto reduzem o saldo "em rua" do cliente na mesma quantidade devolvida.
- **SC-003**: 100% dos lançamentos possuem data/hora e usuário registrados e consultáveis.

## Assumptions

- O cliente é um campo opcional no lançamento (o vendedor pode registrar devoluções sem identificar o cliente, conforme CT-001).
- A devolução avulsa só afeta os estados "Vazios (pátio)" e "Em rua" — nunca o estoque de cheios.
- O recebimento de vazios avulsos depende de produtos cadastrados (RF-001/RF-003) e de clientes cadastrados quando se deseja a baixa do comodato (RF-004).
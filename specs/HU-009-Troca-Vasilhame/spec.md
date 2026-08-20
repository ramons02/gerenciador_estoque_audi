# Feature Specification: Troca de Vasilhame (Venda Normal)

**Feature Branch**: `HU-009-Troca-Vasilhame`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como vendedor, quero registrar a venda com troca (cliente entrega 1 vazio e leva 1 cheio), para que o pátio de vazios seja atualizado automaticamente."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Registrar venda com troca de vasilhame (Priority: P1)
Na venda normal com troca, o cliente entrega 1 vazio e leva 1 cheio. Ao confirmar, o sistema baixa 1 cheio do estoque e adiciona 1 vazio ao pátio por unidade vendida, sem alterar o total da venda — a troca não tem custo para o cliente.

**Why this priority**: É o modelo de negócio central da revenda (casco retornável) — sem ele o pátio de vazios e o estoque ficam desatualizados e o negócio para.

**Independent Test**: Confirmar uma venda com troca de 1 unidade e conferir que o estoque de cheios caiu 1 e o saldo de vazios do pátio subiu 1, com o total da venda inalterado.

**Acceptance Scenarios**:

1. **Given** uma venda com troca de N unidades, **When** o vendedor confirma a venda, **Then** o sistema baixa N cheios do estoque e adiciona N vazios ao pátio.
2. **Given** uma venda com troca, **When** o total da venda é calculado, **Then** o vazio recebido do cliente não altera o total.
3. **Given** a confirmação de uma venda com troca, **When** o saldo de vazios do pátio é consultado, **Then** ele reflete imediatamente o vazio recebido.

---

### User Story 2 - Refletir o saldo de vazios imediatamente (Priority: P2)
O pátio de vazios passa a refletir o vazio recebido do cliente imediatamente após a confirmação da venda com troca, sem ações manuais adicionais.

**Why this priority**: A atualização imediata do pátio é o que mantém o controle de vazios confiável para as devoluções à distribuidora (RDN-003).

**Independent Test**: Após confirmar uma venda com troca, consultar o saldo de vazios do pátio e verificar o incremento sem nenhuma ação adicional.

**Acceptance Scenarios**:

1. **Given** uma venda com troca confirmada, **When** o saldo de vazios do pátio é consultado em seguida, **Then** o vazio recebido já está somado ao saldo.

---

### User Story 3 - Garantir a atomicidade da operação (Priority: P3)
A atualização de estoque da venda com troca (baixar cheio + adicionar vazio) é atômica — nunca pode haver estado parcial, como baixar o cheio sem adicionar o vazio.

**Why this priority**: É a garantia de confiabilidade do fluxo — em operação normal não aparece, mas uma falha no meio da operação sem atomicidade corromperia o estoque permanentemente.

**Independent Test**: Simular falha durante a confirmação (ex.: queda de conexão) e verificar que o estoque permanece consistente — ou a operação completa é aplicada ou nada é aplicado.

**Acceptance Scenarios**:

1. **Given** uma falha durante a confirmação da venda com troca, **When** o sistema tenta aplicar as atualizações de estoque, **Then** ambas as atualizações (cheio e vazio) são aplicadas juntas ou nenhuma é aplicada, sem estado parcial.

### Edge Cases

- Venda com troca de múltiplas unidades — N cheios baixados e N vazios adicionados na mesma operação.
- Pátio já possuindo saldo de vazios — o vazio recebido apenas incrementa o saldo.
- Falha de conexão no meio da confirmação — o estado do estoque deve permanecer consistente (atomicidade).
- Estoque de cheios insuficiente para a venda com troca — a venda é bloqueada antes de qualquer atualização.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema deve, na venda com troca, baixar 1 cheio do estoque e adicionar 1 vazio ao pátio por unidade vendida.
- **FR-002**: O sistema deve manter a operação de atualização de estoque da troca atômica, nunca permitindo estado parcial.
- **FR-003**: O sistema não deve alterar o total da venda pelo vazio recebido do cliente (troca sem custo).
- **FR-004**: O sistema deve refletir o saldo de vazios do pátio imediatamente após a confirmação da venda com troca.

### Key Entities

- **Venda com Troca**: venda normal em que cada unidade vendida adiciona 1 vazio ao pátio, sem custo para o cliente.
- **Pátio de Vazios**: saldo de cascos vazios disponíveis, atualizado automaticamente a cada troca.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% das vendas com troca geram −1 cheio e +1 vazio por unidade vendida.
- **SC-002**: Zero ocorrências de estado parcial de estoque (cheio baixado sem vazio adicionado).
- **SC-003**: 100% dos saldos de vazios do pátio refletem o recebido imediatamente após a confirmação.

## Assumptions

- A troca é 1:1 — 1 vazio por unidade vendida (RF-023).
- O estoque nunca fica negativo em nenhum estado (RDN-005).
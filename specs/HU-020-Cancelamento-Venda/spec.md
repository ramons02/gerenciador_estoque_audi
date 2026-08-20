# Feature Specification: Cancelamento/Estorno de Venda

**Feature Branch**: `HU-020-Cancelamento-Venda`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como administrador, quero cancelar/estornar uma venda com reversão automática de estoque e caixa, para corrigir lançamentos errados mantendo o histórico."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Cancelar/estornar uma venda com reversão automática (Priority: P1)
O administrador cancela uma venda lançada errada. O sistema reverte automaticamente o estoque (o cheio volta ao estoque de cheios e o vazio sai do pátio) e o caixa (estorno do valor conforme a forma de pagamento), corrigindo o lançamento sem necessidade de ajustes manuais.

**Why this priority**: É a finalidade central da HU — corrigir lançamentos errados de forma automática, sem risco de estoque ou caixa divergentes.

**Independent Test**: Registrar uma venda com troca, cancelá-la em seguida e conferir que o cheio voltou ao estoque de cheios, o vazio saiu do pátio e o valor foi estornado do caixa.

**Acceptance Scenarios**:

1. **Given** uma venda com troca de vasilhame registrada, **When** o administrador cancela a venda, **Then** o estoque é revertido automaticamente: o cheio volta ao estoque de cheios e o vazio sai do pátio.
2. **Given** uma venda registrada com uma forma de pagamento, **When** o administrador cancela a venda, **Then** o caixa é revertido automaticamente, estornando o valor da venda.
3. **Given** uma venda registrada, **When** o administrador cancela a venda, **Then** o registro permanece no histórico com o status "cancelado" e não é apagado.

---

### User Story 2 - Manter o histórico com status e rastro de auditoria (Priority: P2)
O administrador localiza a venda cancelada no histórico e identifica o cancelamento com data/hora e usuário responsável, garantindo rastreabilidade de quem corrigiu o lançamento.

**Why this priority**: Manter o histórico íntegro e auditável é pré-requisito de integridade do sistema — sem isso o cancelamento perderia confiança.

**Independent Test**: Cancelar uma venda e abrir o histórico, conferindo que a venda aparece com status "cancelado", data/hora do cancelamento e o usuário responsável.

**Acceptance Scenarios**:

1. **Given** uma venda cancelada, **When** o administrador consulta o histórico de vendas, **Then** a venda permanece no histórico com o status "cancelado".
2. **Given** uma venda cancelada, **When** o administrador consulta os detalhes do cancelamento, **Then** o registro exibe a data/hora e o usuário responsável pelo cancelamento.

---

### User Story 3 - Excluir vendas canceladas dos totais de relatórios e dashboard (Priority: P3)
As vendas canceladas não entram nos totais de relatórios (vendas, carregamentos, balanço) nem do dashboard, para que os totais reflitam apenas as vendas válidas.

**Why this priority**: O efeito sobre relatórios e dashboard é consequência do cancelamento e não bloqueia a correção do lançamento — por isso fica por último.

**Independent Test**: Cancelar uma venda e conferir que ela não aparece nas linhas do relatório de vendas nem soma nos totais do relatório e do dashboard do dia.

**Acceptance Scenarios**:

1. **Given** uma venda cancelada em um período, **When** o administrador gera o relatório de vendas do período, **Then** a venda cancelada não aparece nas linhas nem nos totais do relatório.
2. **Given** uma venda cancelada no dia, **When** o administrador abre o dashboard do dia, **Then** os totais do dashboard não incluem a venda cancelada.

### Edge Cases

- Tentativa de cancelar uma venda já cancelada é recusada pelo sistema.
- Cancelamento de venda de vasilhame novo (sem devolução de vazio) reverte o casco "em rua" de volta ao estoque de cheios.
- A reversão nunca gera estoque negativo em nenhum estado do produto.
- O cancelamento exige usuário autenticado; o responsável é sempre registrado.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O cancelamento de venda deve reverter automaticamente o estoque: o cheio volta ao estoque de cheios e o vazio sai do pátio.
- **FR-002**: O cancelamento de venda deve reverter automaticamente o caixa, estornando o valor da venda conforme a forma de pagamento.
- **FR-003**: A venda cancelada deve permanecer no histórico com status "cancelado", sem ser apagada.
- **FR-004**: O cancelamento deve registrar a data/hora e o usuário responsável pela operação.
- **FR-005**: Relatórios (vendas, carregamentos e balanço) e dashboard devem excluir as vendas canceladas dos totais.
- **FR-006**: O sistema deve recusar o cancelamento de uma venda que já esteja cancelada.

### Key Entities *(include if feature involves data)*

- **Venda**: registro de venda com itens (produto, quantidade, valor), forma de pagamento, tipo, data/hora e status (ativo/"cancelado").
- **Cancelamento**: operação que reverte estoque e caixa, registrada com data/hora e usuário responsável, mantendo o histórico da venda.
- **Estoque de Produto**: estados de cheios, vazios e em rua que a reversão ajusta automaticamente.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Todo cancelamento reverte estoque e caixa de forma automática, sem necessidade de ajuste manual posterior.
- **SC-002**: Nenhuma venda cancelada é apagada do histórico; 100% dos cancelamentos possuem data/hora e usuário responsável.
- **SC-003**: Relatórios e dashboard refletem somente vendas válidas; vendas canceladas não aparecem em nenhum total.

## Assumptions

- Somente usuário autenticado com permissão de administrador pode cancelar uma venda.
- As vendas canceladas permanecem consultáveis no histórico por pelo menos 12 meses, junto às demais movimentações.
- A regra de exclusão de canceladas nos totais é aplicada por relatórios e dashboard de forma consistente.

# Feature Specification: Dashboard (Resumo do Dia)

**Feature Branch**: `HU-019-Dashboard-Resumo-Dia`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como administrador, quero ver na tela inicial o resumo do dia (faturamento, formas de pagamento e unidades vendidas), para saber como foi o dia de um relance."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Ver o faturamento do dia e os totais por forma de pagamento (Priority: P1)
Ao abrir a tela inicial, o administrador vê o Total Faturado no Dia (R$) e o total por forma de pagamento — Dinheiro, PIX e Cartão (crédito e débito somados em um único valor) —, sabendo como foi o dia de um relance e conferindo o caixa sem abrir relatórios.

**Why this priority**: O faturamento do dia e a divisão por forma de pagamento são o resumo mais importante para o administrador conferir o dia na tela inicial.

**Independent Test**: Abrir a tela inicial com vendas registradas no dia e conferir que o Total Faturado no Dia (R$) e os totais de Dinheiro, PIX e Cartão coincidem com as vendas do dia, com Cartão somando crédito e débito em um único valor.

**Acceptance Scenarios**:

1. **Given** vendas registradas no dia, **When** o administrador abre a tela inicial, **Then** o dashboard exibe o Total Faturado no Dia em R$.
2. **Given** vendas registradas no dia em Dinheiro, PIX e Cartão, **When** o administrador abre a tela inicial, **Then** o dashboard exibe o total por forma de pagamento, com Cartão representando crédito e débito somados em um único valor.
3. **Given** um dia sem vendas, **When** o administrador abre a tela inicial, **Then** o dashboard exibe valores zerados, sem erro.

---

### User Story 2 - Ver as unidades vendidas no dia (Priority: P2)
O administrador vê na tela inicial o total de botijões/galões vendidos no dia, tanto por produto quanto o total geral, para avaliar o volume de vendas do dia de um relance.

**Why this priority**: O volume de unidades vendidas complementa o resumo financeiro, mas é secundário ao faturamento (US1).

**Independent Test**: Abrir a tela inicial com vendas do dia de múltiplos produtos e conferir que o total por produto e o total geral de botijões/galões coincidem com as vendas registradas.

**Acceptance Scenarios**:

1. **Given** vendas de botijões/galões registradas no dia, **When** o administrador abre a tela inicial, **Then** o dashboard exibe o total de unidades vendidas por produto.
2. **Given** vendas de botijões/galões registradas no dia, **When** o administrador abre a tela inicial, **Then** o dashboard exibe o total geral de unidades vendidas no dia.

---

### User Story 3 - Ver os alertas de estoque baixo ativos (Priority: P3)
O administrador vê na tela inicial os alertas de estoque baixo ativos — produtos cujo saldo de cheios está no limite mínimo configurado ou abaixo dele — para saber de imediato o que precisa ser reposto.

**Why this priority**: Os alertas ajudam na reposição, mas não fazem parte do resumo financeiro central (US1/US2) — por isso ficam por último.

**Independent Test**: Abrir a tela inicial com um produto abaixo do limite mínimo configurado e conferir que o alerta é exibido; reabastecer o produto e conferir que o alerta some.

**Acceptance Scenarios**:

1. **Given** um produto com saldo de cheios no limite mínimo configurado ou abaixo, **When** o administrador abre a tela inicial, **Then** o dashboard exibe o alerta de estoque baixo desse produto.
2. **Given** um alerta de estoque baixo ativo e uma nova entrada de caminhão, **When** o administrador abre a tela inicial, **Then** o alerta do produto reabastecido deixa de ser exibido.

### Edge Cases

- Dia sem vendas: todos os totais exibem zero, sem erro.
- Forma de pagamento sem movimentação no dia: exibida com valor zero.
- Produto sem vendas no dia não aparece nas unidades vendidas.
- Alertas de estoque baixo são exibidos somente para produtos no limite mínimo configurado ou abaixo.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O dashboard deve exibir o Total Faturado no Dia (R$) na tela inicial.
- **FR-002**: O dashboard deve exibir o total por forma de pagamento no dia — Dinheiro, PIX e Cartão, com crédito e débito somados em um único valor.
- **FR-003**: O dashboard deve exibir o total de botijões/galões vendidos no dia, por produto e total geral.
- **FR-004**: O dashboard deve exibir os alertas de estoque baixo ativos (produtos no limite mínimo configurado ou abaixo).
- **FR-005**: Vendas canceladas não devem ser somadas nos totais do dashboard.

### Key Entities *(include if feature involves data)*

- **Resumo do Dia**: agregação diária de total faturado, totais por forma de pagamento (Dinheiro, PIX, Cartão) e unidades vendidas (por produto e total geral).
- **Alerta de Estoque Baixo**: aviso gerado para produtos com saldo de cheios no limite mínimo configurado ou abaixo.
- **Venda**: fonte dos totais do dia, considerando apenas vendas válidas (não canceladas) do dia atual.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: O administrador vê o resumo do dia (faturamento, formas de pagamento e unidades vendidas) na tela inicial, sem navegar para outro módulo.
- **SC-002**: Os totais do dashboard coincidem com o relatório de vendas do mesmo dia.
- **SC-003**: Os alertas de estoque baixo exibidos refletem o limite mínimo configurado por produto no momento da visualização.

## Assumptions

- As vendas do dia estão registradas no sistema com data/hora, forma de pagamento e quantidade por produto.
- A tela inicial é o primeiro conteúdo visto pelo administrador ao entrar no sistema.
- O limite mínimo de estoque por produto é configurado no cadastro do produto e alimenta o alerta de estoque baixo.

# Feature Specification: Limite Mínimo de Estoque (Alerta)

**Feature Branch**: `HU-003-Limite-Estoque-Baixo`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como administrador, quero definir um limite mínimo de estoque de cheios por produto, para que o sistema me avise quando for hora de comprar mais."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Definição do limite mínimo de cheios por produto (Priority: P1)
O administrador define, por produto, um limite mínimo de estoque de cheios (ex.: 20 cheios de Gás P13). Esse limite é a referência usada pelo sistema para avisar quando for hora de comprar mais.

**Why this priority**: Sem o limite configurado, o sistema não tem referência para gerar o alerta de reposição — é a base da feature.

**Independent Test**: Cadastrar o limite mínimo de 20 cheios para um produto e verificar que o valor é salvo e exibido no cadastro.

**Acceptance Scenarios**:

1. **Given** o cadastro de um produto, **When** o administrador informa um limite mínimo de cheios, **Then** o sistema salva o limite junto ao produto
2. **Given** um produto com limite mínimo definido, **When** o administrador consulta o cadastro, **Then** o sistema exibe o limite mínimo configurado

---

### User Story 2 - Alerta visual quando o estoque atinge ou fica abaixo do limite (Priority: P1)
Quando o saldo de cheios de um produto atinge ou fica abaixo do limite mínimo configurado, o sistema emite um alerta visual persistente (RF-032), visível no painel, avisando que é hora de comprar mais.

**Why this priority**: O alerta é o propósito central da feature — sem ele, o administrador não é avisado a tempo de repor o estoque e pode perder vendas.

**Independent Test**: Com o limite de 20 cheios configurado, baixar o estoque para 20 ou menos (via venda) e verificar que o alerta visual aparece no painel e permanece exibido.

**Acceptance Scenarios**:

1. **Given** um produto com limite mínimo de 20 cheios, **When** o saldo de cheios atinge exatamente 20, **Then** o sistema emite alerta visual de estoque baixo
2. **Given** um produto com limite mínimo de 20 cheios, **When** o saldo de cheios fica abaixo de 20, **Then** o sistema mantém o alerta visual de estoque baixo ativo
3. **Given** um alerta de estoque baixo ativo, **When** o administrador navega pelo sistema, **Then** o alerta permanece visível (persistente) até a condição de dispensa ser atendida

---

### User Story 3 - Dispensa do alerta após nova entrada de caminhão (Priority: P2)
O alerta de estoque baixo só é dispensado quando uma nova entrada de caminhão eleva o estoque de cheios acima do limite mínimo. Nenhuma outra ação (ex.: baixar manualmente, alterar o limite) dispensa o alerta.

**Why this priority**: Garante que o aviso persista até a reposição real acontecer, evitando que um alerta desapareça sem o estoque ser recomposto.

**Independent Test**: Com o alerta ativo, registrar uma entrada de caminhão que eleve o estoque acima do limite e verificar que o alerta desaparece; registrar uma entrada insuficiente e verificar que o alerta permanece.

**Acceptance Scenarios**:

1. **Given** um alerta de estoque baixo ativo, **When** uma entrada de caminhão eleva o estoque de cheios acima do limite, **Then** o sistema dispensa o alerta
2. **Given** um alerta de estoque baixo ativo, **When** uma entrada de caminhão não eleva o estoque acima do limite, **Then** o alerta permanece ativo

### Edge Cases

- Estoque exatamente igual ao limite dispara o alerta (condição "atingir ou ficar abaixo").
- Produtos com limite não definido não geram alerta.
- Múltiplos produtos abaixo do limite geram alertas simultâneos, um por produto.
- O alerta persiste entre sessões/reinícios do sistema até a dispensa por entrada de caminhão.
- Entrada de caminhão parcial, que não eleve o estoque acima do limite, mantém o alerta ativo.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O cadastro do produto deve aceitar um limite mínimo de estoque de cheios (CT-001).
- **FR-002**: O sistema deve emitir alerta visual de estoque baixo quando o saldo de cheios atingir ou ficar abaixo do limite mínimo configurado (RF-032) (CT-002).
- **FR-003**: O alerta deve ser persistente, permanecendo visível até sua dispensa (CT-002).
- **FR-004**: O alerta deve ser dispensado somente após uma nova entrada de caminhão elevar o estoque de cheios acima do limite (CT-003).

### Key Entities *(include if feature involves data)*

- **Produto**: item com limite mínimo de cheios configurado; atributo-chave: limite mínimo.
- **Estoque de cheios**: saldo de itens prontos para venda, comparado ao limite para disparar o alerta.
- **Carregamento**: entrada de caminhão que incrementa os cheios e é a única via de dispensa do alerta.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% dos produtos com saldo de cheios igual ou abaixo do limite exibem o alerta visual.
- **SC-002**: Nenhum alerta é dispensado sem uma entrada de caminhão que eleve o estoque acima do limite.
- **SC-003**: 100% dos produtos possuem limite mínimo configurável no cadastro.

## Assumptions

- O limite mínimo se refere ao estoque de cheios, não a vazios ou "em rua".
- A entrada de caminhão é a única forma de reposição que dispensa o alerta (RGN-004).
- Não há dispensa manual do alerta pelo administrador; alterar o limite não interfere na condição de dispensa.
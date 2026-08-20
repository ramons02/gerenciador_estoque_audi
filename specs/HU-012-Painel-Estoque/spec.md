# Feature Specification: Painel de Estoque em Tempo Real (Pátio)

**Feature Branch**: `HU-012-Painel-Estoque`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como administrador, quero ver o estoque em tempo real por produto (Cheios, Vazios, Em rua), para saber a situação do pátio a qualquer momento."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Visualização do estoque por produto (Priority: P1)
O administrador abre o painel de estoque e visualiza, por produto, os três estados do estoque: Cheios, Vazios no pátio e Em rua (clientes). A visão é a fonte de verdade da situação atual do pátio a qualquer momento.

**Why this priority**: A exibição por produto nos três estados é o requisito central da HU (RF-030) — sem ela o painel não cumpre seu propósito de informar a situação do pátio.

**Independent Test**: Estando logado como administrador, abrir o painel de estoque e confirmar que cada produto exibe os valores de Cheios, Vazios e Em rua.

**Acceptance Scenarios**:

1. **Given** o administrador logado e produtos com estoque registrado, **When** ele abre o painel de estoque, **Then** o painel exibe por produto a quantidade de cheios, vazios no pátio e em rua (CT-001, RF-030).

---

### User Story 2 - Atualização imediata após movimentações (Priority: P1)
Os números do painel refletem cada entrada, venda ou devolução imediatamente, sem necessidade de recarregar a tela, para que a situação do pátio esteja sempre atualizada.

**Why this priority**: O caráter "tempo real" é o diferencial definido na história da HU; sem atualização imediata o painel poderia induzir decisões erradas com base em dados defasados.

**Independent Test**: Realizar uma venda (ou entrada/devolução) e verificar, sem recarregar a tela, que o painel reflete a mudança nos valores do produto afetado.

**Acceptance Scenarios**:

1. **Given** o painel de estoque aberto, **When** uma venda é confirmada para um produto, **Then** o painel atualiza imediatamente os valores de cheios/vazios do produto (CT-002).
2. **Given** o painel de estoque aberto, **When** uma entrada (carregamento) ou devolução é confirmada, **Then** o painel atualiza imediatamente os valores do produto afetado (CT-002).

---

### User Story 3 - Destaque de produtos com estoque baixo (Priority: P2)
Produtos cujo estoque de cheios atingiu ou ficou abaixo do limite mínimo aparecem destacados no painel com alerta visual, chamando a atenção do administrador.

**Why this priority**: O destaque visual é o elo com o alerta de estoque baixo (RF-032/RF-053) e ajuda o administrador a agir preventivamente; porém depende do alerta existir, sendo um complemento à exibição base do painel.

**Independent Test**: Configurar um produto com limite mínimo acima do saldo de cheios atual e verificar que ele aparece destacado com alerta visual no painel.

**Acceptance Scenarios**:

1. **Given** um produto com estoque cheio igual ou abaixo do limite mínimo configurado, **When** o administrador visualiza o painel de estoque, **Then** o produto aparece destacado com alerta visual (CT-003, RF-032).

### Edge Cases

- Produto sem movimentações no dia ainda deve aparecer no painel com seus saldos atuais (inclusive zerados).
- Múltiplas movimentações simultâneas em um mesmo produto devem refletir no painel sem valores intermediários incorretos.
- Produto recém-cadastrado, sem limite mínimo configurado, não deve gerar falso alerta de estoque baixo.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema deve exibir no painel, em tempo real e por produto, as quantidades de Cheios, Vazios (pátio) e Em rua (clientes) (RF-030, CT-001).
- **FR-002**: O painel deve atualizar os valores imediatamente após cada entrada, venda ou devolução, sem recarga manual (CT-002).
- **FR-003**: O painel deve destacar com alerta visual os produtos com estoque cheio igual ou abaixo do limite mínimo configurado (RF-032, CT-003).

### Key Entities *(include if feature involves data)*

- **Produto**: item composto por carga/conteúdo + vasilhame/casco (ex.: Gás P13), exibido no painel com seus três estados de estoque.
- **Cheios**: quantidade pronta para venda, exibida por produto.
- **Vazios (Pátio)**: quantidade de cascos vazios aguardando recarga/devolução, exibida por produto.
- **Em rua**: quantidade de vasilhames em poder dos clientes (comodato), exibida por produto.
- **Limite mínimo de estoque**: parâmetro por produto (RF-003) usado para o destaque de alerta.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% dos produtos com estoque registrado são exibidos no painel com os três estados (cheios, vazios, em rua).
- **SC-002**: Toda movimentação confirmada (entrada, venda ou devolução) reflete no painel em menos de 1 segundo.
- **SC-003**: 100% dos produtos com estoque cheio no/abaixo do limite mínimo exibem destaque visual no painel.

## Assumptions

- O painel é destinado ao perfil administrador (conforme a história da HU).
- O painel consulta o mesmo estoque dos estados Cheios, Vazios e Em rua mantidos pelas demais funcionalidades (RF-011, RF-025, RF-026, RF-027).
- A configuração do limite mínimo por produto (RF-003) já é fornecida por outra feature (HU-003) — o painel apenas a consome para o destaque.
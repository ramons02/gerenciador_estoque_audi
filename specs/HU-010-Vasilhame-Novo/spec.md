# Feature Specification: Venda de Vasilhame Novo (Casco + Carga)

**Feature Branch**: `HU-010-Vasilhame-Novo`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como vendedor, quero vender vasilhame novo (cliente compra o casco + a carga), para registrar a saída do casco do estoque da loja."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Vender vasilhame novo (casco + carga) (Priority: P1)
O vendedor marca a venda como "vasilhame novo" — sem devolução de vazio. O preço é composto pelo preço do casco mais o preço da carga. Ao confirmar, o sistema baixa 1 cheio do estoque e registra 1 vasilhame "em rua" para o cliente.

**Why this priority**: É o fluxo que registra a saída do casco do estoque da loja e cria o controle de comodato — sem ele, a loja perde o rastro dos cascos vendidos.

**Independent Test**: Marcar uma venda como vasilhame novo, conferir que o preço é casco + carga e que, ao confirmar, o estoque de cheios cai 1 e o vasilhame aparece como "em rua" para o cliente.

**Acceptance Scenarios**:

1. **Given** o lançamento de venda aberto, **When** o vendedor marca a venda como "vasilhame novo", **Then** o sistema considera a venda sem devolução de vazio.
2. **Given** um vasilhame novo com preço de casco e preço de carga cadastrados, **When** o total é calculado, **Then** o preço exibido é preço do casco + preço da carga.
3. **Given** uma venda de vasilhame novo de N unidades, **When** o vendedor confirma a venda, **Then** o sistema baixa N cheios do estoque e registra N vasilhames "em rua" para o cliente.

---

### User Story 2 - Identificar o cliente para controle de comodato (Priority: P2)
Como não há devolução de vazio, o sistema solicita o cliente na venda de vasilhame novo, para permitir o controle dos vasilhames em comodato (em rua) por cliente.

**Why this priority**: Sem o cliente identificado não há como rastrear os cascos em rua nem cobrar a devolução — a identificação é pré-requisito do controle de comodato.

**Independent Test**: Tentar concluir uma venda de vasilhame novo sem informar o cliente e verificar que o sistema solicita o cliente antes de concluir.

**Acceptance Scenarios**:

1. **Given** uma venda de vasilhame novo, **When** o vendedor tenta confirmar sem informar o cliente, **Then** o sistema solicita o cliente antes de concluir a venda.

---

### User Story 3 - Acompanhar o vasilhame em rua por cliente (Priority: P3)
Cada vasilhame vendido sem devolução fica registrado como "em rua" para o cliente, permitindo controlar o comodato e dar baixa quando o vazio for devolvido.

**Why this priority**: O acompanhamento em rua é o controle posterior à venda — necessário para conciliar o comodato, mas não bloqueia o lançamento do dia.

**Independent Test**: Após uma venda de vasilhame novo, consultar o controle por cliente e verificar o vasilhame registrado como "em rua".

**Acceptance Scenarios**:

1. **Given** uma venda de vasilhame novo confirmada, **When** o controle de vasilhames do cliente é consultado, **Then** o vasilhame aparece como "em rua" vinculado ao cliente.
2. **Given** um vasilhame "em rua" registrado, **When** o cliente devolve o vazio, **Then** o sistema dá baixa no "em rua" correspondente.

### Edge Cases

- Cliente ainda não cadastrado ao lançar a venda de vasilhame novo — o cadastro deve ser possível antes de concluir a venda.
- Venda de vasilhame novo com múltiplas unidades — N cheios baixados e N vasilhames em rua registrados.
- Vasilhame em rua devolvido posteriormente — a baixa deve ocorrer no registro do cliente correto.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema deve permitir marcar a venda como "vasilhame novo" (sem devolução de vazio).
- **FR-002**: O sistema deve calcular o preço do vasilhame novo como preço do casco + preço da carga.
- **FR-003**: Ao confirmar a venda de vasilhame novo, o sistema deve baixar 1 cheio do estoque e registrar 1 vasilhame "em rua" para o cliente por unidade.
- **FR-004**: O sistema deve solicitar o cliente na venda sem devolução de vazio, para o controle de comodato.

### Key Entities

- **Venda de Vasilhame Novo**: venda sem devolução de vazio, com preço de casco + carga e geração de vasilhame em rua.
- **Vasilhame em Rua**: vasilhame em comodato com o cliente, rastreado por cliente para baixa na devolução.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% das vendas de vasilhame novo registram a saída do cheio e o vasilhame "em rua" por cliente.
- **SC-002**: 100% das vendas de vasilhame novo exigem cliente identificado.
- **SC-003**: 100% dos preços de vasilhame novo exibidos são casco + carga.
- **SC-004**: O saldo "em rua" por cliente confere com as vendas de vasilhame novo não devolvidas.

## Assumptions

- Clientes estão cadastrados para o controle de comodato (RF-004).
- O preço do casco é configurado à parte do preço da carga (RGN-010).
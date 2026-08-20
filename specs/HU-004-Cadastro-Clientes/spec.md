# Feature Specification: Cadastro de Clientes

**Feature Branch**: `HU-004-Cadastro-Clientes`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como vendedor, quero cadastrar clientes com identificação e endereço, para o controle de vasilhames em rua (comodato) e venda de vasilhame novo."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Cadastro de cliente com identificação e endereço (Priority: P1)
O vendedor cadastra clientes informando nome, telefone, endereço e, opcionalmente, documento. O cadastro é a base para o controle de vasilhames em rua (comodato) e para a venda de vasilhame novo.

**Why this priority**: Sem o cliente cadastrado, não é possível controlar vasilhames em rua nem vender vasilhame novo — o cadastro é pré-requisito desses fluxos.

**Independent Test**: Cadastrar um cliente com nome, telefone, endereço e documento e verificar que o registro é salvo e exibido na listagem de clientes.

**Acceptance Scenarios**:

1. **Given** a tela de cadastro de clientes, **When** o vendedor informa nome, telefone, endereço e documento, **Then** o sistema salva o cliente com todos os dados
2. **Given** a tela de cadastro de clientes, **When** o vendedor informa nome, telefone e endereço sem documento, **Then** o sistema salva o cliente com documento em branco (campo opcional)
3. **Given** clientes cadastrados, **When** o vendedor consulta a listagem de clientes, **Then** o sistema exibe os dados de identificação e endereço de cada cliente

---

### User Story 2 - Busca do cliente por nome ou telefone na venda (Priority: P1)
No momento da venda, o vendedor busca o cliente pelo nome ou pelo telefone sem sair do fluxo de venda, agilizando o atendimento na portaria.

**Why this priority**: A busca integrada ao momento da venda é essencial para o lançamento rápido (RNF-001), principalmente em venda de vasilhame novo, onde o cliente é obrigatório.

**Independent Test**: Abrir o lançamento de venda, digitar parte do nome ou o telefone de um cliente cadastrado e verificar que o cliente é encontrado e selecionável na mesma tela.

**Acceptance Scenarios**:

1. **Given** um cliente cadastrado, **When** o vendedor digita o nome do cliente na busca da venda, **Then** o sistema retorna o cliente correspondente
2. **Given** um cliente cadastrado, **When** o vendedor digita o telefone do cliente na busca da venda, **Then** o sistema retorna o cliente correspondente
3. **Given** a busca de cliente na tela de venda, **When** o vendedor seleciona o cliente encontrado, **Then** o cliente é associado à venda em andamento

---

### User Story 3 - Cliente obrigatório na venda de vasilhame novo (Priority: P2)
Na venda de vasilhame novo, o cliente é obrigatório: a venda não pode ser confirmada sem um cliente associado. A forma de pagamento Fiado não existe no sistema (RGN-002) — as vendas são sempre identificadas por cliente quando exigido, sem venda "fiada".

**Why this priority**: O vínculo com o cliente é necessário para registrar o vasilhame como "em rua" (comodato) e cobrar a devolução; a ausência de Fiado simplifica as formas de pagamento aceitas.

**Independent Test**: Tentar confirmar uma venda de vasilhame novo sem selecionar cliente e verificar que o sistema bloqueia; confirmar a mesma venda com cliente selecionado e verificar que é aceita.

**Acceptance Scenarios**:

1. **Given** uma venda de vasilhame novo em andamento, **When** o vendedor tenta confirmar sem selecionar um cliente, **Then** o sistema bloqueia a confirmação por cliente obrigatório
2. **Given** uma venda de vasilhame novo em andamento, **When** o vendedor seleciona um cliente e confirma, **Then** o sistema aceita a venda e vincula o vasilhame ao cliente (em rua)
3. **Given** o lançamento de venda, **When** o vendedor consulta as formas de pagamento, **Then** a forma Fiado não existe nas opções disponíveis

### Edge Cases

- Cliente sem documento é aceito (documento é opcional).
- Busca retorna clientes por nome parcial ou telefone, permitindo seleção na venda.
- Venda de vasilhame novo sem cliente associado nunca é confirmada.
- Nenhuma venda do sistema pode usar a forma de pagamento Fiado (RGN-002).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O cadastro deve capturar nome, telefone, endereço e documento (opcional) do cliente (CT-001).
- **FR-002**: O sistema deve permitir buscar o cliente por nome ou telefone no momento da venda (CT-002).
- **FR-003**: O cliente deve ser obrigatório na venda de vasilhame novo (CT-003).
- **FR-004**: O sistema não deve oferecer a forma de pagamento Fiado (RGN-002) (CT-003).

### Key Entities *(include if feature involves data)*

- **Cliente**: pessoa identificada com dados de contato e endereço; atributos-chave: nome, telefone, endereço, documento (opcional).
- **Venda**: lançamento que associa o cliente, obrigatório na venda de vasilhame novo.
- **Vasilhame em rua**: casco que fica com o cliente (comodato), controlado por cliente e dado baixa na devolução.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% dos clientes possuem nome, telefone e endereço cadastrados.
- **SC-002**: A busca por nome ou telefone encontra o cliente sem sair da tela de venda.
- **SC-003**: Nenhuma venda de vasilhame novo é confirmada sem cliente associado.

## Assumptions

- O documento é facultativo no cadastro do cliente.
- A venda de vasilhame novo sempre exige cliente identificado (RF-024/RF-026).
- As formas de pagamento aceitas são configuráveis e não incluem Fiado (RGN-002, RF-052).
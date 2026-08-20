# Feature Specification: Cadastro de Fornecedores (Distribuidoras)

**Feature Branch**: `HU-005-Cadastro-Fornecedores`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como administrador, quero cadastrar fornecedores/distribuidoras (ex.: Ultragaz, Nacional), para que as entradas de caminhão sejam identificadas."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Cadastro de fornecedor/distribuidora (Priority: P1)
O administrador cadastra fornecedores/distribuidoras informando o nome da distribuidora (ex.: Ultragaz, Nacional) e o contato. Esses registros identificam a origem das entradas de caminhão.

**Why this priority**: Sem o fornecedor cadastrado, a entrada de caminhão não tem origem identificada, comprometendo a rastreabilidade das compras e os relatórios de carregamento.

**Independent Test**: Cadastrar uma distribuidora com nome e contato e verificar que o registro é salvo e exibido na listagem de fornecedores.

**Acceptance Scenarios**:

1. **Given** a tela de cadastro de fornecedores, **When** o administrador informa o nome da distribuidora e o contato, **Then** o sistema salva o fornecedor com os dados informados
2. **Given** fornecedores cadastrados, **When** o administrador consulta a listagem de fornecedores, **Then** o sistema exibe o nome e o contato de cada fornecedor

---

### User Story 2 - Disponibilidade do fornecedor no registro de carregamento (Priority: P2)
O fornecedor cadastrado fica disponível na seleção do registro de carregamento (RF-010): ao registrar a chegada de um caminhão, o administrador escolhe o fornecedor a partir da lista de cadastrados.

**Why this priority**: Conecta o cadastro ao fluxo de entradas — é o uso real do fornecedor, que identifica cada carregamento no estoque e nos relatórios.

**Independent Test**: Abrir o registro de carregamento, verificar que a distribuidora cadastrada aparece na seleção de fornecedor e selecioná-la para salvar o carregamento.

**Acceptance Scenarios**:

1. **Given** um fornecedor cadastrado, **When** o administrador abre o registro de carregamento, **Then** o fornecedor aparece disponível na seleção de fornecedor
2. **Given** o registro de carregamento aberto, **When** o administrador seleciona o fornecedor e salva o carregamento, **Then** o sistema registra a entrada vinculada ao fornecedor selecionado

### Edge Cases

- O contato é mínimo obrigatório do cadastro, mas o funcionamento da seleção no carregamento não depende dele.
- Fornecedor não cadastrado não pode ser usado em um registro de carregamento.
- Fornecedores com o mesmo nome devem ser tratados sem duplicidade na seleção do carregamento.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O cadastro deve capturar o nome da distribuidora e o contato do fornecedor (CT-001).
- **FR-002**: O sistema deve disponibilizar os fornecedores cadastrados na seleção do registro de carregamento (RF-010) (CT-002).
- **FR-003**: O registro de carregamento deve ser vinculado a um fornecedor cadastrado.

### Key Entities *(include if feature involves data)*

- **Fornecedor**: distribuidora identificada no sistema; atributos-chave: nome, contato.
- **Carregamento**: entrada de caminhão que referencia o fornecedor que originou a carga (cheios recebidos e vazios devolvidos).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% dos fornecedores possuem nome e contato cadastrados.
- **SC-002**: 100% dos fornecedores cadastrados ficam disponíveis na seleção do registro de carregamento.
- **SC-003**: Nenhum carregamento é registrado sem fornecedor cadastrado vinculado.

## Assumptions

- O contato é o dado mínimo de identificação do fornecedor além do nome.
- A seleção do fornecedor no carregamento usa a lista de fornecedores cadastrados (RF-010).
- Todo carregamento possui um fornecedor; o cadastro antecede o registro de entradas.
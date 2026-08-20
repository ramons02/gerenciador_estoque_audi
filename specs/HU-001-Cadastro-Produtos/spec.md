# Feature Specification: Cadastro de Produtos (Carga + Vasilhame)

**Feature Branch**: `HU-001-Cadastro-Produtos`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como administrador, quero cadastrar produtos compostos por Carga/Conteúdo e Vasilhame/Casco (ex.: Gás P13 = carga Gás + casco P13), para que o estoque e as vendas reflitam o item real vendido."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Cadastro de produto composto por carga e vasilhame (Priority: P1)
O administrador acessa o cadastro de produtos e informa a carga (ex.: Gás, Água) e o vasilhame (ex.: P13, P45, Galão 20L). O sistema combina os dois campos na definição do item vendido e exibe o produto pelo nome combinado (ex.: "Gás P13").

**Why this priority**: É a operação central da feature — sem o cadastro com a composição carga + vasilhame, nenhum produto existe para o estoque e as vendas refletirem o item real vendido (RDN-001).

**Independent Test**: Cadastrar um produto informando a carga "Gás" e o vasilhame "P13" e verificar que o item "Gás P13" passa a existir no cadastro, exibido pelo nome combinado.

**Acceptance Scenarios**:

1. **Given** um produto sem cadastro, **When** o administrador informa a carga "Gás" e o vasilhame "P13", **Then** o sistema salva o produto e o exibe pelo nome combinado "Gás P13"
2. **Given** o cadastro de produtos aberto, **When** o administrador informa apenas a carga ou apenas o vasilhame, **Then** o sistema não permite salvar até que ambos estejam preenchidos
3. **Given** produtos cadastrados com cargas e vasilhames distintos, **When** o administrador consulta a listagem de produtos, **Then** cada item aparece pelo nome combinado de sua carga e vasilhame (ex.: "Gás P13", "Gás P45", "Água Galão 20L")

---

### User Story 2 - Proteção contra produto duplicado (Priority: P1)
Ao tentar cadastrar um produto com a mesma combinação de carga + vasilhame de um produto já existente, o sistema bloqueia o cadastro, evitando itens duplicados que distorceriam estoque e vendas.

**Why this priority**: Duplicidade quebraria a unicidade do item vendido e corromperia as contagens de estoque e os relatórios — é uma validação que protege a integridade dos dados desde a primeira venda.

**Independent Test**: Cadastrar "Gás P13" e, em seguida, tentar cadastrar novamente a mesma combinação; verificar que o segundo cadastro é bloqueado.

**Acceptance Scenarios**:

1. **Given** um produto "Gás P13" já cadastrado, **When** o administrador tenta cadastrar outro produto com carga "Gás" e vasilhame "P13", **Then** o sistema bloqueia com mensagem de produto duplicado
2. **Given** um produto "Gás P13" já cadastrado, **When** o administrador cadastra "Gás P45" ou "Água P13", **Then** o cadastro é aceito por se tratar de combinações diferentes

---

### User Story 3 - Edição e exclusão condicionada (Priority: P2)
O administrador pode editar um produto existente a qualquer momento. A exclusão, porém, só é permitida quando não houver movimentações (vendas, entradas, devoluções) vinculadas ao produto; com movimentações, o produto não pode ser excluído.

**Why this priority**: Edição é necessária para corrigir dados do cadastro, mas a exclusão é rara e precisa preservar o histórico de movimentações — por isso vem depois das funcionalidades de cadastro e duplicidade.

**Independent Test**: Editar o nome/combinação de um produto sem movimentações; tentar excluir um produto com vendas lançadas e verificar que a exclusão é bloqueada.

**Acceptance Scenarios**:

1. **Given** um produto cadastrado, **When** o administrador altera a carga ou o vasilhame, **Then** o sistema salva a alteração e mantém a unicidade da nova combinação
2. **Given** um produto sem movimentações vinculadas, **When** o administrador tenta excluí-lo, **Then** o sistema permite a exclusão
3. **Given** um produto com vendas ou entradas vinculadas, **When** o administrador tenta excluí-lo, **Then** o sistema bloqueia a exclusão por existirem movimentações vinculadas

### Edge Cases

- Um mesmo vasilhame pode existir com cargas diferentes (ex.: "Gás P13" e "Água P13" são produtos distintos).
- Uma mesma carga pode existir com vasilhames diferentes (ex.: "Gás P13" e "Gás P45" são produtos distintos).
- A exclusão bloqueada por movimentações não pode ser contornada por outra tela do sistema.
- Produto sem carga ou sem vasilhame não pode ser salvo.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O cadastro deve permitir informar a carga/conteúdo (ex.: Gás, Água) e o vasilhame/casco (ex.: P13, P45, Galão 20L) do produto (CT-001).
- **FR-002**: O sistema deve listar o produto pelo nome combinado de carga + vasilhame (ex.: "Gás P13") (CT-002).
- **FR-003**: O sistema deve bloquear o cadastro de produto duplicado com a mesma combinação carga + vasilhame (CT-003).
- **FR-004**: O sistema deve permitir editar um produto existente (CT-004).
- **FR-005**: O sistema deve permitir a exclusão de produto somente quando não houver movimentações vinculadas (CT-004).

### Key Entities *(include if feature involves data)*

- **Produto**: item vendido composto por Carga/Conteúdo + Vasilhame/Casco, exibido pelo nome combinado; atributos-chave: carga, vasilhame, nome combinado.
- **Movimentação**: registros de venda, entrada e devolução que, quando vinculados a um produto, bloqueiam sua exclusão.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% dos produtos cadastrados são exibidos pelo nome combinado de carga + vasilhame.
- **SC-002**: Nenhuma combinação carga + vasilhame duplicada é persistida no cadastro.
- **SC-003**: 100% das tentativas de exclusão de produto com movimentações vinculadas são bloqueadas.

## Assumptions

- A combinação carga + vasilhame é única por produto (RDN-001).
- As cargas têm capacidades fixas, definidas pela distribuidora, e o revendedor apenas as combina com o casco (RDN-007).
- As movimentações vinculadas são preservadas quando a exclusão é bloqueada; nenhuma venda ou entrada pode ser apagada (RNF-007).
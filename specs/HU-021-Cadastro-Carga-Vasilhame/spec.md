# Feature Specification: Cadastro de Carga e Vasilhame

**Feature Branch**: `HU-021-cadastro-carga-vasilhame`

**Created**: 2026-08-20

**Status**: Draft

**Input**: User description: "Como administrador, quero cadastrar um produto que não seja Gás P13 nem Água Galão 20L. Para isso preciso poder cadastrar novas cargas (conteúdos) e novos vasilhames (cascos) diretamente na tela de cadastro de produtos, já que hoje a tela só oferece as cargas Gás/Água e os vasilhames P13/Galão 20L que já existem na base."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Cadastro de carga nova no formulário de produto (Priority: P1)

O administrador, na tela de cadastro de produtos, precisa informar uma carga que ainda não existe na base (ex.: "Refrigerante", "Cerveja", "Água Mineral"). O sistema permite criar a carga nova no momento do cadastro, sem sair da tela, e ela passa a ficar disponível no seletor de cargas.

**Why this priority**: Sem a carga nova não existe produto fora de Gás/Água, que é o motivo central da feature (RDN-001). É o primeiro passo do fluxo.

**Independent Test**: Abrir a tela de produtos, escolher a opção de criar carga nova, informar o nome "Refrigerante" e verificar que a carga passa a aparecer no seletor de cargas e permite cadastrar o produto combinado.

**Acceptance Scenarios**:

1. **Given** a tela de cadastro de produtos aberta, **When** o administrador aciona a opção de criar carga nova e informa o nome "Refrigerante", **Then** a carga "Refrigerante" é salva e fica selecionada no seletor de cargas
2. **Given** a tela de cadastro de produtos aberta, **When** o administrador tenta criar uma carga com nome vazio, **Then** o sistema não permite salvar até que um nome seja informado
3. **Given** a carga "Gás" já cadastrada, **When** o administrador tenta criar uma carga com o mesmo nome "Gás", **Then** o sistema bloqueia com mensagem de carga duplicada

---

### User Story 2 - Cadastro de vasilhame novo no formulário de produto (Priority: P1)

O administrador, na tela de cadastro de produtos, precisa informar um vasilhame que ainda não existe na base (ex.: "Lata 350ml", "Garrafa 1L", "P45"). O sistema permite criar o vasilhame novo no momento do cadastro, sem sair da tela, e ele passa a ficar disponível no seletor de vasilhames.

**Why this priority**: O produto é sempre a combinação carga + vasilhame (RDN-001). Sem o vasilhame novo, a carga nova não vira produto vendável. Mesma prioridade da carga por ser metade da combinação.

**Independent Test**: Abrir a tela de produtos, escolher a opção de criar vasilhame novo, informar o nome "Lata 350ml" (e o preço do casco, se houver) e verificar que o vasilhame passa a aparecer no seletor de vasilhames.

**Acceptance Scenarios**:

1. **Given** a tela de cadastro de produtos aberta, **When** o administrador aciona a opção de criar vasilhame novo e informa o nome "Lata 350ml", **Then** o vasilhame "Lata 350ml" é salvo e fica selecionado no seletor de vasilhames
2. **Given** a tela de cadastro de produtos aberta, **When** o administrador tenta criar um vasilhame com nome vazio, **Then** o sistema não permite salvar até que um nome seja informado
3. **Given** o vasilhame "P13" já cadastrado, **When** o administrador tenta criar um vasilhame com o mesmo nome "P13", **Then** o sistema bloqueia com mensagem de vasilhame duplicado

---

### User Story 3 - Cadastro do produto combinado com carga e vasilhame novos (Priority: P1)

Com a carga e o vasilhame novos cadastrados, o administrador completa o cadastro do produto (preço de custo, preço de venda e limite mínimo) e o sistema salva o item exibindo-o pelo nome combinado de carga + vasilhame.

**Why this priority**: É a entrega final da feature: o produto novo (ex.: "Refrigerante Lata 350ml") passa a existir no cadastro, no estoque e nas vendas. Depende das US1 e US2.

**Independent Test**: Cadastrar a carga "Refrigerante" e o vasilhame "Lata 350ml", informar preços e limite, salvar o produto e verificar que "Refrigerante Lata 350ml" aparece na listagem de produtos.

**Acceptance Scenarios**:

1. **Given** a carga "Refrigerante" e o vasilhame "Lata 350ml" criados, **When** o administrador informa preço de custo, preço de venda e limite mínimo e confirma o cadastro, **Then** o produto é salvo e aparece na listagem como "Refrigerante Lata 350ml"
2. **Given** um produto "Refrigerante Lata 350ml" já cadastrado, **When** o administrador tenta cadastrar novamente a mesma combinação carga + vasilhame, **Then** o sistema bloqueia com mensagem de produto duplicado

---

### Edge Cases

- Carga nova sem vasilhame correspondente: o seletor de vasilhame não é preenchido automaticamente e o administrador precisa criar ou escolher o vasilhame manualmente.
- Preço do casco: um vasilhame novo pode ter preço de casco informado ou iniciar com valor zero, sem bloquear o cadastro.
- Nome duplicado com diferença de maiúsculas/minúsculas (ex.: "gas" vs "Gás"): o sistema trata pelo nome exato informado.
- Produto novo com carga que não é Gás: a regra de acréscimo do cartão continua valendo somente para a carga Gás (RF-021-A), sem afetar os produtos novos.
- Erro do servidor ao salvar carga/vasilhame: o sistema mostra a mensagem de erro e mantém a tela de cadastro aberta, sem perder os dados já informados.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema deve permitir cadastrar uma carga nova a partir da tela de cadastro de produtos (CT-001).
- **FR-002**: O sistema deve permitir cadastrar um vasilhame novo a partir da tela de cadastro de produtos, com nome e preço do casco (CT-002).
- **FR-003**: Após criar, a carga ou o vasilhame novo deve ficar selecionado no seletor do formulário, permitindo continuar o cadastro do produto (CT-003).
- **FR-004**: O sistema deve bloquear o cadastro de carga ou vasilhame com nome já existente na base (CT-004).
- **FR-005**: O sistema deve bloquear o cadastro de carga ou vasilhame com nome vazio (CT-005).
- **FR-006**: O sistema deve permitir o cadastro do produto combinado com carga e vasilhame novos, exibindo-o pelo nome combinado (CT-006).

### Key Entities *(include if feature involves data)*

- **Carga**: conteúdo do produto (ex.: Gás, Água, Refrigerante); nome único na base.
- **Vasilhame**: casco/embalagem do produto (ex.: P13, Galão 20L, Lata 350ml); nome único e preço do casco.
- **Produto**: combinação única de carga + vasilhame, exibida pelo nome combinado; atributos-chave: preço de custo, preço de venda, limite mínimo.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% das cargas novas criadas pela tela de produtos ficam disponíveis no seletor de cargas imediatamente após a criação.
- **SC-002**: 100% dos vasilhames novos criados pela tela de produtos ficam disponíveis no seletor de vasilhames imediatamente após a criação.
- **SC-003**: 100% dos cadastros de carga ou vasilhame com nome duplicado são bloqueados com mensagem clara.
- **SC-004**: O administrador consegue cadastrar um produto novo completo (carga + vasilhame + preços) sem sair da tela de produtos.

## Assumptions

- As cargas e vasilhames podem ser criados somente pela tela de produtos; telas de manutenção dedicadas ficam fora de escopo desta versão.
- A validação de nome duplicado considera o nome exato informado, sem normalização de maiúsculas/minúsculas.
- O preço do casco de um vasilhame novo pode ser zero no momento da criação.
- A regra de acréscimo do cartão (RF-021-A) permanece restrita à carga Gás, não se estendendo às cargas novas.
- O backend já possui os recursos de criação de carga e vasilhame; a feature entrega a interface na tela de produtos.
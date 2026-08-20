# Feature Specification: Cadastro de Preços (Custo e Venda)

**Feature Branch**: `HU-002-Cadastro-Precos`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como administrador, quero cadastrar preço de custo e preço de venda por produto, para que as vendas e relatórios usem os valores corretos."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Cadastro de preço de custo e preço de venda (Priority: P1)
O administrador cadastra, por produto, o preço de custo e o preço de venda em reais (R$). As vendas e os relatórios passam a usar esses valores corretos.

**Why this priority**: Sem os preços cadastrados, a venda não tem valor a cobrar e os relatórios de faturamento, margem e fechamento de caixa ficam incorretos — é pré-requisito para o fluxo de vendas.

**Independent Test**: Cadastrar um produto com preço de custo R$ 50,00 e preço de venda R$ 80,00 e verificar que os dois valores são salvos no cadastro do produto.

**Acceptance Scenarios**:

1. **Given** o cadastro de um produto, **When** o administrador informa preço de custo e preço de venda em R$, **Then** o sistema salva ambos os valores junto ao produto
2. **Given** um produto já cadastrado, **When** o administrador consulta o cadastro, **Then** o sistema exibe o preço de custo e o preço de venda atuais

---

### User Story 2 - Validação de preço de venda não inferior ao custo (Priority: P1)
Ao cadastrar ou alterar preços, o sistema valida que o preço de venda não seja inferior ao preço de custo (RGN-005), bloqueando valores que gerariam venda com prejuízo sem autorização.

**Why this priority**: Protege a margem do negócio — um preço de venda abaixo do custo lançado por engano contaminaria vendas e relatórios financeiros.

**Independent Test**: Tentar salvar um produto com preço de venda abaixo do preço de custo e verificar que o sistema bloqueia com mensagem de validação.

**Acceptance Scenarios**:

1. **Given** um produto com preço de custo de R$ 50,00, **When** o administrador informa preço de venda inferior a R$ 50,00, **Then** o sistema bloqueia o salvamento por preço de venda inferior ao custo
2. **Given** um produto com preço de custo de R$ 50,00, **When** o administrador informa preço de venda igual ou superior a R$ 50,00, **Then** o sistema aceita o salvamento

---

### User Story 3 - Alteração de preço sem recalcular vendas antigas (Priority: P2)
O preço pode ser alterado a qualquer momento. Quando o preço muda, o sistema mantém o preço praticado no momento em que cada venda foi lançada, sem recalcular vendas antigas.

**Why this priority**: Garante a fidelidade do histórico financeiro — vendas passadas precisam refletir o que foi cobrado na época, não o preço atual.

**Independent Test**: Alterar o preço de venda de um produto após lançar uma venda e verificar que a venda antiga mantém o preço original.

**Acceptance Scenarios**:

1. **Given** um produto com preço de venda R$ 80,00 e uma venda lançada com esse preço, **When** o administrador altera o preço de venda para R$ 90,00, **Then** a venda já lançada permanece com o preço de R$ 80,00
2. **Given** um produto com preço de venda alterado, **When** uma nova venda é lançada, **Then** a nova venda usa o preço de venda vigente no momento do lançamento

### Edge Cases

- Preço de venda igual ao preço de custo é permitido (a validação proíbe apenas venda inferior ao custo).
- Preço de venda abaixo do custo só é aceito se o administrador autorizar (RGN-005).
- Valores zerados, negativos ou não numéricos devem ser bloqueados no campo de preço.
- Vendas lançadas antes de uma alteração de preço nunca são recalculadas.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O cadastro do produto deve aceitar preço de custo e preço de venda em reais (R$) (CT-001).
- **FR-002**: O sistema deve validar que o preço de venda não seja inferior ao preço de custo (RGN-005) (CT-002).
- **FR-003**: O sistema deve permitir alterar o preço de custo e o preço de venda a qualquer momento (CT-003).
- **FR-004**: O sistema deve manter o preço da venda no momento em que ela é lançada, sem recalcular vendas antigas após alteração (CT-003).

### Key Entities *(include if feature involves data)*

- **Produto**: item com preço de custo e preço de venda cadastrados em R$; atributos-chave: preço de custo, preço de venda.
- **Venda**: registro que captura o preço vigente no momento do lançamento, imutável após a alteração de preço.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% dos produtos possuem preço de custo e preço de venda cadastrados.
- **SC-002**: Nenhum produto é salvo com preço de venda inferior ao preço de custo sem autorização.
- **SC-003**: Nenhuma venda lançada tem seu preço recalculado após alteração do preço do produto.

## Assumptions

- A validação de venda inferior ao custo segue a RGN-005, com exceção mediante autorização do administrador.
- Os valores monetários são tratados em reais (R$).
- A alteração de preço vale apenas para lançamentos futuros; vendas antigas não são retroativamente alteradas.
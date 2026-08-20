# Pesquisa Técnica: Cadastro de Fornecedores (Distribuidoras)

**HU**: HU-005 - Cadastro de Fornecedores (Distribuidoras)
**Fase**: 0 (decisões de arquitetura)
**Branch**: `HU-005-Cadastro-Fornecedores`

Formato por decisão: **Decision** / **Rationale** / **Alternatives considered**.

---

## Decisão 1: tab_fornecedor com nome obrigatório e único

**Decision**: `tab_fornecedor` (migration V2) com `nome VARCHAR(100) NOT NULL UNIQUE` e
`contato VARCHAR(100)` opcional, mais exclusão lógica via `ativo`.

**Rationale**: O CT-001 exige nome da distribuidora e contato; o edge case da spec exige que
fornecedores com o mesmo nome sejam tratados sem duplicidade na seleção do carregamento. A
unicidade no banco (`UNIQUE` + validação no Service) garante um único registro por
distribuidora (CONVENTIONS §8, constraints em migrations).

**Alternatives considered**: permitir nomes duplicados e desambiguar na seleção (polui a
seleção do carregamento e o relatório de carregamentos RF-042); usar CNPJ como identificador
(requisito não prevê CNPJ no cadastro).

## Decisão 2: Validação de nome e duplicidade no Service

**Decision**: `FornecedorService.validar` exige nome não vazio e bloqueia duplicidade por
nome entre ativos (`findByNomeAndAtivoTrue`), com mensagens pt-BR.

**Rationale**: Regra de negócio no Service conforme CONVENTIONS §6; as mensagens "Informe o
nome do fornecedor." e "Já existe um fornecedor com este nome." são devolvidas como erro
estruturado 422 (CONSTITUICAO §XI.3). A verificação ignora o próprio registro na edição.

**Alternatives considered**: validação apenas com bean validation (cobre campo vazio, mas a
duplicidade exige consulta ao banco, própria do Service); unicidade apenas no banco
(mensagem crua do PostgreSQL, sem pt-BR amigável).

## Decisão 3: Exclusão lógica com histórico de carregamentos preservado

**Decision**: A exclusão usa `DELETE /api/fornecedores/{id}` que marca `ativo = false`
(`BaseService.excluirLogico`); carregamentos históricos permanecem vinculados.

**Rationale**: O fornecedor identifica a origem dos carregamentos (RF-010); o relatório de
carregamentos (RF-042) precisa do nome do fornecedor mesmo depois de o cadastro ser
encerrado. A exclusão lógica preserva o histórico (RNF-007) e o fornecedor inativo deixa de
aparecer na seleção de novos carregamentos.

**Alternatives considered**: `DELETE` físico (quebraria o vínculo dos carregamentos e o
histórico, violando RNF-007); exclusão lógica com bloqueio por carregamentos vinculados
(regra inexistente nos requisitos desta HU; a preservação pelo filtro de ativos é suficiente).

## Decisão 4: Disponibilidade na seleção do carregamento pela listagem de ativos

**Decision**: O registro de carregamento (HU-006) carrega a seleção de fornecedor de
`GET /api/fornecedores` (somente ativos) e grava o carregamento vinculado por
`fornecedor_id` obrigatório.

**Rationale**: O CT-002 exige que o fornecedor cadastrado apareça na seleção do carregamento
e que o carregamento seja vinculado a um fornecedor cadastrado (RF-010). A FK obrigatória
em `tab_carregamento` impede carregamento sem fornecedor (assumption da spec: todo
carregamento possui um fornecedor).

**Alternatives considered**: campo texto livre para a distribuidora no carregamento
(permitiria fornecedor não cadastrado, contrariando o edge case da spec); endpoint dedicado
de "seleção de fornecedores" (redundante com a listagem de ativos).

## Decisão 5: CRUD REST em /api/fornecedores

**Decision**: API REST com `GET /api/fornecedores`, `GET /api/fornecedores/{id}`,
`POST /api/fornecedores`, `PUT /api/fornecedores/{id}` e `DELETE /api/fornecedores/{id}`
(exclusão lógica, 204).

**Rationale**: Padrão REST do CONVENTIONS §6 (GET listar/obter, POST criar, PUT atualizar,
DELETE marcar inativo), rotas `/api/<modulo>` minúsculas (CONVENTIONS §4).

**Alternatives considered**: RPC (`/api/fornecedores/criar`) (diverge do CONVENTIONS §6);
endpoints somente de leitura + cadastro interno (o requisito RF-005 exige cadastro pelo
administrador).

## Decisão 6: Contato como dado mínimo de identificação

**Decision**: O contato é opcional no schema (coluna nullable), mas capturado no formulário
como dado de identificação complementar.

**Rationale**: O CT-001 menciona "nome da distribuidora e contato"; a spec trata o contato
como dado mínimo de identificação além do nome, sem depender dele para a seleção do
carregamento (edge case: "o funcionamento da seleção no carregamento não depende dele").

**Alternatives considered**: contato obrigatório (o CT não impõe; o edge case da spec
explicita que a seleção não depende do contato).
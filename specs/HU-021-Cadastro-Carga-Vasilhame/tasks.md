---

description: "Task list for feature implementation"

---

# Tasks: Cadastro de Carga e Vasilhame

**Input**: Design documents from `/specs/HU-021-Cadastro-Carga-Vasilhame/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/, quickstart.md

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **API**: `gerenciador_estoque_api/src/main/java/com/gerenciador/estoque/`
- **Testes API**: `gerenciador_estoque_api/src/test/java/com/gerenciador/estoque/business/`
- **App**: `gerenciador_estoque_app/src/`

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Projeto e infraestrutura existentes; nenhuma inicialização necessária.

- [x] T001 Confirmar endpoints existentes `POST /api/cargas` e `POST /api/vasilhames` (CargaController, VasilhameController)
- [x] T002 Confirmar schema existente `tab_carga`/`tab_vasilhame` (nome UNIQUE desde V1) e sem necessidade de migration

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Validação de nome obrigatório e duplicado na API, base para todas as user stories (CT-004, CT-005).

- [x] T003 [P] Adicionar `findByNome(String)` em `gerenciador_estoque_api/.../business/carga/CargaRepository.java`
- [x] T004 [P] Adicionar `findByNome(String)` em `gerenciador_estoque_api/.../business/vasilhame/VasilhameRepository.java`
- [x] T005 Sobrescrever `validar()` em `gerenciador_estoque_api/.../business/carga/CargaService.java` (nome vazio e duplicado -> BusinessException 422)
- [x] T006 Sobrescrever `validar()` em `gerenciador_estoque_api/.../business/vasilhame/VasilhameService.java` (nome vazio e duplicado -> BusinessException 422)
- [x] T007 [P] Testes de CargaService: `CargaServiceTest.java` (+3 casos: nome vazio, duplicado, update mantem nome)
- [x] T008 [P] Testes de VasilhameService: `VasilhameServiceTest.java` (+3 casos: nome vazio, duplicado, update mantem nome)

**Checkpoint**: API valida nome obrigatório e duplicidade com mensagem clara em pt-BR (18/18 testes ok nos dois services).

---

## Phase 3: User Story 1 - Cadastro de carga nova no formulario de produto (Priority: P1) MVP

**Goal**: O administrador cria uma carga nova pela tela de produtos, sem sair dela (CT-001, CT-003, CT-004, CT-005).

**Independent Test**: Abrir a tela de produtos, "Nova carga...", informar "Refrigerante", "Criar carga": a carga aparece selecionada no seletor e em `GET /api/cargas`; "Gás" duplicada gera mensagem `Já existe uma carga cadastrada com o nome 'Gás'.`.

- [x] T009 [P] [US1] Adicionar `criarCarga(nome)` em `gerenciador_estoque_app/src/api/produtos.ts` (POST /api/cargas)
- [x] T010 [US1] Adicionar opcao "Nova carga..." no seletor de Carga e formulario inline em `gerenciador_estoque_app/src/features/produtos/ProdutosPage.tsx`
- [x] T011 [US1] Adicionar classe CSS `.nova-opcao` em `gerenciador_estoque_app/src/App.css`

**Checkpoint**: US1 funcional e testavel de forma independente.

---

## Phase 4: User Story 2 - Cadastro de vasilhame novo no formulario de produto (Priority: P1)

**Goal**: O administrador cria um vasilhame novo pela tela de produtos, com nome e preco do casco (CT-002, CT-003, CT-004, CT-005).

**Independent Test**: "Novo vasilhame...", informar "Lata 350ml" (casco 0), "Criar vasilhame": aparece selecionado no seletor e em `GET /api/vasilhames`; "P13" duplicado gera mensagem `Já existe um vasilhame cadastrado com o nome 'P13'.`.

- [x] T012 [P] [US2] Adicionar `criarVasilhame(nome, precoCasco)` em `gerenciador_estoque_app/src/api/produtos.ts` (POST /api/vasilhames)
- [x] T013 [US2] Adicionar opcao "Novo vasilhame..." no seletor de Vasilhame e formulario inline em `gerenciador_estoque_app/src/features/produtos/ProdutosPage.tsx`

**Checkpoint**: US1 e US2 funcionam de forma independente.

---

## Phase 5: User Story 3 - Cadastro do produto combinado com carga e vasilhame novos (Priority: P1)

**Goal**: O produto "Carga + Vasilhame" novo e salvo e exibido pelo nome combinado (CT-006).

**Independent Test**: Com "Refrigerante" + "Lata 350ml" criados, preencher precos e limite e clicar "Cadastrar": o produto aparece na tabela como "Refrigerante Lata 350ml" e em `GET /api/produtos`; repetir a mesma combinacao e bloqueado.

- [x] T014 [US3] Validar fluxo completo no formulario existente de produto (`ProdutosPage.tsx`) com carga e vasilhame novos (combinacao unica garantida pela constraint `uk_produto_carga_vasilhame`)

**Checkpoint**: Todas as user stories funcionais de forma independente.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Build, deploy e validacao end-to-end.

- [x] T015 Rodar suite de testes da API (`mvn -o test` - suíte completa 100% verde)
- [x] T016 Build do app (`npm run build`) e copia de `dist/` para `gerenciador_estoque_api/src/main/resources/static/`
- [x] T017 Build do JAR (`mvn -o package -DskipTests`) e substituicao em `gerenciador_estoque_jar/gerenciador-estoque-api-0.0.1-SNAPSHOT.jar`
- [x] T018 Reiniciar o servico e validar ao vivo: bundle novo servido, POST carga/vasilhame/produto ok, duplicidade 422, limpeza dos dados de teste
- [x] T019 Rodar os cenarios do quickstart.md (CT-001 a CT-006)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - concluida (projeto existente)
- **Foundational (Phase 2)**: Depende da fase 1 - BLOQUEIA todas as user stories (validacao de duplicidade antes da UI)
- **User Stories (Phase 3+)**: Dependem da fase 2; US3 depende de US1 e US2 (combina carga e vasilhame novos)
- **Polish (Final Phase)**: Depende das user stories completas

### User Story Dependencies

- **US1 (P1)**: Sem dependencias de outras stories (endpoints ja existiam)
- **US2 (P1)**: Sem dependencias de outras stories (endpoints ja existiam; pode rodar em paralelo com US1)
- **US3 (P1)**: Depende de US1 e US2 (requer carga e vasilhame novos cadastrados)

### Parallel Opportunities

- T003/T004 (findByNome) e T007/T008 (testes) em paralelo
- US1 e US2 em paralelo (arquivos distintos: produtos.ts compartilhado apenas para as funcoes novas, ambas sem dependencia entre si)
- T009/T012 (api client) em paralelo com T010/T011/T013 (UI)

---

## Parallel Example: US1 + US2

```bash
# Tarefas em paralelo possivel:
Task: "criarCarga em src/api/produtos.ts"
Task: "criarVasilhame em src/api/produtos.ts"
Task: "opcao Nova carga em ProdutosPage.tsx"
Task: "opcao Novo vasilhame em ProdutosPage.tsx"
Task: "classe .nova-opcao em App.css"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Fase 1: Setup (existente)
2. Fase 2: Foundational (validacao na API + testes)
3. Fase 3: US1 (carga nova) - MVP
4. STOP e VALIDATE: US1 testada isoladamente

### Incremental Delivery

1. Foundational concluida -> base valida
2. US1 (carga nova) -> testada -> MVP
3. US2 (vasilhame novo) -> testada
4. US3 (produto combinado) -> testada
5. Polish: build, JAR, deploy, quickstart

---

## Notes

- [x] = tarefa concluida e validada nesta entrega (2026-08-20)
- [P] tasks = arquivos diferentes, sem dependencias
- [Story] label mapeia a tarefa para a user story do spec
- O acrescimo do cartao continua somente para carga Gas (RF-021-A); cargas novas nao recebem acrescimo
- Commit por grupo logico: API (validacao + testes), app (UI), jar (rebuild), audi (artefatos HU-021)
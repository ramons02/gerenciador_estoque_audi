# Implementation Plan: Cadastro de Fornecedores (Distribuidoras)

**Branch**: `HU-005-Cadastro-Fornecedores` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-005-Cadastro-Fornecedores/spec.md`

## Summary

Permitir ao administrador cadastrar fornecedores/distribuidoras com nome e contato, e
disponibilizar o fornecedor cadastrado na seleção do registro de carregamento (RF-005,
RF-010, CT-001 e CT-002). Abordagem: tabela `tab_fornecedor` (migration V2) com nome único,
CRUD REST em `/api/fornecedores`, exclusão lógica (`ativo = false`), e vinculação obrigatória
do fornecedor ao carregamento por chave estrangeira (`tab_carregamento.fornecedor_id`,
implementada na HU-006).

## Technical Context

**Language/Version**: Java 21 + Spring Boot 3.x (backend); React 18 + TypeScript 5.x (frontend)
**Primary Dependencies**: Spring Data JPA, Spring Validation, Flyway, Lombok (backend); React, Vite, axios, react-router (frontend)
**Storage**: PostgreSQL (único banco) + migrações Flyway forward-only
**Testing**: JUnit 5 + Mockito (Service/Controller), integração com @SpringBootTest; Vitest + React Testing Library (app)
**Target Platform**: navegador web (SPA) + API REST
**Project Type**: full-stack web (REST API + SPA)
**Performance Goals**: cadastro e listagem de fornecedores em menos de 2 s (RNF-003)
**Constraints**: nome único (sem duplicidade na seleção de carregamento), carregamento exige fornecedor cadastrado (RF-010), exclusão lógica preserva histórico de carregamentos (RNF-007), erro JSON com mensagem pt-BR
**Scale/Scope**: revenda única, 1 a 5 usuários simultâneos, histórico de 12 meses (RNF-010)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate | Verificação | Resultado |
|---|---|---|
| §I-A Artefatos por HU | spec.md, plan.md, research.md, data-model.md, contracts/ e quickstart.md existem antes de qualquer código | OK |
| §II Vocabulário | Documentação usa Carga, Vasilhame, Cheio, Vazio, Em rua, Troca, Carregamento e Pátio | OK |
| §III Invariantes de estoque | Feature não altera estoque; somente identifica a origem do carregamento | OK |
| §V Critério fechado = CT provado | Cada CT (CT-001 e CT-002) só fecha com prova registrada no tasks.md | OK |
| §VI Proibido | Sem DELETE físico; sem regra fora dos requisitos; sem termo não canônico | OK |
| §VII Documentação | pt-BR, hífen normal, sem travessão | OK |
| §X Rastreabilidade | RF-005, RF-010 e HU-005 referenciados em commits e doc do módulo | OK |
| §XI Qualidade | Validação de nome e duplicidade no Service (lógica pura testada); erro estruturado | OK |

**GATE: PASS**

## Project Structure

### Documentation (this feature)

```text
specs/HU-005-Cadastro-Fornecedores/
├── plan.md              # Este arquivo
├── research.md          # Fase 0: decisões técnicas
├── data-model.md        # Fase 1: modelo de dados
├── quickstart.md        # Fase 1: validação end-to-end
├── contracts/
│   ├── api.md           # Contrato da API REST
│   └── ui.md            # Contrato de UI
└── tasks.md             # Fase 2: checklist de CTs
```

### Source Code (repository root)

```text
gerenciador_estoque_api/
├── pom.xml
└── src/
    ├── main/java/com/gerenciador/estoque/
    │   ├── core/
    │   │   ├── exception/         # BusinessException, NotFoundException, GlobalExceptionHandler
    │   │   ├── model/             # BaseModel
    │   │   ├── repository/        # BaseRepository
    │   │   └── service/           # BaseService
    │   └── fornecedor/
    │       ├── FornecedorController.java   # GET/POST/PUT/DELETE /api/fornecedores
    │       ├── FornecedorService.java      # validar nome + duplicidade
    │       ├── FornecedorRepository.java   # findByNomeAndAtivoTrue
    │       └── Fornecedor.java
    ├── main/resources/db/migration/     # V2__criar_tab_fornecedor.sql (tab_fornecedor)
    └── test/java/com/gerenciador/estoque/
        └── business/fornecedor/FornecedorServiceTest.java   # JUnit 5 + Mockito

gerenciador_estoque_app/
└── src/
    ├── api/
    │   ├── client.ts            # Cliente HTTP tipado (axios)
    │   ├── types.ts             # Fornecedor, FornecedorInput
    │   └── fornecedores.ts      # listarFornecedores, criarFornecedor, atualizarFornecedor
    └── features/
        ├── fornecedores/        # Listagem e formulário de fornecedores (HU-005)
        └── carregamentos/       # Seleção do fornecedor no carregamento (HU-006, consume daqui)
```

**Structure Decision**: monorepo com repositórios irmãos conforme Constituição §I. A feature
vive no módulo `com.gerenciador.estoque.fornecedor` (camadas Controller/Service/Repository).
No app, a feature vive em `src/features/fornecedores/`; a listagem de ativos é reutilizada
pelo registro de carregamento (`src/features/carregamentos/`, HU-006) como seleção de
fornecedor (RF-010, CT-002).

## Complexity Tracking

> Nenhuma violação. Tabela única com unicidade de nome e consulta simples, sem necessidade
> de estrutura adicional para a seleção do carregamento.
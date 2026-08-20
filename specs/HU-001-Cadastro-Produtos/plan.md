# Implementation Plan: Cadastro de Produtos (Carga + Vasilhame)

**Branch**: `HU-001-Cadastro-Produtos` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-001-Cadastro-Produtos/spec.md`

## Summary

Permitir ao administrador cadastrar produtos compostos por Carga/Conteúdo e Vasilhame/Casco
(ex.: Gás P13 = carga Gás + vasilhame P13), listados pelo nome combinado, com bloqueio de
duplicidade e exclusão condicionada à inexistência de movimentações (RF-001, RDN-001,
CT-001 a CT-004). Abordagem: entidades de referência `tab_carga` e `tab_vasilhame`,
`tab_produto` com composição por chave estrangeira e unicidade `(carga_id, vasilhame_id)`,
nome combinado derivado, CRUD REST em `/api/produtos`, `/api/cargas` e `/api/vasilhames`,
exclusão lógica (`ativo = false`) bloqueada quando existirem vendas ou carregamentos
vinculados (RNF-007).

## Technical Context

**Language/Version**: Java 21 + Spring Boot 3.x (backend); React 18 + TypeScript 5.x (frontend)
**Primary Dependencies**: Spring Data JPA, Spring Validation, Flyway, Lombok (backend); React, Vite, axios, react-router (frontend)
**Storage**: PostgreSQL (único banco) + migrações Flyway forward-only
**Testing**: JUnit 5 + Mockito (Service/Controller), integração com @SpringBootTest; Vitest + React Testing Library (app)
**Target Platform**: navegador web (SPA) + API REST
**Project Type**: full-stack web (REST API + SPA)
**Performance Goals**: cadastro e listagem de produtos em menos de 2 s, alinhado ao RNF-003
**Constraints**: transação atômica estoque+caixa (RNF-005), lock por produto em operações de estoque (RNF-008), sem DELETE físico de movimento (RNF-007), validação de negócio no Service, erro JSON com mensagem pt-BR
**Scale/Scope**: revenda única, 1 a 5 usuários simultâneos, histórico de 12 meses (RNF-010)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate | Verificação | Resultado |
|---|---|---|
| §I-A Artefatos por HU | spec.md, plan.md, research.md, data-model.md, contracts/ e quickstart.md existem antes de qualquer código | OK |
| §II Vocabulário | Documentação usa Carga, Vasilhame, Cheio, Vazio, Em rua, Troca, Carregamento e Pátio | OK |
| §III Invariantes de estoque | Feature não altera saldo de estoque; exclusão lógica preserva movimentações | OK |
| §V Critério fechado = CT provado | Cada CT (CT-001 a CT-004) só fecha com prova registrada no task.md | OK |
| §VI Proibido | Sem DELETE físico de movimento; sem regra fora dos requisitos; sem termo não canônico | OK |
| §VII Documentação | pt-BR, hífen normal, sem travessão | OK |
| §X Rastreabilidade | RF-001 e HU-001 referenciados em commits e doc do módulo | OK |
| §XI Qualidade | Validação de duplicidade no Service (lógica pura testada); erro estruturado | OK |

**GATE: PASS**

## Project Structure

### Documentation (this feature)

```text
specs/HU-001-Cadastro-Produtos/
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
    │   │   ├── config/            # CorsConfig, TimezoneConfig
    │   │   ├── exception/         # BusinessException, NotFoundException, GlobalExceptionHandler
    │   │   ├── model/             # BaseModel (criado_em, atualizado_em, ativo)
    │   │   ├── repository/        # BaseRepository
    │   │   └── service/           # BaseService (listar, buscar, salvar, atualizar, excluirLogico)
    │   ├── carga/
    │   │   ├── CargaController.java
    │   │   ├── CargaService.java
    │   │   ├── CargaRepository.java
    │   │   └── Carga.java
    │   ├── vasilhame/
    │   │   ├── VasilhameController.java
    │   │   ├── VasilhameService.java
    │   │   ├── VasilhameRepository.java
    │   │   └── Vasilhame.java
    │   └── produto/
    │       ├── ProdutoController.java
    │       ├── ProdutoService.java
    │       ├── ProdutoRepository.java
    │       └── Produto.java
    ├── main/resources/db/migration/   # V1__criar_tabelas_base.sql (tab_carga, tab_vasilhame, tab_produto)
    └── test/java/com/gerenciador/estoque/
        └── business/produto/ProdutoServiceTest.java   # JUnit 5 + Mockito

gerenciador_estoque_app/
└── src/
    ├── api/
    │   ├── client.ts            # Cliente HTTP tipado (axios)
    │   ├── types.ts             # Tipos Carga, Vasilhame, Produto, ProdutoInput
    │   └── produtos.ts          # listarCargas, listarVasilhames, listarProdutos, criarProduto, atualizarProduto
    └── features/produtos/
        ├── ProdutosPage.tsx     # Lista e formulário de produto (carga + vasilhame)
        └── componentes/         # Subcomponentes do formulário e da listagem
```

**Structure Decision**: monorepo com repositórios irmãos (`gerenciador_estoque_api`,
`gerenciador_estoque_app`, `gerenciador_estoque_infra`, `gerenciador_estoque_audi`,
`gerenciador_estoque_jar`) conforme Constituição §I. Na API, a feature vive no módulo
`com.gerenciador.estoque.produto` com as camadas Controller (REST), Service (`@Transactional`,
regra de negócio e validação) e Repository (Spring Data JPA), além dos módulos de apoio
`carga` e `vasilhame` que alimentam a composição do produto. No app, a feature vive em
`src/features/produtos/`, consumindo a API por cliente HTTP tipado em `src/api/`. As
entidades de referência (carga e vasilhame) são módulos próprios por serem reutilizadas por
outras features (venda, carregamento, estoque).

## Complexity Tracking

> Nenhuma violação. A estrutura de 5 repositórios irmãos e a separação dos módulos carga e
> vasilhame seguem a Constituição §I e o CONVENTIONS §1, sem necessidade de exceção.
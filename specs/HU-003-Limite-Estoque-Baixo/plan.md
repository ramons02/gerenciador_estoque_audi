# Implementation Plan: Limite Mínimo de Estoque (Alerta)

**Branch**: `HU-003-Limite-Estoque-Baixo` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-003-Limite-Estoque-Baixo/spec.md`

## Summary

Permitir ao administrador definir um limite mínimo de estoque de cheios por produto e gerar
alerta visual persistente quando o saldo de cheios atingir ou ficar abaixo do limite,
dispensado somente após nova entrada de caminhão elevar o estoque acima do limite (RF-003,
RF-032, RGN-004, CT-001 a CT-003). Abordagem: coluna `limite_minimo` na `tab_produto`
(migration V1), alerta derivado da condição `estoque_cheios <= limite_minimo` na consulta
(nenhum estado persistido), endpoint `PUT /api/produtos/{id}/limite-minimo`, e dispensa
avaliada pelo recálculo automático no registro de carregamento (HU-006).

## Technical Context

**Language/Version**: Java 21 + Spring Boot 3.x (backend); React 18 + TypeScript 5.x (frontend)
**Primary Dependencies**: Spring Data JPA, Spring Validation, Flyway, Lombok (backend); React, Vite, axios, react-router (frontend)
**Storage**: PostgreSQL (único banco) + migrações Flyway forward-only
**Testing**: JUnit 5 + Mockito (Service/Controller), integração com @SpringBootTest; Vitest + React Testing Library (app)
**Target Platform**: navegador web (SPA) + API REST
**Project Type**: full-stack web (REST API + SPA)
**Performance Goals**: consulta de estoque e alerta em menos de 2 s (RNF-003), escala pequena
**Constraints**: condição de alerta "atingir ou ficar abaixo" (`<=`), dispensa somente por carregamento (RGN-004), sem estado persistente de alerta, transações de estoque atômicas (RNF-005) e com lock por produto (RNF-008), erro JSON com mensagem pt-BR
**Scale/Scope**: revenda única, 1 a 5 usuários simultâneos, histórico de 12 meses (RNF-010)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate | Verificação | Resultado |
|---|---|---|
| §I-A Artefatos por HU | spec.md, plan.md, research.md, data-model.md, contracts/ e quickstart.md existem antes de qualquer código | OK |
| §II Vocabulário | Documentação usa Carga, Vasilhame, Cheio, Vazio, Em rua, Troca, Carregamento e Pátio | OK |
| §III Invariantes de estoque | Feature apenas lê o saldo de cheios para derivar o alerta; não altera estoque | OK |
| §V Critério fechado = CT provado | Cada CT (CT-001 a CT-003) só fecha com prova registrada no tasks.md | OK |
| §VI Proibido | Sem DELETE físico; regra RGN-004 documentada em requisitos; sem termo não canônico | OK |
| §VII Documentação | pt-BR, hífen normal, sem travessão | OK |
| §X Rastreabilidade | RF-003, RF-032, RGN-004 e HU-003 referenciados em commits e doc do módulo | OK |
| §XI Qualidade | Condição de alerta como lógica pura testada (sem mock); erro estruturado | OK |

**GATE: PASS**

## Project Structure

### Documentation (this feature)

```text
specs/HU-003-Limite-Estoque-Baixo/
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
    │   └── produto/
    │       ├── ProdutoController.java   # PUT /api/produtos/{id}/limite-minimo
    │       ├── ProdutoService.java      # alterarLimiteMinimo(); estoqueBaixo() derivado
    │       ├── ProdutoRepository.java
    │       └── Produto.java             # limiteMinimo, estoqueCheios; estoqueBaixo() = cheios <= limite
    ├── main/resources/db/migration/     # V1__criar_tabelas_base.sql (coluna limite_minimo)
    └── test/java/com/gerenciador/estoque/
        └── business/produto/ProdutoServiceTest.java   # condicao de alerta (JUnit 5, sem mock)

gerenciador_estoque_app/
└── src/
    ├── api/
    │   ├── client.ts            # Cliente HTTP tipado (axios)
    │   ├── types.ts             # EstoqueProduto (estoqueBaixo, sugestaoReposicao)
    │   └── produtos.ts          # atualizarLimiteMinimo
    └── features/
        ├── produtos/ProdutosPage.tsx    # Campo "limite mínimo" no formulário
        ├── estoque/                     # Painel de estoque (HU-012): exibe saldos e alertas
        └── dashboard/                   # Dashboard (HU-019): alerta persistente (RF-032/RF-053)
```

**Structure Decision**: monorepo com repositórios irmãos conforme Constituição §I. O limite
mínimo é atributo de `tab_produto` e vive no módulo `com.gerenciador.estoque.produto`; o
alerta derivado é exposto pelos módulos de leitura (`estoque` e `dashboard`), sem estado
persistente. No app, o campo de configuração fica em `src/features/produtos/` e a exibição do
alerta em `src/features/estoque/` e `src/features/dashboard/`.

## Complexity Tracking

> Nenhuma violação. O alerta é derivado da comparação de saldo com o limite (lógica pura),
> sem tabela ou fila de alertas: alternativa mais simples e consistente com a escala.
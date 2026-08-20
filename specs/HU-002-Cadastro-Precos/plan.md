# Implementation Plan: Cadastro de Preços (Custo e Venda)

**Branch**: `HU-002-Cadastro-Precos` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-002-Cadastro-Precos/spec.md`

## Summary

Permitir ao administrador cadastrar o preço de custo e o preço de venda por produto, em
reais, com validação de que o preço de venda nunca seja inferior ao custo (RGN-005) e com
preservação do preço praticado no momento de cada venda, sem recalcular vendas antigas
(RF-002, CT-001 a CT-003). Abordagem: colunas `preco_custo` e `preco_venda` (NUMERIC 12,2)
na `tab_produto`, validação no `ProdutoService` (regra de negócio), snapshot do valor
unitário na `tab_venda` no lançamento da venda (HU-007) e atualização de preço pelo
`PUT /api/produtos/{id}`.

## Technical Context

**Language/Version**: Java 21 + Spring Boot 3.x (backend); React 18 + TypeScript 5.x (frontend)
**Primary Dependencies**: Spring Data JPA, Spring Validation, Flyway, Lombok (backend); React, Vite, axios, react-router (frontend)
**Storage**: PostgreSQL (único banco) + migrações Flyway forward-only
**Testing**: JUnit 5 + Mockito (Service/Controller), integração com @SpringBootTest; Vitest + React Testing Library (app)
**Target Platform**: navegador web (SPA) + API REST
**Project Type**: full-stack web (REST API + SPA)
**Performance Goals**: cadastro e alteração de preço em menos de 2 s, alinhado ao RNF-003
**Constraints**: validação RGN-005 no Service (transacional), valores monetários sem ponto flutuante, preço da venda congelado no lançamento (CT-003), sem DELETE físico de movimento (RNF-007), erro JSON com mensagem pt-BR
**Scale/Scope**: revenda única, 1 a 5 usuários simultâneos, histórico de 12 meses (RNF-010)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate | Verificação | Resultado |
|---|---|---|
| §I-A Artefatos por HU | spec.md, plan.md, research.md, data-model.md, contracts/ e quickstart.md existem antes de qualquer código | OK |
| §II Vocabulário | Documentação usa Carga, Vasilhame, Cheio, Vazio, Em rua, Troca, Carregamento e Pátio | OK |
| §III Invariantes de estoque | Feature não altera saldo de estoque; alteração de preço não toca movimentações passadas | OK |
| §V Critério fechado = CT provado | Cada CT (CT-001 a CT-003) só fecha com prova registrada no task.md | OK |
| §VI Proibido | Sem DELETE físico; regra RGN-005 documentada em requisitos; sem termo não canônico | OK |
| §VII Documentação | pt-BR, hífen normal, sem travessão | OK |
| §X Rastreabilidade | RF-002, RGN-005 e HU-002 referenciados em commits e doc do módulo | OK |
| §XI Qualidade | Validação de venda inferior ao custo no Service (lógica pura testada); erro estruturado | OK |

**GATE: PASS**

## Project Structure

### Documentation (this feature)

```text
specs/HU-002-Cadastro-Precos/
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
    │       ├── ProdutoController.java   # PUT /api/produtos/{id} atualiza preços
    │       ├── ProdutoService.java      # validar() aplica RGN-005
    │       ├── ProdutoRepository.java
    │       └── Produto.java             # precoCusto, precoVenda (BigDecimal)
    ├── main/resources/db/migration/     # V1__criar_tabelas_base.sql (colunas de preço)
    └── test/java/com/gerenciador/estoque/
        └── business/produto/ProdutoServiceTest.java   # validação RGN-005 (JUnit 5 + Mockito)

gerenciador_estoque_app/
└── src/
    ├── api/
    │   ├── client.ts            # Cliente HTTP tipado (axios)
    │   ├── types.ts             # Produto (precoCusto, precoVenda como string)
    │   └── produtos.ts          # criarProduto, atualizarProduto
    └── features/produtos/
        └── ProdutosPage.tsx     # Campos de preço no formulário e colunas na listagem
```

**Structure Decision**: monorepo com repositórios irmãos conforme Constituição §I. A feature
não cria módulo novo: os preços são atributos de `tab_produto` e vivem no módulo
`com.gerenciador.estoque.produto` (camadas Controller/Service/Repository). No app, os campos
de preço integram `src/features/produtos/`. O congelamento do preço na venda é
responsabilidade do módulo `venda` (HU-007), que grava o valor unitário no momento do
lançamento.

## Complexity Tracking

> Nenhuma violação. Não há necessidade de tabela de histórico de preços nesta fase: o
> CT-003 é satisfeito pelo snapshot do valor unitário na venda, e o cadastro mantém apenas o
> preço vigente (alternativa mais simples).
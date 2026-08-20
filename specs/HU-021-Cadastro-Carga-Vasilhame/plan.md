# Implementation Plan: Cadastro de Carga e Vasilhame

**Branch**: `HU-021-Cadastro-Carga-Vasilhame` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-021-Cadastro-Carga-Vasilhame/spec.md`

## Summary

A feature permite ao administrador cadastrar uma carga nova (ex.: "Refrigerante") e um vasilhame novo (ex.: "Lata 350ml") diretamente na tela de cadastro de produtos, sem sair da tela, e em seguida concluir o cadastro do produto combinado (carga + vasilhame + preços). Abordagem técnica: os recursos de criação já existem na API (`POST /api/cargas` e `POST /api/vasilhames`, módulos `business.carga` e `business.vasilhame`); a feature adiciona validação de nome duplicado e nome vazio no Service (via hook `validar` do `BaseService`, com `BusinessException` em pt-BR) e, no app, as opções "Nova carga..." e "Novo vasilhame..." nos seletores do formulário de produto da `ProdutosPage`, com formulário inline que cria o registro e já o deixa selecionado para continuar o cadastro (CT-001 a CT-006). Nenhuma migration é necessária: `tab_carga` e `tab_vasilhame` já existem desde a V1 com nome UNIQUE.

## Technical Context

**Language/Version**: Backend Java 21 + Spring Boot 3.x (Maven); Frontend React 18 + TypeScript 5.x (Vite)
**Primary Dependencies**: Spring Web (REST), Spring Data JPA, Lombok (API); React (app)
**Storage**: PostgreSQL único; migrations Flyway forward-only (CONVENTIONS §8). Sem migration nova: `tab_carga` e `tab_vasilhame` existem desde a V1 (`nome VARCHAR(50) NOT NULL UNIQUE`, `ativo BOOLEAN`)
**Testing**: JUnit 5 + Mockito (Service); testes existentes de `CargaServiceTest` e `VasilhameServiceTest` estendidos com casos de nome vazio, nome duplicado e atualização mantendo o próprio nome
**Target Platform**: Navegador web (desktop e tablet)
**Project Type**: Full-stack web: REST API (Spring Boot) + SPA (React/Vite)
**Performance Goals**: Criação de carga/vasilhame respondendo em até 2 segundos; sem necessidade de recarregar a página para o novo item aparecer no seletor
**Constraints**: Nome de carga/vasilhame obrigatório e único (nome exato, sem normalização de maiúsculas); erro de duplicidade com mensagem clara em pt-BR (CONVENTIONS §6); sem telas de manutenção dedicadas nesta versão (escopo do spec); acréscimo do cartão continua somente para carga Gás (RF-021-A), cargas novas não recebem acréscimo
**Scale/Scope**: Revenda única, 1 a 5 usuários simultâneos (RNF-010); criação de carga/vasilhame é operação rara, sem exigência de volume

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate | Verificação | Resultado |
|---|---|---|
| §I-A | spec.md e plan.md existem e são consistentes com a HU-021 antes de qualquer código | OK |
| §II | Vocabulário canônico: carga, vasilhame, produto, casco; sem sinônimos proibidos | OK |
| §III | Invariantes: produto é combinação única de carga + vasilhame; carga/vasilhame inativáveis, nunca apagados | OK |
| §V | Plano precede o código; CT fechado exige prova (teste ou evidência registrada) | OK |
| §VI | Sem regra fora de requisitos; sem referência a IA; sem comportamento silencioso (erro duplicado é explícito) | OK |
| §VII | Documentação em pt-BR; hífen normal, proibido travessão (em-dash) | OK |
| §X | Rastreabilidade CT-001 a CT-006 e HU-021 nos commits e na doc da feature | OK |
| §XI | CTs provados; validação de duplicidade não colapsa em erro de sistema | OK |

**GATE: PASS** - a feature não viola nenhum gate da Constituição.

## Project Structure

### Documentation (this feature)

```text
specs/HU-021-Cadastro-Carga-Vasilhame/
├── plan.md              # Este arquivo
├── research.md          # Fase 0: decisoes tecnicas
├── data-model.md        # Fase 1: modelo de dados
├── quickstart.md        # Fase 1: validacao end-to-end
├── contracts/           # Fase 1: contratos
│   ├── api.md           # Contrato REST da feature
│   └── ui.md            # Contrato de UI da feature
└── tasks.md             # Fase 2: checklist de CTs
```

### Source Code (repository root)

```text
Gerenciador de Estoque/                       # monorepo em repositórios irmãos
├── gerenciador_estoque_api/                  # Backend: Java 21 + Spring Boot 3.x (Maven)
│   ├── src/main/java/com/gerenciador/estoque/
│   │   ├── core/                             # Base compartilhada (service/BaseService com hook validar)
│   │   └── business/
│   │       ├── carga/                        # Módulo HU-021 (carga)
│   │       │   ├── CargaService.java         # validar(): nome obrigatório e único (BusinessException)
│   │       │   └── CargaRepository.java      # findByNome(String) novo
│   │       └── vasilhame/                    # Módulo HU-021 (vasilhame)
│   │           ├── VasilhameService.java     # validar(): nome obrigatório e único (BusinessException)
│   │           └── VasilhameRepository.java  # findByNome(String) novo
│   └── src/test/java/com/gerenciador/estoque/business/
│       ├── carga/CargaServiceTest.java       # +3 casos (vazio, duplicado, update mantém nome)
│       └── vasilhame/VasilhameServiceTest.java  # +3 casos (vazio, duplicado, update mantém nome)
├── gerenciador_estoque_app/                  # Frontend: React 18 + TypeScript + Vite
│   └── src/
│       ├── api/produtos.ts                   # +criarCarga(nome), +criarVasilhame(nome, precoCasco)
│       └── features/produtos/
│           └── ProdutosPage.tsx              # opções "Nova carga..." e "Novo vasilhame..." + formulário inline
├── gerenciador_estoque_infra/                # Infraestrutura, PostgreSQL, deploy
├── gerenciador_estoque_audi/                 # Documentação e auditoria (este plano)
└── gerenciador_estoque_jar/                  # JAR executável (privado)
```

**Structure Decision**: Monorepo em 5 repositórios irmãos (api, app, infra, audi, jar) conforme §I da Constituição. Na API, a validação vive nos Services existentes de `business.carga` e `business.vasilhame` sobrescrevendo o hook `validar()` do `BaseService` (padrão já usado por `ClienteService` e `VendaService`), sem novo módulo. No app, a feature vive na `ProdutosPage` existente com cliente HTTP tipado em `src/api/produtos.ts`, nunca fetch solto (CONVENTIONS §7). Os endpoints `POST /api/cargas` e `POST /api/vasilhames` já existiam; nenhum endpoint novo foi criado.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| Nenhuma violação | - | - |
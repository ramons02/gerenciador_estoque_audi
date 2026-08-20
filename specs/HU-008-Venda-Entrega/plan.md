# Implementation Plan: Venda com Entrega (Taxa de Entrega)

**Branch**: `HU-008-Venda-Entrega` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-008-Venda-Entrega/spec.md`

## Summary

A feature soma automaticamente a taxa de entrega configurada ao total quando o tipo da venda é Entrega, permite ao administrador configurar o valor da taxa e discrimina tipo e total com taxa no relatório de vendas. É uma variação do lançamento da HU-007 sobre a mesma entidade Venda: o `VendaService` lê `taxa_entrega` de `tab_configuracao` ao persistir vendas com `tipo = ENTREGA` (o frontend exibe o valor no formulário), e a tela de Configurações expõe o campo da taxa via `PUT /api/configuracoes`. O plano cobre apenas os CTs da HU-008, referenciando a estrutura comum de venda da HU-007.

## Technical Context

**Language/Version**: Backend Java 21 + Spring Boot 3.x (Maven); Frontend React 18 + TypeScript 5.x (Vite)
**Primary Dependencies**: Spring Web (REST), Spring Data JPA, Flyway, Lombok (API); React, react-router, axios (app)
**Storage**: PostgreSQL único; migrations Flyway `V<N>__descricao_snake_case.sql` forward-only (CONVENTIONS §8); taxa em `tab_configuracao`, nunca hardcoded (CONVENTIONS §8)
**Testing**: JUnit 5 + Mockito (Service/Controller); `@SpringBootTest` para integração; Vitest + React Testing Library (app)
**Target Platform**: Navegador web (desktop e tablet) consumindo a REST API (RNF-011)
**Project Type**: Full-stack web: REST API (Spring Boot) + SPA (React/Vite)
**Performance Goals**: Lançamento de venda (incluindo Entrega) < 2s (RNF-003); mínimo de passos na portaria (RNF-001)
**Constraints**: Taxa aplicada automaticamente no Service quando tipo ENTREGA (RF-022, RGN-001); taxa configurável e não retroativa (só novas vendas); transação atômica estoque + caixa (RNF-005); sem DELETE de venda (RNF-007); erros estruturados em pt-BR (CONVENTIONS §6); retenção de 12 meses (RNF-010)
**Scale/Scope**: Revenda única, 1 a 5 usuários simultâneos (RNF-010); vendas de entrega são fração do volume diário de vendas

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate | Verificação | Resultado |
|---|---|---|
| §I-A | spec.md, plan.md e task.md existem e são consistentes com a HU-008 antes de qualquer código | OK |
| §II | Vocabulário canônico: venda, entrega, balcão, taxa de entrega, carga, vasilhame; sem sinônimos proibidos | OK |
| §III | Invariantes: estoque nunca negativo; movimentação altera estoque e caixa na mesma transação; venda nunca apagada | OK |
| §V | CT fechado exige prova (teste ou evidência registrada); plano precede o código | OK |
| §VI | Sem DELETE de venda; sem regra fora de requisitos (taxa vem de RF-022/RGN-001); sem referência a IA | OK |
| §VII | Documentação em pt-BR; hífen normal, proibido travessão (em-dash) | OK |
| §X | Rastreabilidade RF-020/RF-022/RF-052 e HU-008 nos commits e na doc do módulo | OK |
| §XI | CTs provados; regras de caixa com prioridade máxima de cobertura; validação não colapsa em erro de sistema | OK |

**GATE: PASS** - a feature não viola nenhum gate da Constituição.

## Project Structure

### Documentation (this feature)

```text
specs/HU-008-Venda-Entrega/
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
│   │   ├── core/                             # Base compartilhada (model, repository, service, exception, config)
│   │   └── business/
│   │       ├── venda/                        # Módulo venda (base HU-007; HU-008 estende)
│   │       │   ├── VendaService.java         # Aplica taxa_entrega quando tipo = ENTREGA (RF-022)
│   │       │   ├── Venda.java                # Campo taxa_entrega (persistido no cabeçalho)
│   │       │   └── dto/VendaRequest.java, VendaResponse.java
│   │       ├── configuracao/                 # ConfiguracaoController/Service: GET e PUT /api/configuracoes (taxa_entrega)
│   │       └── relatorio/                    # RelatorioVendas (discrimina tipo e total com taxa - CT-003)
│   ├── src/main/resources/db/migration/      # V9__configuracoes_pagamento.sql já insere taxa_entrega (10.00)
│   └── src/test/java/com/gerenciador/estoque/business/
│       ├── venda/                            # VendaServiceTest (caso ENTREGA com e sem taxa)
│       └── configuracao/                     # ConfiguracaoServiceTest (PUT da taxa)
├── gerenciador_estoque_app/                  # Frontend: React 18 + TypeScript + Vite
│   └── src/
│       ├── api/
│       │   ├── configuracoes.ts              # GET/PUT configurações (taxa de entrega)
│       │   └── vendas.ts                     # POST /api/vendas com tipo ENTREGA
│       └── features/
│           ├── vendas/
│           │   └── components/LancamentoVenda.tsx   # Exibe e aplica a taxa quando tipo Entrega (CT-001)
│           └── configuracoes/
│               └── ConfiguracoesPage.tsx     # Campo "Taxa de entrega (R$)" (CT-002)
├── gerenciador_estoque_infra/                # Infraestrutura, PostgreSQL, deploy
├── gerenciador_estoque_audi/                 # Documentação e auditoria (este plano)
└── gerenciador_estoque_jar/                  # JAR executável (privado)
```

**Structure Decision**: Monorepo em 5 repositórios irmãos (api, app, infra, audi, jar) conforme §I da Constituição. HU-008 não cria módulo novo: estende `com.gerenciador.estoque.business.venda` (lógica da taxa no `VendaService`, campo `taxa_entrega` em `tab_venda`) e `com.gerenciador.estoque.business.configuracao` (GET/PUT da taxa), seguindo as camadas Controller/Service/Repository/dto. No app, o campo da taxa entra no `LancamentoVenda.tsx` existente e na `ConfiguracoesPage.tsx`. Relatório de vendas (`business.relatorio`) passa a exibir tipo e total com taxa (CT-003), usando os cabeçalhos pt-BR do CONVENTIONS §10 (RF-041).

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| Nenhuma violação | - | - |
# Implementation Plan: Dashboard (Resumo do Dia)

**HU**: HU-019 - Dashboard (Resumo do Dia)

**Branch**: `HU-019-Dashboard-Resumo-Dia` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-019-Dashboard-Resumo-Dia/spec.md`

## Summary

Requisito primário: exibir na tela inicial o resumo do dia - Total Faturado (RF-050, CT-001), totais por forma de pagamento Dinheiro/PIX/Cartão com crédito e débito somados em um único valor (RF-051, CT-002), total de vasilhames vendidos por produto e total geral (RF-052, CT-003) e alertas de estoque baixo ativos (RF-053, CT-004), com vendas canceladas fora de todos os totais (FR-005).

Abordagem técnica: módulo `dashboard` na API com `GET /api/dashboard/resumo-dia`, consulta de leitura que agrega as vendas `status = ATIVA` do dia por forma de pagamento (RF-050/RF-051/RGN-008) e por produto (RF-052) e deriva os alertas comparando `tab_estoque.qtd_cheios` com `limite_minimo` (RF-053, RF-032, RGN-004). A resposta JSON única alimenta a tela inicial em `src/features/dashboard/`, com cards de valores grandes (RNF-002) e alerta persistente enquanto ativo. Leitura pura, sem escrita (RNF-003: resposta em menos de 2 segundos).

## Technical Context

**Language/Version**: Java 21 + Spring Boot 3.x (API); React 18 + TypeScript 5.x (app)
**Primary Dependencies**: Spring Data JPA, Flyway, Lombok (API); React, Vite, axios, react-router (app)
**Storage**: PostgreSQL único + Flyway forward-only; tabelas `tab_*`, sequências `seq_*`, colunas snake_case pt-BR
**Testing**: JUnit 5 + Mockito (Service/Controller); integração com @SpringBootTest; Vitest + React Testing Library (app)
**Target Platform**: navegador web (SPA) + REST API
**Project Type**: full-stack web: REST API + SPA
**Performance Goals**: resposta do resumo do dia em menos de 2 segundos, mesmo com o histórico do dia aberto (RNF-003)
**Constraints**: totais apenas de vendas ATIVA do dia (FR-005, RGN-007); Cartão somando crédito e débito em um único valor (RF-051); alertas somente para saldo de cheios no limite mínimo ou abaixo (RF-053, RF-032); dia sem vendas exibe zeros sem erro (Edge Case); leitura pura, sem escrita; sem DELETE de movimento (RNF-007)
**Scale/Scope**: revenda única, 1-5 usuários simultâneos, dados de 12 meses (RNF-010), agregação diária com dezenas a centenas de vendas por dia

## Constitution Check

Gates avaliados contra a CONSTITUICAO.md:

| Gate | Avaliação |
|---|---|
| §I-A Artefatos antes do código | spec.md, plan.md e task.md existem e são consistentes com a HU-019 - atendido |
| §II Vocabulário | usa carga/vasilhame, cheio, pátio; total de vendidos expresso em vasilhames/unidades - atendido |
| §III Invariantes de estoque | leitura pura; alerta usa saldo materializado, que respeita os três estados - atendido |
| §V Orientado a requisitos | feature nasce da HU-019 com CTs vinculados a RF-050 a RF-053 - atendido |
| §VI Proibido | sem escrita fora de transação (nenhuma escrita); vocabulário respeitado - atendido |
| §VII Documentação | pt-BR, hífen normal, sem travessão - atendido |
| §X Rastreabilidade | RF-050 a RF-053, RF-032, RGN-004, CT-001 a CT-004 referenciados - atendido |
| §XI Qualidade | CT provado por teste ou evidência; agregação conferida contra o relatório de vendas do dia (SC-002) - atendido |
| §XIII.2 Auditoria de cancelamento | vendas canceladas fora dos totais; o dashboard não altera rastro de cancelamento - atendido |

**GATE: PASS**

## Project Structure

### Documentation (this feature)

```text
specs/HU-019-Dashboard-Resumo-Dia/
├── spec.md                # Especificação: objetivo, CTs, requisitos vinculados
├── plan.md                # Este arquivo (fase de plano)
├── research.md            # Fase 0: decisões técnicas e alternativas
├── data-model.md          # Fase 1: modelo de dados da feature
├── quickstart.md          # Fase 1: validação end-to-end dos CTs
├── contracts/
│   ├── api.md             # Contrato REST /api/dashboard/resumo-dia
│   └── ui.md              # Contrato de UI da tela inicial
└── task.md                # Checklist de CTs com status (pendente/em andamento/concluído)
```

### Source Code (repository root)

```text
gerenciador_estoque_api/
└── src/
    ├── main/java/com/gerenciador/estoque/dashboard/
    │   ├── controller/DashboardController.java
    │   ├── service/ResumoDiaService.java            # agregação RF-050 a RF-053
    │   ├── repository/
    │   │   ├── ResumoDiaVendaRepository.java        # totais do dia por forma/produto
    │   │   └── ResumoDiaEstoqueRepository.java      # saldos e limite_minimo
    │   └── dto/ (ResumoDiaResponse, TotalPorFormaResponse, UnidadeVendidaResponse, AlertaEstoqueResponse)
    └── test/java/com/gerenciador/estoque/dashboard/
        ├── ResumoDiaServiceTest.java                # agregação, exclusão de cancelada
        ├── DashboardControllerTest.java
        └── ResumoDiaIntegrationTest.java            # banco real, conferência com relatório

gerenciador_estoque_app/
└── src/
    ├── features/dashboard/
    │   ├── DashboardPage.tsx                        # tela inicial (rota /)
    │   ├── components/ (CardTotalFaturado, CardsFormasPagamento, TabelaUnidades, AlertaEstoqueBaixo)
    │   └── types.ts
    ├── api/
    │   └── dashboardApi.ts                          # GET /api/dashboard/resumo-dia
    └── context/
        └── DashboardContext.tsx                     # refresh do resumo após lançamento
```

**Structure Decision**: monorepo com 5 repositórios irmãos: `gerenciador_estoque_api`, `gerenciador_estoque_app`, `gerenciador_estoque_infra`, `gerenciador_estoque_audi` (este diretório) e `gerenciador_estoque_jar` (Constituição §I). A feature é implementada na API no módulo `com.gerenciador.estoque.dashboard` (CONVENTIONS §4) e no app em `src/features/dashboard/` (CONVENTIONS §7). O dashboard é a tela inicial (RF-050 a RF-053, CONVENTIONS §7) e o alerta de estoque baixo (RF-032) permanece visível enquanto ativo (RGN-004).

## Complexity Tracking

Nenhuma violação dos gates da Constituição detectada; nada a justificar.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|---|---|---|
| - | - | - |
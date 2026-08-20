# Implementation Plan: Painel de Estoque em Tempo Real (Pátio)

**HU**: HU-012 - Painel de Estoque em Tempo Real (Pátio)

**Branch**: `HU-012-Painel-Estoque` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-012-Painel-Estoque/spec.md`

## Summary

Requisito primário: exibir em tempo real, por produto, os três estados do estoque - Cheios, Vazios (pátio) e Em rua (clientes) - (RF-030, CT-001), com atualização imediata após cada entrada, venda ou devolução sem recarga manual (CT-002) e destaque visual para produtos com estoque cheio no/abaixo do limite mínimo (RF-032, CT-003).

Abordagem técnica: endpoints de leitura `GET /api/estoque` (saldos por produto) e `GET /api/estoque/alertas` (produtos em alerta) no módulo `estoque` da API, lendo o saldo materializado em `tab_estoque` (junção com `tab_produto`) sem nenhuma escrita. No app, tela em `src/features/estoque/` que reconsulta os saldos automaticamente após qualquer mutação confirmada (venda, entrada, devolução) e em intervalo periódico curto, sem recarga manual (CT-002). O destaque de estoque baixo é uma flag calculada na API (saldo <= limite mínimo), consumida também pelo dashboard (RF-053) e pela HU-014.

## Technical Context

**Language/Version**: Java 21 + Spring Boot 3.x (API); React 18 + TypeScript 5.x (app)
**Primary Dependencies**: Spring Data JPA, Flyway, Lombok (API); React, Vite, axios, react-router (app)
**Storage**: PostgreSQL único + Flyway forward-only; tabelas `tab_*`, sequências `seq_*`, colunas snake_case pt-BR
**Testing**: JUnit 5 + Mockito (Controller/Service); integração com @SpringBootTest; Vitest + React Testing Library (app)
**Target Platform**: navegador web (SPA) + REST API
**Project Type**: full-stack web: REST API + SPA
**Performance Goals**: painel reflete a movimentação em menos de 1 segundo (SC-002); consulta de saldos em menos de 500 ms com histórico de 12 meses (RNF-010)
**Constraints**: leitura somente do saldo materializado (sem derivação por soma a cada request); sem escrita nesta feature; destaque de alerta derivado por comparação (RF-032), nunca por estado persistido; produto sem limite mínimo não gera falso alerta (Edge Case); sem DELETE de movimento (RNF-007)
**Scale/Scope**: revenda única, 1-5 usuários simultâneos, dados de 12 meses (RNF-010)

## Constitution Check

Gates avaliados contra a CONSTITUICAO.md:

| Gate | Avaliação |
|---|---|
| §I-A Artefatos antes do código | spec.md, plan.md e tasks.md existem e são consistentes com a HU-012 - atendido |
| §II Vocabulário | usa carga/vasilhame, cheio, vazio, em rua, pátio; sem sinônimos proibidos - atendido |
| §III Invariantes de estoque | exibição dos 3 estados sem alterar saldos; invariantes preservadas pelas features de escrita - atendido |
| §V Orientado a requisitos | feature nasce da HU-012 com CTs vinculados a RF-030/RF-032/RF-053 - atendido |
| §VI Proibido | sem DELETE, sem escrita fora de transação (nenhuma escrita aqui), vocabulário respeitado - atendido |
| §VII Documentação | pt-BR, hífen normal, sem travessão - atendido |
| §X Rastreabilidade | RF-030, RF-032, RF-053, CT-001 a CT-003 referenciados - atendido |
| §XI Qualidade | CT provado por teste ou evidência; consistência com HU-014 garantida pelo mesmo endpoint de alertas - atendido |

**GATE: PASS**

## Project Structure

### Documentation (this feature)

```text
specs/HU-012-Painel-Estoque/
├── spec.md                # Especificação: objetivo, CTs, requisitos vinculados
├── plan.md                # Este arquivo (fase de plano)
├── research.md            # Fase 0: decisões técnicas e alternativas
├── data-model.md          # Fase 1: modelo de dados da feature
├── quickstart.md          # Fase 1: validação end-to-end dos CTs
├── contracts/
│   ├── api.md             # Contrato REST /api/estoque
│   └── ui.md              # Contrato de UI do painel
└── tasks.md                # Checklist de CTs com status (pendente/em andamento/concluído)
```

### Source Code (repository root)

```text
gerenciador_estoque_api/
└── src/
    ├── main/java/com/gerenciador/estoque/estoque/
    │   ├── controller/EstoqueController.java
    │   ├── service/EstoqueConsultaService.java
    │   ├── repository/EstoqueRepository.java
    │   ├── repository/ProdutoRepository.java
    │   └── dto/
    │       ├── SaldoProdutoResponse.java
    │       └── AlertaEstoqueResponse.java
    └── test/java/com/gerenciador/estoque/estoque/
        ├── EstoqueConsultaServiceTest.java
        ├── EstoqueControllerTest.java
        └── EstoqueConsultaIntegrationTest.java

gerenciador_estoque_app/
└── src/
    ├── features/estoque/
    │   ├── PainelEstoquePage.tsx
    │   ├── components/ (CardSaldoProduto, BadgeAlertaEstoqueBaixo, TabelaSaldo)
    │   └── types.ts
    ├── api/
    │   └── estoqueApi.ts                              # cliente HTTP tipado
    └── hooks/
        └── useSaldosEstoque.ts                        # refetch automático após mutações
```

**Structure Decision**: monorepo com 5 repositórios irmãos: `gerenciador_estoque_api`, `gerenciador_estoque_app`, `gerenciador_estoque_infra`, `gerenciador_estoque_audi` (este diretório) e `gerenciador_estoque_jar` (Constituição §I). A feature é implementada na API no módulo `com.gerenciador.estoque.estoque` (CONVENTIONS §4) e no app em `src/features/estoque/` (CONVENTIONS §7). O mesmo endpoint `GET /api/estoque/alertas` é consumido pelo dashboard (RF-053, HU-014) para garantir consistência entre as telas.

## Complexity Tracking

Nenhuma violação dos gates da Constituição detectada; nada a justificar.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|---|---|---|
| - | - | - |
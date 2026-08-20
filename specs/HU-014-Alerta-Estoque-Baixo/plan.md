# Implementation Plan: Alerta de Estoque Baixo

**HU**: HU-014 - Alerta de Estoque Baixo

**Branch**: `HU-014-Alerta-Estoque-Baixo` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-014-Alerta-Estoque-Baixo/spec.md`

## Summary

Requisito primário: gerar alerta visual de estoque baixo quando o saldo de cheios de um produto atinge ou fica abaixo do limite mínimo configurado (RF-032, CT-001), visível no dashboard e no painel de estoque (RF-053, CT-002), persistente até nova entrada elevar o saldo acima do limite (RGN-004, CT-003), com sugestão de reposição baseada na média de vendas diárias (RGN-009, CT-004).

Abordagem técnica: feature de LEITURA sobre `tab_estoque` e `tab_venda_item`. O alerta é um estado DERIVADO: a API calcula `qtd_cheios <= limite_minimo` no endpoint `GET /api/estoque/alertas` (módulo `estoque`), reutilizado pelo dashboard (RF-053) e pelo painel (HU-012) para garantir fonte única da regra (RF-032). A sugestão de reposição é calculada a partir da média de vendas diárias dos últimos 30 dias em `tab_venda_item`, com tratamento de produto sem histórico (Edge Case). Como o alerta é derivado do saldo, ele persiste automaticamente até a entrada elevar o saldo acima do limite (RGN-004, CT-003) e reflete imediatamente vendas que derrubam o saldo durante o dia (Edge Case). No app, componentes reutilizáveis de alerta em `src/features/estoque/` exibidos no dashboard e no painel.

## Technical Context

**Language/Version**: Java 21 + Spring Boot 3.x (API); React 18 + TypeScript 5.x (app)
**Primary Dependencies**: Spring Data JPA, Flyway, Lombok (API); React, Vite, axios, react-router (app)
**Storage**: PostgreSQL único + Flyway forward-only; tabelas `tab_*`, sequências `seq_*`, colunas snake_case pt-BR
**Testing**: JUnit 5 + Mockito (Service/Controller); integração com @SpringBootTest; Vitest + React Testing Library (app)
**Target Platform**: navegador web (SPA) + REST API
**Project Type**: full-stack web: REST API + SPA
**Performance Goals**: alerta refletido imediatamente após venda ou entrada (Edge Case do spec); consulta de alertas e sugestão em menos de 1 segundo (RNF-003)
**Constraints**: alerta é derivado por comparação de saldo, nunca persistido (fonte única RF-032); sem escrita nesta feature; limite mínimo consumido de outra feature (RF-003, HU-003); alerta persiste até entrada elevar saldo acima do limite (RGN-004); produto sem limite configurado ou sem histórico não gera alerta inválido (Edge Cases); sem DELETE de movimento (RNF-007)
**Scale/Scope**: revenda única, 1-5 usuários simultâneos, dados de 12 meses (RNF-010)

## Constitution Check

Gates avaliados contra a CONSTITUICAO.md:

| Gate | Avaliação |
|---|---|
| §I-A Artefatos antes do código | spec.md, plan.md e tasks.md existem e são consistentes com a HU-014 - atendido |
| §II Vocabulário | usa carga/vasilhame, cheio, vazio, em rua, pátio; sem sinônimos proibidos - atendido |
| §III Invariantes de estoque | leitura pura; invariantes preservadas pelas features de escrita - atendido |
| §V Orientado a requisitos | feature nasce da HU-014 com CTs vinculados a RF-032/RF-053/RGN-004/RGN-009 - atendido |
| §VI Proibido | sem escrita fora de transação (nenhuma escrita); vocabulário respeitado - atendido |
| §VII Documentação | pt-BR, hífen normal, sem travessão - atendido |
| §X Rastreabilidade | RF-032, RF-053, RGN-004, RGN-009, CT-001 a CT-004 referenciados - atendido |
| §XI Qualidade | CT provado por teste ou evidência; regra de alerta (estoque) com prioridade máxima de cobertura - atendido |

**GATE: PASS**

## Project Structure

### Documentation (this feature)

```text
specs/HU-014-Alerta-Estoque-Baixo/
├── spec.md                # Especificação: objetivo, CTs, requisitos vinculados
├── plan.md                # Este arquivo (fase de plano)
├── research.md            # Fase 0: decisões técnicas e alternativas
├── data-model.md          # Fase 1: modelo de dados da feature
├── quickstart.md          # Fase 1: validação end-to-end dos CTs
├── contracts/
│   ├── api.md             # Contrato REST /api/estoque/alertas
│   └── ui.md              # Contrato de UI do alerta no dashboard e no painel
└── tasks.md                # Checklist de CTs com status (pendente/em andamento/concluído)
```

### Source Code (repository root)

```text
gerenciador_estoque_api/
└── src/
    ├── main/java/com/gerenciador/estoque/estoque/
    │   ├── controller/EstoqueController.java          # GET /api/estoque/alertas
    │   ├── service/EstoqueAlertaService.java
    │   ├── service/MediaVendasService.java            # média de vendas diárias (RGN-009)
    │   ├── repository/EstoqueRepository.java
    │   ├── repository/VendaItemRepository.java        # média de vendas por produto
    │   └── dto/AlertaEstoqueResponse.java
    └── test/java/com/gerenciador/estoque/estoque/
        ├── EstoqueAlertaServiceTest.java
        ├── MediaVendasServiceTest.java                # lógica pura, sem mock
        ├── EstoqueControllerTest.java
        └── EstoqueAlertaIntegrationTest.java

gerenciador_estoque_app/
└── src/
    ├── features/estoque/
    │   ├── components/AlertaEstoqueBaixo.tsx          # reutilizado no dashboard e no painel
    │   ├── components/BadgeAlertaEstoqueBaixo.tsx
    │   └── types.ts
    ├── api/
    │   └── estoqueApi.ts                              # cliente HTTP tipado
    ├── features/dashboard/
    │   └── DashboardPage.tsx                          # exibe AlertasEstoqueBaixo (RF-053)
    └── hooks/
        └── useAlertasEstoque.ts                       # refetch após venda/entrada
```

**Structure Decision**: monorepo com 5 repositórios irmãos: `gerenciador_estoque_api`, `gerenciador_estoque_app`, `gerenciador_estoque_infra`, `gerenciador_estoque_audi` (este diretório) e `gerenciador_estoque_jar` (Constituição §I). A feature é implementada na API no módulo `com.gerenciador.estoque.estoque` (CONVENTIONS §4) e no app em `src/features/estoque/`, com o componente de alerta reutilizado pelo dashboard em `src/features/dashboard/` (RF-053). O endpoint de alertas é o mesmo consumido pela HU-012, garantindo consistência.

## Complexity Tracking

Nenhuma violação dos gates da Constituição detectada; nada a justificar.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|---|---|---|
| - | - | - |
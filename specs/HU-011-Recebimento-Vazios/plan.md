# Implementation Plan: Recebimento de Vasilhames Vazios Avulsos

**HU**: HU-011 - Recebimento de Vasilhames Vazios Avulsos

**Branch**: `HU-011-Recebimento-Vazios` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-011-Recebimento-Vazios/spec.md`

## Summary

Requisito primário: permitir o lançamento de recebimento de vasilhames vazios avulsos, ou seja, devoluções fora de venda (RF-027), com produto obrigatório e cliente opcional (CT-001), incrementando o pátio de vazios na quantidade lançada (CT-002) e, quando o cliente possui vasilhames "em rua", baixando o comodato dele na mesma operação (RF-028, CT-003). Todo lançamento registra data/hora e usuário responsável (CT-004, RNF-007).

Abordagem técnica: endpoint `POST /api/devolucoes` no módulo `devolucao` da API (Java 21 + Spring Boot 3.x). O Service é `@Transactional`: dentro da transação, lê o saldo de `tab_estoque` do produto com lock pessimista (`PESSIMISTIC_WRITE`), grava o lançamento em `tab_devolucao_vazio`, registra a movimentação `ENTRADA_VAZIO` em `tab_movimentacao_estoque`, incrementa `qtd_vazios` e, se vinculado a cliente com comodato, baixa `qtd_em_rua` global e o saldo por cliente em `tab_cliente_em_rua` sem nunca gerar saldo negativo (RDN-005). No app, tela em `src/features/devolucao/` com cliente HTTP tipado (axios), nunca `fetch` solto (CONVENTIONS §7).

## Technical Context

**Language/Version**: Java 21 + Spring Boot 3.x (API); React 18 + TypeScript 5.x (app)
**Primary Dependencies**: Spring Data JPA, Flyway, Lombok (API); React, Vite, axios, react-router (app)
**Storage**: PostgreSQL único + Flyway forward-only; tabelas `tab_*`, sequências `seq_*`, colunas snake_case pt-BR
**Testing**: JUnit 5 + Mockito (Service/Controller); integração com @SpringBootTest; Vitest + React Testing Library (app)
**Target Platform**: navegador web (SPA) + REST API
**Project Type**: full-stack web: REST API + SPA
**Performance Goals**: lançamento de recebimento respondendo em até 2 segundos (RNF-003); saldo refletido na mesma transação (RNF-005)
**Constraints**: transação atômica estoque + registro do movimento (CONVENTIONS §5.1, RNF-005); lock por produto ao consultar saldo dentro da transação (RNF-008); sem DELETE de movimento, apenas cancelamento com reversão (RNF-007); erro de validação não colapsa em estado de negócio (Constituição §XI.3); pátio de vazios nunca negativo (RDN-005)
**Scale/Scope**: revenda única, 1-5 usuários simultâneos, histórico de 12 meses (RNF-010)

## Constitution Check

Gates avaliados contra a CONSTITUICAO.md:

| Gate | Avaliação |
|---|---|
| §I-A Artefatos antes do código | spec.md, plan.md e tasks.md existem e são consistentes com a HU-011 - atendido |
| §II Vocabulário | usa carga/vasilhame, cheio, vazio, em rua, pátio; sem sinônimos proibidos - atendido |
| §III Invariantes de estoque | vasilhame em exatamente um estado; pátio nunca negativo; movimentação e estoque na mesma transação; nada apagado - atendido |
| §V Orientado a requisitos | feature nasce da HU-011 com CTs vinculados a RF-027/RF-028 - atendido |
| §VI Proibido | sem DELETE de movimento; sem estoque fora de transação; sem divergir do vocabulário - atendido |
| §VII Documentação | pt-BR, hífen normal, sem travessão - atendido |
| §X Rastreabilidade | RF-027, RF-028, CT-001 a CT-004 referenciados em código e docs - atendido |
| §XI Qualidade | CT provado por teste ou evidência; regra de estoque com prioridade máxima de cobertura - atendido |

**GATE: PASS**

## Project Structure

### Documentation (this feature)

```text
specs/HU-011-Recebimento-Vazios/
├── spec.md                # Especificação: objetivo, CTs, requisitos vinculados
├── plan.md                # Este arquivo (fase de plano)
├── research.md            # Fase 0: decisões técnicas e alternativas
├── data-model.md          # Fase 1: modelo de dados da feature
├── quickstart.md          # Fase 1: validação end-to-end dos CTs
├── contracts/
│   ├── api.md             # Contrato REST /api/devolucoes
│   └── ui.md              # Contrato de UI da tela de recebimento
└── tasks.md                # Checklist de CTs com status (pendente/em andamento/concluído)
```

### Source Code (repository root)

```text
gerenciador_estoque_api/
└── src/
    ├── main/java/com/gerenciador/estoque/devolucao/
    │   ├── controller/DevolucaoVazioController.java
    │   ├── service/DevolucaoVazioService.java          # @Transactional
    │   ├── repository/DevolucaoVazioRepository.java
    │   ├── repository/MovimentacaoEstoqueRepository.java
    │   ├── dto/RecebimentoVazioRequest.java
    │   └── dto/RecebimentoVazioResponse.java
    ├── main/java/com/gerenciador/estoque/estoque/
    │   ├── repository/EstoqueRepository.java           # lock PESSIMISTIC_WRITE
    │   └── repository/ClienteEmRuaRepository.java
    ├── main/resources/db/migration/
    │   └── V<N>__criar_tab_devolucao_vazio.sql
    └── test/java/com/gerenciador/estoque/devolucao/
        ├── DevolucaoVazioServiceTest.java
        ├── DevolucaoVazioControllerTest.java
        └── DevolucaoVazioIntegrationTest.java

gerenciador_estoque_app/
└── src/
    ├── features/devolucao/
    │   ├── RecebimentoVaziosPage.tsx
    │   ├── components/ (ProdutoSelect, ClienteSelect, QuantidadeInput)
    │   └── types.ts
    ├── api/
    │   └── devolucoesApi.ts                             # cliente HTTP tipado
    └── components/ (layout, mensagens, estilos comuns)
```

**Structure Decision**: monorepo com 5 repositórios irmãos: `gerenciador_estoque_api` (backend), `gerenciador_estoque_app` (frontend), `gerenciador_estoque_infra` (banco/deploy), `gerenciador_estoque_audi` (documentação e auditoria, este diretório) e `gerenciador_estoque_jar` (JAR executável) (Constituição §I). A feature é implementada na API no módulo `com.gerenciador.estoque.devolucao` (CONVENTIONS §4) e no app em `src/features/devolucao/` (CONVENTIONS §7). Migrations Flyway versionadas e forward-only em `gerenciador_estoque_api/src/main/resources/db/migration` (CONVENTIONS §8).

## Complexity Tracking

Nenhuma violação dos gates da Constituição detectada; nada a justificar.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|---|---|---|
| - | - | - |
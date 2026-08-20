# Implementation Plan: Fechamento de Caixa e Balanço de Estoque

**HU**: HU-018 - Fechamento de Caixa e Balanço de Estoque

**Branch**: `HU-018-Fechamento-Caixa-Balanco` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-018-Fechamento-Caixa-Balanco/spec.md`

## Summary

Requisito primário: gerar o balanço de estoque por período com as colunas Produto, Estoque Inicial, (+) Entradas, (-) Vendas, Estoque Final, Vazios em Pátio (RF-043, CT-001), com estoque inicial calculado pelas movimentações anteriores ao período (CT-004), e concluir o fechamento de caixa do dia somente com todas as vendas conciliadas (RGN-006, CT-002), com exportação em Excel/CSV (RF-044, CT-003).

Abordagem técnica: módulo `caixa` na API com `GET /api/caixa/fechamento?data=` (resumo do dia: totais por forma de pagamento, status do fechamento e vendas pendentes de conciliação) e `POST /api/caixa/fechar` (grava `tab_fechamento_caixa` com status FECHADO em transação única, recalculando os totais das vendas ATIVA do dia e bloqueando se houver pendência - RGN-006). O balanço de estoque deriva de `tab_movimentacao_estoque` + saldo atual de `tab_estoque` (RF-043, CT-001/CT-004) e é exportado pelo endpoint consolidado `GET /api/relatorios/balanco` da HU-015, com colunas EXATAS de CONVENTIONS §10 (RF-044, CT-003). No app, tela de fechamento em `src/features/caixa/` e seção Balanço no painel de relatórios.

## Technical Context

**Language/Version**: Java 21 + Spring Boot 3.x (API); React 18 + TypeScript 5.x (app)
**Primary Dependencies**: Spring Data JPA, Flyway, Lombok (API); React, Vite, axios, react-router (app)
**Storage**: PostgreSQL único + Flyway forward-only; tabelas `tab_*`, sequências `seq_*`, colunas snake_case pt-BR; nova tabela `tab_fechamento_caixa` via migration `V<N>__cria_tab_fechamento_caixa.sql`
**Testing**: JUnit 5 + Mockito (Service/Controller); integração com @SpringBootTest; Vitest + React Testing Library (app)
**Target Platform**: navegador web (SPA) + REST API
**Project Type**: full-stack web: REST API + SPA
**Performance Goals**: consulta do resumo do dia e fechamento em menos de 2 segundos (RNF-003); balanço de até 12 meses exportado em poucos segundos (RNF-010)
**Constraints**: fechamento só conclui com todas as vendas do dia conciliadas (RGN-006, CT-002); um único fechamento por data; totais do fechamento = soma das vendas ATIVA do dia por forma de pagamento (RGN-008); balanço com colunas exatas de CONVENTIONS §10 (RF-043); vendas canceladas não entram nas vendas (-) do balanço (FR-005); escrita transacional única; sem DELETE de movimento (RNF-007)
**Scale/Scope**: revenda única, 1-5 usuários simultâneos, dados de 12 meses (RNF-010), um fechamento por dia

## Constitution Check

Gates avaliados contra a CONSTITUICAO.md:

| Gate | Avaliação |
|---|---|
| §I-A Artefatos antes do código | spec.md, plan.md e task.md existem e são consistentes com a HU-018 - atendido |
| §II Vocabulário | usa carga/vasilhame, cheio, vazio, pátio, carregamento; cabeçalhos seguem CONVENTIONS §10 - atendido |
| §III Invariantes de estoque | balanço reflete os três estados; fechamento não altera estoque; invariantes preservadas - atendido |
| §V Orientado a requisitos | feature nasce da HU-018 com CTs vinculados a RF-043/RF-044/RGN-006 - atendido |
| §VI Proibido | sem DELETE de movimento; fechamento não apaga venda; vocabulário respeitado - atendido |
| §VII Documentação | pt-BR, hífen normal, sem travessão - atendido |
| §X Rastreabilidade | RF-043, RF-044, RGN-006, CT-001 a CT-004 referenciados - atendido |
| §XI Qualidade | regra de caixa com prioridade máxima de cobertura (CONVENTIONS §11); CT provado por teste ou evidência - atendido |
| §XIII.2 Auditoria de cancelamento | fechamento registra usuário responsável (id_usuario_fechamento); vendas canceladas saem das somas do dia - atendido |

**GATE: PASS**

## Project Structure

### Documentation (this feature)

```text
specs/HU-018-Fechamento-Caixa-Balanco/
├── spec.md                # Especificação: objetivo, CTs, requisitos vinculados
├── plan.md                # Este arquivo (fase de plano)
├── research.md            # Fase 0: decisões técnicas e alternativas
├── data-model.md          # Fase 1: modelo de dados da feature
├── quickstart.md          # Fase 1: validação end-to-end dos CTs
├── contracts/
│   ├── api.md             # Contrato REST /api/caixa e /api/relatorios/balanco
│   └── ui.md              # Contrato de UI do fechamento e do balanço
└── task.md                # Checklist de CTs com status (pendente/em andamento/concluído)
```

### Source Code (repository root)

```text
gerenciador_estoque_api/
└── src/
    ├── main/java/com/gerenciador/estoque/caixa/
    │   ├── controller/CaixaController.java
    │   ├── service/
    │   │   ├── FechamentoCaixaService.java        # conciliação (RGN-006) + fechamento
    │   │   └── ResumoCaixaService.java            # resumo do dia por forma de pagamento
    │   ├── repository/
    │   │   ├── FechamentoCaixaRepository.java
    │   │   └── ResumoVendaRepository.java         # totais do dia (ATIVA)
    │   └── dto/ (ResumoCaixaResponse, FecharCaixaRequest, FechamentoCaixaResponse)
    └── test/java/com/gerenciador/estoque/caixa/
        ├── FechamentoCaixaServiceTest.java        # conciliação, bloqueio, unicidade
        ├── ResumoCaixaServiceTest.java            # agregação por forma de pagamento
        ├── CaixaControllerTest.java
        └── FechamentoCaixaIntegrationTest.java    # banco real, RGN-006

gerenciador_estoque_app/
└── src/
    ├── features/caixa/
    │   ├── FechamentoCaixaPage.tsx                # resumo do dia + botão fechar
    │   ├── components/ (CardsTotais, ListaVendasPendentes, ModalConfirmarFechamento)
    │   └── types.ts
    ├── features/relatorio/                        # seção Balanço (mesma do HU-015)
    │   ├── components/ (TabelaBalanco, BotaoExportar)
    │   └── types.ts
    └── api/
        └── caixaApi.ts                            # GET /api/caixa/fechamento, POST /api/caixa/fechar
```

**Structure Decision**: monorepo com 5 repositórios irmãos: `gerenciador_estoque_api`, `gerenciador_estoque_app`, `gerenciador_estoque_infra`, `gerenciador_estoque_audi` (este diretório) e `gerenciador_estoque_jar` (Constituição §I). A feature é implementada na API no módulo `com.gerenciador.estoque.caixa` (CONVENTIONS §4) e no app em `src/features/caixa/` (CONVENTIONS §7). A exportação do balanço reutiliza o endpoint `GET /api/relatorios/balanco` da HU-015, garantindo colunas exatas (CONVENTIONS §10, RF-043/RF-044). `tab_fechamento_caixa` nasce da migration desta feature e é consumida pelo caixa; o balanço permanece derivado de `tab_movimentacao_estoque` e `tab_estoque` (não duplica estado).

## Complexity Tracking

Nenhuma violação dos gates da Constituição detectada; nada a justificar.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|---|---|---|
| - | - | - |
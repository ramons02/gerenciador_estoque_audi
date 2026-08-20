# Implementation Plan: Exportação de Relatórios (Período Personalizado)

**HU**: HU-015 - Exportação de Relatórios (Período Personalizado)

**Branch**: `HU-015-Exportacao-Relatorios` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-015-Exportacao-Relatorios/spec.md`

## Summary

Requisito primário: exportar relatórios em Excel/CSV por período - Hoje, Últimos 7 dias, Mês Atual ou período personalizado - (RF-040, CT-001), com botão de exportação para cada relatório (RF-044, CT-002), arquivos compatíveis com Excel, LibreOffice e Google Sheets (RNF-009, CT-003) e cabeçalhos claros em pt-BR (CT-004).

Abordagem técnica: endpoints de leitura no módulo `relatorio` da API: `GET /api/relatorios/vendas`, `GET /api/relatorios/carregamentos` e `GET /api/relatorios/balanco`, todos com parâmetro `periodo` (HOJE|7DIAS|MES|PERSONALIZADO) e, para personalizado, `inicio`/`fim` (RF-040). A resposta é JSON quando o app consome para exibição, e CSV quando o cliente pede `Accept: text/csv` ou `?formato=csv` (RF-044), com BOM UTF-8, cabeçalhos e colunas EXATAS conforme CONVENTIONS §10 (RF-041/042/043). O balanço de estoque deriva Estoque Inicial e Estoque Final a partir de `tab_movimentacao_estoque` e do saldo atual de `tab_estoque` (RF-043). No app, painel de relatórios em `src/features/relatorio/` com seletor de período e botão de exportação por relatório (CT-002).

## Technical Context

**Language/Version**: Java 21 + Spring Boot 3.x (API); React 18 + TypeScript 5.x (app)
**Primary Dependencies**: Spring Data JPA, Flyway, Lombok (API); React, Vite, axios, react-router (app)
**Storage**: PostgreSQL único + Flyway forward-only; tabelas `tab_*`, sequências `seq_*`, colunas snake_case pt-BR
**Testing**: JUnit 5 + Mockito (Service/Controller); integração com @SpringBootTest; Vitest + React Testing Library (app)
**Target Platform**: navegador web (SPA) + REST API
**Project Type**: full-stack web: REST API + SPA
**Performance Goals**: exportação CSV de períodos de até 12 meses gerada em poucos segundos (RNF-003, RNF-010); resposta JSON da consulta do painel em menos de 2 segundos
**Constraints**: colunas exatas por relatório conforme CONVENTIONS §10 (RF-041/042/043); CSV UTF-8 com BOM para compatibilidade (RNF-009); período personalizado exige fim >= inicio (Edge Case); período sem movimentações gera arquivo válido com zero linhas (Edge Case); leitura pura, sem escrita; sem DELETE de movimento (RNF-007)
**Scale/Scope**: revenda única, 1-5 usuários simultâneos, dados de 12 meses (RNF-010), exportação de milhares de linhas em CSV

## Constitution Check

Gates avaliados contra a CONSTITUICAO.md:

| Gate | Avaliação |
|---|---|
| §I-A Artefatos antes do código | spec.md, plan.md e task.md existem e são consistentes com a HU-015 - atendido |
| §II Vocabulário | usa carga/vasilhame, cheio, vazio, pátio; cabeçalhos seguem CONVENTIONS §10 - atendido |
| §III Invariantes de estoque | leitura pura; invariantes preservadas pelas features de escrita - atendido |
| §V Orientado a requisitos | feature nasce da HU-015 com CTs vinculados a RF-040/RF-044/RNF-009 - atendido |
| §VI Proibido | sem escrita fora de transação (nenhuma escrita); vocabulário respeitado - atendido |
| §VII Documentação | pt-BR, hífen normal, sem travessão - atendido |
| §X Rastreabilidade | RF-040 a RF-044, RNF-009, CT-001 a CT-004 referenciados - atendido |
| §XI Qualidade | CT provado por teste ou evidência; exportação com formato validado contra CONVENTIONS §10 - atendido |

**GATE: PASS**

## Project Structure

### Documentation (this feature)

```text
specs/HU-015-Exportacao-Relatorios/
├── spec.md                # Especificação: objetivo, CTs, requisitos vinculados
├── plan.md                # Este arquivo (fase de plano)
├── research.md            # Fase 0: decisões técnicas e alternativas
├── data-model.md          # Fase 1: modelo de dados da feature
├── quickstart.md          # Fase 1: validação end-to-end dos CTs
├── contracts/
│   ├── api.md             # Contrato REST /api/relatorios
│   └── ui.md              # Contrato de UI do painel de relatórios
└── task.md                # Checklist de CTs com status (pendente/em andamento/concluído)
```

### Source Code (repository root)

```text
gerenciador_estoque_api/
└── src/
    ├── main/java/com/gerenciador/estoque/relatorio/
    │   ├── controller/RelatorioController.java
    │   ├── service/
    │   │   ├── RelatorioVendasService.java
    │   │   ├── RelatorioCarregamentosService.java
    │   │   └── RelatorioBalancoService.java
    │   ├── repository/
    │   │   ├── RelatorioVendaRepository.java
    │   │   ├── RelatorioCarregamentoRepository.java
    │   │   └── RelatorioMovimentacaoRepository.java
    │   ├── dto/ (PeriodoRequest, LinhaRelatorio..., ...)
    │   └── export/
    │       ├── CsvExporter.java                      # BOM UTF-8 + escape CSV
    │       └── CabeçalhosRelatorio.java              # colunas CONVENTIONS §10
    └── test/java/com/gerenciador/estoque/relatorio/
        ├── CsvExporterTest.java                      # lógica pura, sem mock
        ├── RelatorioVendasServiceTest.java
        ├── RelatorioControllerTest.java
        └── RelatorioExportacaoIntegrationTest.java   # arquivo abre + colunas exatas

gerenciador_estoque_app/
└── src/
    ├── features/relatorio/
    │   ├── PainelRelatoriosPage.tsx
    │   ├── components/ (SeletorPeriodo, BotaoExportar, TabelaRelatorio)
    │   └── types.ts
    ├── api/
    │   └── relatoriosApi.ts                          # cliente HTTP tipado
    └── utils/
        └── downloadArquivo.ts                        # dispara download do CSV
```

**Structure Decision**: monorepo com 5 repositórios irmãos: `gerenciador_estoque_api`, `gerenciador_estoque_app`, `gerenciador_estoque_infra`, `gerenciador_estoque_audi` (este diretório) e `gerenciador_estoque_jar` (Constituição §I). A feature é implementada na API no módulo `com.gerenciador.estoque.relatorio` (CONVENTIONS §4) e no app em `src/features/relatorio/` (CONVENTIONS §7). A geração de CSV é centralizada em `CsvExporter` com os cabeçalhos canônicos de CONVENTIONS §10, garantindo colunas exatas nos três relatórios (RF-041/042/043).

## Complexity Tracking

Nenhuma violação dos gates da Constituição detectada; nada a justificar.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|---|---|---|
| - | - | - |
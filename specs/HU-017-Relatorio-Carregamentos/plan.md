# Implementation Plan: Relatório de Carregamentos (Entradas)

**HU**: HU-017 - Relatório de Carregamentos (Entradas)

**Branch**: `HU-017-Relatorio-Carregamentos` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-017-Relatorio-Carregamentos/spec.md`

## Summary

Requisito primário: gerar o relatório de carregamentos por período com as colunas padrão - Data, Fornecedor, Produto, Qtd Cheios Entraram, Qtd Vazios Saíram, Custo Total - (RF-042, CT-001), filtrando por Hoje, Últimos 7 dias, Mês Atual ou período personalizado (RF-040, CT-002), com exportação em Excel/CSV (RF-044, CT-003), consolidando apenas carregamentos registrados e confirmados (FR-004).

Abordagem técnica: módulo `carregamento` na API com `GET /api/carregamentos/relatorio?periodo=HOJE|7DIAS|MES|PERSONALIZADO&inicio=&fim=` (RF-040, RF-042), consulta de leitura em `tab_carregamento` + `tab_carregamento_item` + `tab_fornecedor` (RDN-009: fluxo duplo cheios entram e vazios saem). A exportação CSV reutiliza o endpoint consolidado `GET /api/relatorios/carregamentos` definido na HU-015, com BOM UTF-8 e colunas EXATAS de CONVENTIONS §10 (RF-044, CT-003, RNF-009). No app, a seção Relatório de Carregamentos em `src/features/relatorio/` com seletor de período e botão de exportação (CT-002, CT-003).

## Technical Context

**Language/Version**: Java 21 + Spring Boot 3.x (API); React 18 + TypeScript 5.x (app)
**Primary Dependencies**: Spring Data JPA, Flyway, Lombok (API); React, Vite, axios, react-router (app)
**Storage**: PostgreSQL único + Flyway forward-only; tabelas `tab_*`, sequências `seq_*`, colunas snake_case pt-BR
**Testing**: JUnit 5 + Mockito (Service/Controller); integração com @SpringBootTest; Vitest + React Testing Library (app)
**Target Platform**: navegador web (SPA) + REST API
**Project Type**: full-stack web: REST API + SPA
**Performance Goals**: consulta do relatório em menos de 2 segundos (RNF-003); exportação de período de até 12 meses gerada em poucos segundos (RNF-010)
**Constraints**: colunas exatas de CONVENTIONS §10 (RF-042); apenas carregamentos confirmados (FR-004); `qtd_vazios_sairam` nunca ultrapassa o saldo do pátio na escrita (RDN-003); período personalizado exige fim >= inicio (Edge Case); período sem carregamentos retorna lista vazia sem erro (Edge Case); leitura pura, sem escrita; sem DELETE de movimento (RNF-007)
**Scale/Scope**: revenda única, 1-5 usuários simultâneos, dados de 12 meses (RNF-010), poucas dezenas de carregamentos por mês

## Constitution Check

Gates avaliados contra a CONSTITUICAO.md:

| Gate | Avaliação |
|---|---|
| §I-A Artefatos antes do código | spec.md, plan.md e task.md existem e são consistentes com a HU-017 - atendido |
| §II Vocabulário | usa carga/vasilhame, cheio, vazio, pátio, carregamento, fornecedor; cabeçalhos seguem CONVENTIONS §10 - atendido |
| §III Invariantes de estoque | leitura pura; invariantes (RDN-003, RDN-009) preservadas pelas features de escrita - atendido |
| §V Orientado a requisitos | feature nasce da HU-017 com CTs vinculados a RF-040/RF-042/RF-044 - atendido |
| §VI Proibido | sem escrita fora de transação (nenhuma escrita); vocabulário respeitado - atendido |
| §VII Documentação | pt-BR, hífen normal, sem travessão - atendido |
| §X Rastreabilidade | RF-040, RF-042, RF-044, CT-001 a CT-003 referenciados - atendido |
| §XI Qualidade | CT provado por teste ou evidência; formato validado contra CONVENTIONS §10 - atendido |
| §XIII.2 Auditoria de cancelamento | feature não realiza cancelamento; consulta apenas carregamentos confirmados - atendido |

**GATE: PASS**

## Project Structure

### Documentation (this feature)

```text
specs/HU-017-Relatorio-Carregamentos/
├── spec.md                # Especificação: objetivo, CTs, requisitos vinculados
├── plan.md                # Este arquivo (fase de plano)
├── research.md            # Fase 0: decisões técnicas e alternativas
├── data-model.md          # Fase 1: modelo de dados da feature
├── quickstart.md          # Fase 1: validação end-to-end dos CTs
├── contracts/
│   ├── api.md             # Contrato REST /api/carregamentos/relatorio e CSV
│   └── ui.md              # Contrato de UI da seção Relatório de Carregamentos
└── task.md                # Checklist de CTs com status (pendente/em andamento/concluído)
```

### Source Code (repository root)

```text
gerenciador_estoque_api/
└── src/
    ├── main/java/com/gerenciador/estoque/carregamento/
    │   ├── controller/CarregamentoController.java
    │   ├── service/RelatorioCarregamentoService.java   # filtro por período
    │   ├── repository/CarregamentoRepository.java      # query do relatório (projection)
    │   └── dto/ (PeriodoRequest, LinhaRelatorioCarregamentoResponse, RelatorioCarregamentoResponse)
    └── test/java/com/gerenciador/estoque/carregamento/
        ├── RelatorioCarregamentoServiceTest.java       # filtro, produto/fornecedor
        ├── CarregamentoControllerTest.java             # validação de período
        └── RelatorioCarregamentoIntegrationTest.java   # banco real, colunas exatas

gerenciador_estoque_app/
└── src/
    ├── features/relatorio/
    │   ├── PainelRelatoriosPage.tsx                    # seletor de período compartilhado
    │   ├── components/ (SeletorPeriodo, BotaoExportar, TabelaRelatorioCarregamentos)
    │   └── types.ts                                    # tipos do relatório de carregamentos
    ├── api/
    │   ├── carregamentosApi.ts                         # GET /api/carregamentos/relatorio
    │   └── relatoriosApi.ts                            # exportação CSV (HU-015)
    └── utils/
        └── downloadArquivo.ts                          # dispara download do CSV
```

**Structure Decision**: monorepo com 5 repositórios irmãos: `gerenciador_estoque_api`, `gerenciador_estoque_app`, `gerenciador_estoque_infra`, `gerenciador_estoque_audi` (este diretório) e `gerenciador_estoque_jar` (Constituição §I). A feature é implementada na API no módulo `com.gerenciador.estoque.carregamento` (CONVENTIONS §4) e no app em `src/features/relatorio/` (CONVENTIONS §7). A exportação CSV não é duplicada: o endpoint `GET /api/relatorios/carregamentos` da HU-015 é o contrato canônico de exportação, garantindo colunas exatas (CONVENTIONS §10, RF-042/RF-044).

## Complexity Tracking

Nenhuma violação dos gates da Constituição detectada; nada a justificar.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|---|---|---|
| - | - | - |
# Implementation Plan: Cancelamento/Estorno de Venda

**HU**: HU-020 - Cancelamento/Estorno de Venda

**Branch**: `HU-020-Cancelamento-Venda` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-020-Cancelamento-Venda/spec.md`

## Summary

Requisito primário: cancelar/estornar uma venda com reversão automática de estoque e caixa na MESMA transação (RGN-007, CT-001), mantendo o registro histórico com status "cancelado" (RNF-007, CT-002) e registrando quem, quando e o motivo (RNF-006, §XIII.2, CT-003), com relatórios e dashboard excluindo vendas canceladas dos totais (CT-004).

Abordagem técnica: módulo `venda` na API com `PUT /api/vendas/{id}/cancelar` (corpo: `motivo`). O serviço `@Transactional` valida a venda (existente, status ATIVA - recusa cancelamento duplicado, FR-006), aplica `@Lock(LockModeType.PESSIMISTIC_WRITE)` por produto dos itens (RNF-008, CONVENTIONS §5.3/§8), reverte o estoque simetricamente conforme o tipo de venda (troca: cheio volta e vazio sai do pátio; vasilhame novo/avulsa: casco "em rua" volta a cheios - RF-023/024/026, RDN-004/008), grava movimentações de estorno em `tab_movimentacao_estoque` (tipos ESTORNO_*, fora das somas do balanço - RF-043, FR-005), marca a venda como CANCELADA com `motivo_cancelamento`, `data_hora_cancelamento` e `id_usuario_cancelamento` (auditoria §XIII.2, RNF-006/007). A reversão do caixa é implícita: a venda CANCELADA sai das somas de dashboard, relatórios e fechamento de caixa (CT-004, RGN-007). No app, ação de cancelar no detalhe do histórico em `src/features/venda/` com modal de confirmação e motivo obrigatório.

## Technical Context

**Language/Version**: Java 21 + Spring Boot 3.x (API); React 18 + TypeScript 5.x (app)
**Primary Dependencies**: Spring Data JPA, Flyway, Lombok (API); React, Vite, axios, react-router (app)
**Storage**: PostgreSQL único + Flyway forward-only; tabelas `tab_*`, sequências `seq_*`, colunas snake_case pt-BR; migration `V<N>__adiciona_campos_cancelamento_tab_venda.sql` (motivo_cancelamento, data_hora_cancelamento, id_usuario_cancelamento)
**Testing**: JUnit 5 + Mockito (Service/Controller); integração com @SpringBootTest; Vitest + React Testing Library (app)
**Target Platform**: navegador web (SPA) + REST API
**Project Type**: full-stack web: REST API + SPA
**Performance Goals**: cancelamento concluído em menos de 2 segundos (RNF-003); reversão atômica sem estado parcial (RNF-005)
**Constraints**: estoque e caixa revertidos na MESMA transação (RGN-007, CONVENTIONS §5.1); lock pessimista por produto (RNF-008); reversão nunca gera estoque negativo (Edge Case, RDN-005); motivo obrigatório; recusa de venda já cancelada (FR-006); sem DELETE físico do registro (RNF-007); auditoria com quem/quando/motivo (Constituição §XIII.2)
**Scale/Scope**: revenda única, 1-5 usuários simultâneos, dados de 12 meses (RNF-010), poucos cancelamentos por dia, todos auditados

## Constitution Check

Gates avaliados contra a CONSTITUICAO.md:

| Gate | Avaliação |
|---|---|
| §I-A Artefatos antes do código | spec.md, plan.md e tasks.md existem e são consistentes com a HU-020 - atendido |
| §II Vocabulário | usa carga/vasilhame, cheio, vazio, pátio, em rua, troca - atendido |
| §III Invariantes de estoque | reversão na mesma transação do cancelamento; estoque nunca negativo; nenhum estado parcial - atendido |
| §V Orientado a requisitos | feature nasce da HU-020 com CTs vinculados a RGN-007/RNF-006/RNF-007 - atendido |
| §VI Proibido | sem DELETE de movimento; cancelamento com reversão e rastro - atendido |
| §VII Documentação | pt-BR, hífen normal, sem travessão - atendido |
| §X Rastreabilidade | RGN-007, RNF-006, RNF-007, RF-043, CT-001 a CT-004 referenciados - atendido |
| §XI Qualidade | regra de estoque/caixa com prioridade máxima de cobertura (CONVENTIONS §11); teste de concorrência (RNF-008) - atendido |
| §XIII.2 Auditoria de cancelamento | cancelamento registra quem, quando e o motivo, sem exceção - atendido |

**GATE: PASS**

## Project Structure

### Documentation (this feature)

```text
specs/HU-020-Cancelamento-Venda/
├── spec.md                # Especificação: objetivo, CTs, requisitos vinculados
├── plan.md                # Este arquivo (fase de plano)
├── research.md            # Fase 0: decisões técnicas e alternativas
├── data-model.md          # Fase 1: modelo de dados da feature
├── quickstart.md          # Fase 1: validação end-to-end dos CTs
├── contracts/
│   ├── api.md             # Contrato REST PUT /api/vendas/{id}/cancelar
│   └── ui.md              # Contrato de UI do cancelamento no histórico
└── tasks.md                # Checklist de CTs com status (pendente/em andamento/concluído)
```

### Source Code (repository root)

```text
gerenciador_estoque_api/
└── src/
    ├── main/java/com/gerenciador/estoque/venda/
    │   ├── controller/VendaController.java
    │   ├── service/
    │   │   ├── CancelamentoVendaService.java        # @Transactional: reversão estoque+caixa
    │   │   └── ReversaoEstoqueService.java          # reversão simétrica por tipo de venda
    │   ├── repository/
    │   │   ├── VendaRepository.java                 # consulta da venda com itens
    │   │   └── EstoqueRepository.java               # @Lock(PESSIMISTIC_WRITE) por produto
    │   ├── entity/ (Venda, VendaItem, MovimentacaoEstoque, EstoqueProduto)
    │   └── dto/ (CancelarVendaRequest, CancelarVendaResponse)
    └── test/java/com/gerenciador/estoque/venda/
        ├── CancelamentoVendaServiceTest.java        # reversão troca/novo, duplicado, motivo
        ├── ReversaoEstoqueServiceTest.java          # lógica pura, sem mock
        ├── VendaControllerTest.java
        └── CancelamentoVendaIntegrationTest.java    # banco real: reversão + concorrência

gerenciador_estoque_app/
└── src/
    ├── features/venda/
    │   ├── HistoricoVendasPage.tsx                  # histórico com status e rastro
    │   ├── components/ (DetalheVenda, ModalCancelarVenda, BadgeStatusCancelada)
    │   └── types.ts
    ├── api/
    │   └── vendasApi.ts                             # PUT /api/vendas/{id}/cancelar
    └── context/
        └── DashboardContext.tsx                     # refresh do resumo após cancelar
```

**Structure Decision**: monorepo com 5 repositórios irmãos: `gerenciador_estoque_api`, `gerenciador_estoque_app`, `gerenciador_estoque_infra`, `gerenciador_estoque_audi` (este diretório) e `gerenciador_estoque_jar` (Constituição §I). A feature é implementada na API no módulo `com.gerenciador.estoque.venda` (CONVENTIONS §4) e no app em `src/features/venda/` (CONVENTIONS §7). O lock pessimista por produto (CONVENTIONS §8) garante a invariante de estoque não negativo sob concorrência (RNF-008); a reversão do caixa é implícita pela exclusão de CANCELADA nas somas (RGN-007), mantendo dashboard, relatórios e fechamento de caixa consistentes (CT-004).

## Complexity Tracking

Nenhuma violação dos gates da Constituição detectada; nada a justificar.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|---|---|---|
| - | - | - |
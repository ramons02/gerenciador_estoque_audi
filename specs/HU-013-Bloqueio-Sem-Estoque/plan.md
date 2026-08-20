# Implementation Plan: Bloqueio de Venda sem Estoque

**HU**: HU-013 - Bloqueio de Venda sem Estoque

**Branch**: `HU-013-Bloqueio-Sem-Estoque` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-013-Bloqueio-Sem-Estoque/spec.md`

## Summary

Requisito primário: bloquear a venda quando a quantidade solicitada excede o saldo de cheios disponível, com mensagem clara (RF-031, CT-001), garantindo que vendas simultâneas nunca gerem estoque negativo (RNF-008, RDN-005, CT-002), em todos os fluxos de venda: balcão, entrega, troca e vasilhame novo (CT-003).

Abordagem técnica: a validação de saldo fica centralizada no Service de venda (`com.gerenciador.estoque.venda`), `@Transactional`: antes de gravar, consulta `tab_estoque` do produto com lock pessimista (`PESSIMISTIC_WRITE`) e compara a quantidade solicitada com `qtd_cheios`; insuficiente retorna erro estruturado 409 `ESTOQUE_INSUFICIENTE` com mensagem pt-BR (CONVENTIONS §6), sem baixar nada. A baixa (cheios -N, vazios +1 na troca, em rua +1 no vasilhame novo) acontece na mesma transação, que é revertida integralmente se qualquer passo falhar (RNF-005). Cancelamento de venda reverte o estoque (RGN-007), devolvendo a capacidade de venda. No app, a tela de venda exibe a mensagem de bloqueio inline e não confirma a venda (CT-001).

## Technical Context

**Language/Version**: Java 21 + Spring Boot 3.x (API); React 18 + TypeScript 5.x (app)
**Primary Dependencies**: Spring Data JPA, Flyway, Lombok (API); React, Vite, axios, react-router (app)
**Storage**: PostgreSQL único + Flyway forward-only; tabelas `tab_*`, sequências `seq_*`, colunas snake_case pt-BR
**Testing**: JUnit 5 + Mockito (Service/Controller); integração com @SpringBootTest incluindo teste de concorrência com threads; Vitest + React Testing Library (app)
**Target Platform**: navegador web (SPA) + REST API
**Project Type**: full-stack web: REST API + SPA
**Performance Goals**: lançamento de venda com validação de estoque respondendo em até 2 segundos (RNF-003)
**Constraints**: transação atômica estoque + caixa + registro da venda (RNF-005); lock pessimista por produto dentro da transação (RNF-008, CONVENTIONS §8); sem DELETE de venda, cancelamento com reversão e status (RNF-007, RGN-007); bloqueio por saldo de cheios apenas, nunca por vazios ou em rua (Edge Case); erro de bloqueio nunca vira "sistema indisponível" (Constituição §XI.3); estoque nunca negativo (RDN-005)
**Scale/Scope**: revenda única, 1-5 usuários simultâneos, dados de 12 meses (RNF-010)

## Constitution Check

Gates avaliados contra a CONSTITUICAO.md:

| Gate | Avaliação |
|---|---|
| §I-A Artefatos antes do código | spec.md, plan.md e task.md existem e são consistentes com a HU-013 - atendido |
| §II Vocabulário | usa carga/vasilhame, cheio, vazio, em rua, troca; sem sinônimos proibidos - atendido |
| §III Invariantes de estoque | estoque nunca negativo; movimentação, estoque e caixa na mesma transação; nada apagado - atendido |
| §V Orientado a requisitos | feature nasce da HU-013 com CTs vinculados a RF-031/RDN-005/RNF-008 - atendido |
| §VI Proibido | sem vender com estoque insuficiente (mesmo sob pressão); sem estoque fora de transação; sem DELETE de movimento - atendido |
| §VII Documentação | pt-BR, hífen normal, sem travessão - atendido |
| §X Rastreabilidade | RF-031, RDN-005, RNF-008, CT-001 a CT-003 referenciados - atendido |
| §XI Qualidade | CT provado por teste; regra de estoque com prioridade máxima; teste de concorrência obrigatório (CONVENTIONS §11) - atendido |

**GATE: PASS**

## Project Structure

### Documentation (this feature)

```text
specs/HU-013-Bloqueio-Sem-Estoque/
├── spec.md                # Especificação: objetivo, CTs, requisitos vinculados
├── plan.md                # Este arquivo (fase de plano)
├── research.md            # Fase 0: decisões técnicas e alternativas
├── data-model.md          # Fase 1: modelo de dados da feature
├── quickstart.md          # Fase 1: validação end-to-end dos CTs
├── contracts/
│   ├── api.md             # Contrato REST /api/vendas (bloqueio)
│   └── ui.md              # Contrato de UI da tela de venda
└── task.md                # Checklist de CTs com status (pendente/em andamento/concluído)
```

### Source Code (repository root)

```text
gerenciador_estoque_api/
└── src/
    ├── main/java/com/gerenciador/estoque/venda/
    │   ├── controller/VendaController.java
    │   ├── service/VendaService.java                 # @Transactional, valida saldo
    │   ├── repository/VendaRepository.java
    │   ├── repository/VendaItemRepository.java
    │   ├── repository/MovimentacaoEstoqueRepository.java
    │   ├── dto/VendaRequest.java
    │   ├── dto/VendaItemRequest.java
    │   └── dto/VendaResponse.java
    ├── main/java/com/gerenciador/estoque/estoque/
    │   ├── repository/EstoqueRepository.java         # lock PESSIMISTIC_WRITE
    │   └── repository/ClienteEmRuaRepository.java
    └── test/java/com/gerenciador/estoque/venda/
        ├── VendaServiceTest.java
        ├── VendaControllerTest.java
        ├── VendaBloqueioIntegrationTest.java
        └── VendaConcorrenciaIntegrationTest.java     # RNF-008

gerenciador_estoque_app/
└── src/
    ├── features/venda/
    │   ├── LancamentoVendaPage.tsx
    │   ├── components/ (VendaItemForm, ErroEstoqueInvalido)
    │   └── types.ts
    ├── api/
    │   └── vendasApi.ts                              # cliente HTTP tipado
    └── hooks/
        └── useLancamentoVenda.ts
```

**Structure Decision**: monorepo com 5 repositórios irmãos: `gerenciador_estoque_api`, `gerenciador_estoque_app`, `gerenciador_estoque_infra`, `gerenciador_estoque_audi` (este diretório) e `gerenciador_estoque_jar` (Constituição §I). A feature é implementada na API no módulo `com.gerenciador.estoque.venda` (CONVENTIONS §4), consumindo o saldo de `tab_estoque` com lock por produto no módulo `estoque`, e no app em `src/features/venda/` (CONVENTIONS §7). A validação é única e compartilhada por todos os fluxos (balcão, entrega, troca, vasilhame novo - CT-003).

## Complexity Tracking

Nenhuma violação dos gates da Constituição detectada; nada a justificar.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|---|---|---|
| - | - | - |
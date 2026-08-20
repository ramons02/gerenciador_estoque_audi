# Implementation Plan: Lançamento Rápido de Venda (Balcão/Entrega)

**Branch**: `HU-007-Lancamento-Venda` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-007-Lancamento-Venda/spec.md`

## Summary

A feature lança a venda de balcão/entrega com produto, quantidade, tipo (Balcão/Entrega) e forma de pagamento, calculando o total no Service (quantidade × preço de venda, com acréscimo do cartão por unidade quando aplicável - somente produto de carga Gás), exibindo apenas as formas de pagamento habilitadas nas Configurações e bloqueando a venda sem estoque de cheios suficiente. Abordagem técnica: endpoint `POST /api/vendas` no módulo `business.venda` com transação única (validação de saldo sob lock pessimista por produto + persistência da venda + movimentação de estoque), leitura de `tab_configuracao` (formas habilitadas e acréscimo) e tela `VendasPage` no app para lançamento rápido (RNF-001, RNF-003).

## Technical Context

**Language/Version**: Backend Java 21 + Spring Boot 3.x (Maven); Frontend React 18 + TypeScript 5.x (Vite)
**Primary Dependencies**: Spring Web (REST), Spring Data JPA, Flyway, Lombok (API); React, react-router, axios (app)
**Storage**: PostgreSQL único; migrations Flyway `V<N>__descricao_snake_case.sql` forward-only (CONVENTIONS §8)
**Testing**: JUnit 5 + Mockito (Service/Controller); `@SpringBootTest` para integração; Vitest + React Testing Library (app)
**Target Platform**: Navegador web (desktop e tablet), números grandes nos campos de venda (RNF-002)
**Project Type**: Full-stack web: REST API (Spring Boot) + SPA (React/Vite)
**Performance Goals**: Lançamento de venda respondendo em até 2 segundos, mesmo com histórico do dia aberto (RNF-003); mínimo de passos na portaria (RNF-001)
**Constraints**: Transação atômica estoque + caixa (RNF-005); lock pessimista por produto, vendas simultâneas sem estoque negativo (RNF-008); forma Fiado não existe (RGN-002); acréscimo do cartão aplicado no Service somente para produtos de carga Gás (RF-021-A), produtos de carga Água com preço normal em qualquer forma de pagamento; sem DELETE de venda, cancelamento com reversão (RNF-007); erros estruturados em pt-BR (CONVENTIONS §6); retenção de 12 meses (RNF-010)
**Scale/Scope**: Revenda única, 1 a 5 usuários simultâneos (RNF-010); vendas de balcão de poucos itens por lançamento, alto volume diário na portaria

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate | Verificação | Resultado |
|---|---|---|
| §I-A | spec.md, plan.md e tasks.md existem e são consistentes com a HU-007 antes de qualquer código | OK |
| §II | Vocabulário canônico: venda, balcão, entrega, cheio, vazio, pátio, carga, vasilhame; sem sinônimos proibidos | OK |
| §III | Invariantes: estoque nunca negativo; movimentação altera estoque e caixa na mesma transação; venda nunca apagada, apenas cancelada | OK |
| §V | CT fechado exige prova (teste ou evidência registrada); plano precede o código | OK |
| §VI | Sem DELETE de venda; sem estoque fora de transação atômica; sem vender sem validar saldo; sem regra fora de requisitos; sem referência a IA | OK |
| §VII | Documentação em pt-BR; hífen normal, proibido travessão (em-dash) | OK |
| §X | Rastreabilidade RF-020/RF-021/RF-021-A/RF-031/RF-052 e HU-007 nos commits e na doc do módulo | OK |
| §XI | CTs provados; regras de estoque e caixa com prioridade máxima de cobertura; validação não colapsa em erro de sistema | OK |

**GATE: PASS** - a feature não viola nenhum gate da Constituição.

## Project Structure

### Documentation (this feature)

```text
specs/HU-007-Lancamento-Venda/
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
│   │       ├── venda/                        # Módulo HU-007 (base das HU-008/009/010)
│   │       │   ├── VendaController.java      # GET/POST /api/vendas, PUT /api/vendas/{id}/cancelar
│   │       │   ├── VendaService.java         # @Transactional: valida saldo, calcula total e acréscimo, persiste
│   │       │   ├── VendaRepository.java
│   │       │   ├── Venda.java
│   │       │   ├── VendaItem.java            # Itens da venda (nova)
│   │       │   └── dto/
│   │       │       ├── VendaRequest.java
│   │       │       └── VendaResponse.java
│   │       ├── configuracao/                 # ConfiguracaoService.java (formas habilitadas, acréscimo do cartão)
│   │       ├── produto/                      # ProdutoRepository.java (@Lock PESSIMISTIC_WRITE ao consultar saldo)
│   │       └── estoque/                      # EstoqueController.java (GET /api/estoque, saldos)
│   ├── src/main/resources/db/migration/
│   │   ├── V1__... a V9__...                 # Schema existente (tab_venda, tab_produto, tab_configuracao)
│   │   └── V10__criar_tab_movimentacao_estoque.sql   # Nova (rastro de movimentação, HU-007)
│   └── src/test/java/com/gerenciador/estoque/business/venda/   # VendaServiceTest, VendaControllerTest
├── gerenciador_estoque_app/                  # Frontend: React 18 + TypeScript + Vite
│   └── src/
│       ├── api/
│       │   ├── client.ts                     # Cliente HTTP tipado (axios)
│       │   ├── vendas.ts                     # Endpoints de venda (HU-007)
│       │   ├── configuracoes.ts              # Formas habilitadas e acréscimo
│       │   ├── produtos.ts                   # Produtos para o seletor
│       │   └── estoque.ts                    # Saldos (validação de bloqueio)
│       └── features/
│           └── vendas/
│               ├── VendasPage.tsx            # Lançamento rápido + histórico do dia
│               └── components/
│                   └── LancamentoVenda.tsx   # Formulário de lançamento (HU-007/008/009/010)
├── gerenciador_estoque_infra/                # Infraestrutura, PostgreSQL, deploy
├── gerenciador_estoque_audi/                 # Documentação e auditoria (este plano)
└── gerenciador_estoque_jar/                  # JAR executável (privado)
```

**Structure Decision**: Monorepo em 5 repositórios irmãos (api, app, infra, audi, jar) conforme §I da Constituição. Na API, a feature vive no módulo `com.gerenciador.estoque.business.venda` com camadas Controller (REST), Service (`@Transactional`), Repository (Spring Data JPA) e dto/ (Request/Response), reusando `core` e os módulos `configuracao` (formas e acréscimo), `produto` (saldo sob lock) e `estoque` (saldos). No app, `src/features/vendas/` com cliente HTTP tipado em `src/api/vendas.ts`, nunca fetch solto (CONVENTIONS §7). As HU-008 (entrega), HU-009 (troca) e HU-010 (vasilhame novo) são variações do mesmo `POST /api/vendas` e evoluem este módulo sem duplicar estrutura.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| Nenhuma violação | - | - |
# Implementation Plan: Venda de Vasilhame Novo (Casco + Carga)

**Branch**: `HU-010-Vasilhame-Novo` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-010-Vasilhame-Novo/spec.md`

## Summary

A feature vende vasilhame novo (casco + carga, sem devolução de vazio): o preço é composto pelo preço do casco mais o preço da carga, o cliente é obrigatório para o controle de comodato, e ao confirmar o sistema baixa 1 cheio do estoque e registra 1 vasilhame "em rua" para o cliente por unidade. É uma variação do `POST /api/vendas` da HU-007: o item usa `tipoItem = CASCO_NOVO`, o `VendaService` soma `preco_casco` (tab_vasilhame) ao preço da carga, exige `idCliente` e executa a movimentação dupla SAIDA_CHEIO + EM_RUA no mesmo commit, atualizando o controle por cliente em `tab_cliente_vasilhame`. O plano cobre apenas os CTs da HU-010, referenciando a estrutura comum de venda.

## Technical Context

**Language/Version**: Backend Java 21 + Spring Boot 3.x (Maven); Frontend React 18 + TypeScript 5.x (Vite)
**Primary Dependencies**: Spring Web (REST), Spring Data JPA, Flyway, Lombok (API); React, react-router, axios (app)
**Storage**: PostgreSQL único; migrations Flyway `V<N>__descricao_snake_case.sql` forward-only (CONVENTIONS §8); preço do casco em `tab_vasilhame.preco_casco`, nunca hardcoded (RGN-010)
**Testing**: JUnit 5 + Mockito (Service/Controller); `@SpringBootTest` para integração; Vitest + React Testing Library (app)
**Target Platform**: Navegador web (desktop e tablet) consumindo a REST API (RNF-011)
**Project Type**: Full-stack web: REST API (Spring Boot) + SPA (React/Vite)
**Performance Goals**: Lançamento de venda (incluindo vasilhame novo) < 2s (RNF-003); mínimo de passos na portaria (RNF-001)
**Constraints**: Cliente obrigatório quando não há devolução de vazio (RF-004, RF-028, CT-004); preço = casco + carga calculado no Service (RGN-010); movimentação atômica baixar cheio + registrar em rua (RNF-005); lock pessimista por produto (RNF-008); sem DELETE de venda (RNF-007); erros estruturados em pt-BR (CONVENTIONS §6); retenção de 12 meses (RNF-010)
**Scale/Scope**: Revenda única, 1 a 5 usuários simultâneos (RNF-010); vendas de vasilhame novo são minoria do volume diário, mas geram o controle de comodato permanente por cliente

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate | Verificação | Resultado |
|---|---|---|
| §I-A | spec.md, plan.md e task.md existem e são consistentes com a HU-010 antes de qualquer código | OK |
| §II | Vocabulário canônico: vasilhame novo, casco, carga, cheio, em rua, comodato; sem sinônimos proibidos | OK |
| §III | Invariantes: todo vasilhame em exatamente um estado (em rua); estoque nunca negativo; movimentação altera estoque e caixa na mesma transação; venda nunca apagada | OK |
| §V | CT fechado exige prova (teste ou evidência registrada); plano precede o código | OK |
| §VI | Sem DELETE de venda; sem estoque fora de transação atômica; sem vender sem validar saldo; sem referência a IA | OK |
| §VII | Documentação em pt-BR; hífen normal, proibido travessão (em-dash) | OK |
| §X | Rastreabilidade RF-024/RF-026/RF-028/RDN-008/RGN-010 e HU-010 nos commits e na doc do módulo | OK |
| §XI | CTs provados; regras de estoque e comodato com prioridade máxima de cobertura; validação não colapsa em erro de sistema | OK |

**GATE: PASS** - a feature não viola nenhum gate da Constituição.

## Project Structure

### Documentation (this feature)

```text
specs/HU-010-Vasilhame-Novo/
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
│   │       ├── venda/                        # Módulo venda (base HU-007; HU-010 estende)
│   │       │   ├── VendaService.java         # CASCO_NOVO: preço casco + carga, cliente obrigatório, EM_RUA
│   │       │   ├── VendaItem.java            # tipo_item CASCO_NOVO, preco_casco gravado
│   │       │   └── dto/VendaRequest.java     # idCliente obrigatório; tipoItem CASCO_NOVO
│   │       ├── cliente/                      # ClienteRepository.java (validação e controle de comodato)
│   │       ├── vasilhame/                    # VasilhameRepository.java (preco_casco para o cálculo)
│   │       └── produto/                      # ProdutoRepository.java (@Lock PESSIMISTIC_WRITE)
│   ├── src/main/resources/db/migration/
│   │   ├── V1__... a V9__...                 # Schema existente (tab_cliente_vasilhame na V3, preco_casco na V3)
│   │   └── V10__criar_tab_movimentacao_estoque.sql   # Nova (tipo EM_RUA)
│   └── src/test/java/com/gerenciador/estoque/business/venda/
│       ├── VendaServiceTest.java             # CASCO_NOVO: preço, cliente obrigatório, em rua
│       └── VendaVasilhameNovoIntegracaoTest.java   # @SpringBootTest: atomicidade cheio + em rua
├── gerenciador_estoque_app/                  # Frontend: React 18 + TypeScript + Vite
│   └── src/
│       ├── api/
│       │   ├── vendas.ts                     # POST /api/vendas com tipoItem CASCO_NOVO
│       │   ├── clientes.ts                   # Busca/cadastro rápido de cliente (edge case)
│       │   └── produtos.ts                   # Produtos com preço do casco
│       └── features/
│           ├── vendas/
│           │   └── components/LancamentoVenda.tsx   # Opção "Vasilhame novo" + seletor de cliente (CT-001/CT-004)
│           └── clientes/
│               └── ClientesPage.tsx          # Cadastro de cliente antes de concluir (edge case)
├── gerenciador_estoque_infra/                # Infraestrutura, PostgreSQL, deploy
├── gerenciador_estoque_audi/                 # Documentação e auditoria (este plano)
└── gerenciador_estoque_jar/                  # JAR executável (privado)
```

**Structure Decision**: Monorepo em 5 repositórios irmãos (api, app, infra, audi, jar) conforme §I da Constituição. HU-010 não cria módulo novo: estende `com.gerenciador.estoque.business.venda` (lógica do vasilhame novo no `VendaService`), usando `business.cliente` (validação e controle em rua via `tab_cliente_vasilhame`) e `business.vasilhame` (preço do casco). No app, a opção "Vasilhame novo" entra no `LancamentoVenda.tsx` existente com seletor de cliente, e o cadastro rápido de cliente reusa `ClientesPage` (HU-004). A movimentação EM_RUA é gravada com `id_referencia` da venda, base da baixa na devolução (RF-028, HU-011).

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| Nenhuma violação | - | - |
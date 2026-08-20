# Implementation Plan: Troca de Vasilhame (Venda Normal)

**Branch**: `HU-009-Troca-Vasilhame` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-009-Troca-Vasilhame/spec.md`

## Summary

A feature registra a venda com troca de vasilhame (cliente entrega 1 vazio e leva 1 cheio): ao confirmar, o sistema baixa 1 cheio do estoque e adiciona 1 vazio ao pátio por unidade vendida, na mesma transação atômica, sem alterar o total da venda, refletindo o pátio imediatamente. É uma variação do `POST /api/vendas` da HU-007: o request marca a venda como troca e o `VendaService` executa a movimentação dupla (SAIDA_CHEIO + ENTRADA_VAZIO) sob lock pessimista por produto, com persistência de `tab_venda` + `tab_venda_item` + `tab_movimentacao_estoque` no mesmo commit. O plano cobre apenas os CTs da HU-009, referenciando a estrutura comum de venda.

## Technical Context

**Language/Version**: Backend Java 21 + Spring Boot 3.x (Maven); Frontend React 18 + TypeScript 5.x (Vite)
**Primary Dependencies**: Spring Web (REST), Spring Data JPA, Flyway, Lombok (API); React, react-router, axios (app)
**Storage**: PostgreSQL único; migrations Flyway `V<N>__descricao_snake_case.sql` forward-only (CONVENTIONS §8)
**Testing**: JUnit 5 + Mockito (Service/Controller); `@SpringBootTest` para integração; Vitest + React Testing Library (app)
**Target Platform**: Navegador web (desktop e tablet) consumindo a REST API (RNF-011)
**Project Type**: Full-stack web: REST API (Spring Boot) + SPA (React/Vite)
**Performance Goals**: Lançamento de venda com troca < 2s (RNF-003); mínimo de passos na portaria (RNF-001)
**Constraints**: Movimentação dupla atômica: baixar cheio + adicionar vazio no mesmo commit (RDN-004, RNF-005); lock pessimista por produto para cheios e vazios (RNF-008); troca sem custo, total inalterado (RF-023); saldo de vazios nunca negativo (RDN-003, RDN-005); sem DELETE de venda (RNF-007); erros estruturados em pt-BR (CONVENTIONS §6); retenção de 12 meses (RNF-010)
**Scale/Scope**: Revenda única, 1 a 5 usuários simultâneos (RNF-010); troca é o modelo central de venda da revenda (casco retornável), alto volume diário

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate | Verificação | Resultado |
|---|---|---|
| §I-A | spec.md, plan.md e task.md existem e são consistentes com a HU-009 antes de qualquer código | OK |
| §II | Vocabulário canônico: troca, cheio, vazio, pátio, vasilhame, em rua; sem sinônimos proibidos ("permuta", "devolve e leva") | OK |
| §III | Invariantes: todo vasilhame em exatamente um estado; estoque nunca negativo; movimentação altera estoque e caixa na mesma transação; venda nunca apagada | OK |
| §V | CT fechado exige prova (teste ou evidência registrada); plano precede o código | OK |
| §VI | Sem DELETE de venda; sem estoque fora de transação atômica; sem vender sem validar saldo; sem referência a IA | OK |
| §VII | Documentação em pt-BR; hífen normal, proibido travessão (em-dash) | OK |
| §X | Rastreabilidade RF-023/RF-025/RDN-004 e HU-009 nos commits e na doc do módulo | OK |
| §XI | CTs provados; regras de estoque com prioridade máxima (atomicidade da troca); validação não colapsa em erro de sistema | OK |

**GATE: PASS** - a feature não viola nenhum gate da Constituição.

## Project Structure

### Documentation (this feature)

```text
specs/HU-009-Troca-Vasilhame/
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
│   │       ├── venda/                        # Módulo venda (base HU-007; HU-009 estende)
│   │       │   ├── VendaService.java         # Troca: movimentação dupla atômica (RDN-004)
│   │       │   ├── Venda.java / VendaItem.java   # Flag/indicador de troca no item (tipoItem CHEIO + troca)
│   │       │   └── dto/VendaRequest.java     # Campo "troca" (true/false)
│   │       ├── produto/                      # ProdutoRepository.java (@Lock PESSIMISTIC_WRITE: cheios e vazios)
│   │       └── estoque/                      # EstoqueController.java (GET /api/estoque, saldo de vazios)
│   ├── src/main/resources/db/migration/
│   │   ├── V1__... a V9__...                 # Schema existente (tab_produto com estoque_cheios/estoque_vazios)
│   │   └── V10__criar_tab_movimentacao_estoque.sql   # Nova (ENTRADA_VAZIO da troca)
│   └── src/test/java/com/gerenciador/estoque/business/venda/
│       ├── VendaServiceTest.java             # Troca: -1 cheio +1 vazio por unidade; atomicidade
│       └── VendaTrocaIntegracaoTest.java     # @SpringBootTest: estado parcial inexistente
├── gerenciador_estoque_app/                  # Frontend: React 18 + TypeScript + Vite
│   └── src/
│       ├── api/
│       │   ├── vendas.ts                     # POST /api/vendas com troca: true
│       │   └── estoque.ts                    # Saldo de vazios exibido/atualizado
│       └── features/
│           └── vendas/
│               └── components/LancamentoVenda.tsx   # Opção "Troca de vasilhame" (CT-001/CT-003/CT-004)
├── gerenciador_estoque_infra/                # Infraestrutura, PostgreSQL, deploy
├── gerenciador_estoque_audi/                 # Documentação e auditoria (este plano)
└── gerenciador_estoque_jar/                  # JAR executável (privado)
```

**Structure Decision**: Monorepo em 5 repositórios irmãos (api, app, infra, audi, jar) conforme §I da Constituição. HU-009 não cria módulo novo: estende `com.gerenciador.estoque.business.venda` (lógica da troca no `VendaService`, dentro da mesma transação da HU-007) e usa `business.produto` para o lock de cheios e vazios. No app, a opção "Troca de vasilhame" entra no `LancamentoVenda.tsx` existente. A movimentação dupla é gravada em `tab_movimentacao_estoque` com o mesmo `id_referencia` (venda), base da conferência do pátio (RDN-003) e do cancelamento com reversão (RGN-007, HU-020).

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| Nenhuma violação | - | - |
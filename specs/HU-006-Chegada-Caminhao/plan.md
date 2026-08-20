# Implementation Plan: Registro de Chegada de Caminhão (Entrada)

**Branch**: `HU-006-Chegada-Caminhao` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-006-Chegada-Caminhao/spec.md`

## Summary

A feature registra a chegada de caminhão (carregamento) com fornecedor, produto, data, cheios recebidos, vazios devolvidos e custo total, calculando automaticamente o valor unitário (custo total ÷ cheios). Ao confirmar, em transação única, o estoque de cheios aumenta em N, o saldo de vazios do pátio diminui em N, o custo médio do produto é recalculado e o registro fica disponível ao Relatório de Carregamentos. A abordagem técnica: endpoint `POST /api/carregamentos` no módulo `business.carregamento` da API, com validação da devolução de vazios contra o saldo do pátio sob lock pessimista por produto, persistência de `tab_carregamento` + `tab_carregamento_item` + `tab_movimentacao_estoque` na mesma transação `@Transactional`, e tela `CarregamentosPage` no app consumindo o cliente HTTP tipado.

## Technical Context

**Language/Version**: Backend Java 21 + Spring Boot 3.x (Maven); Frontend React 18 + TypeScript 5.x (Vite)
**Primary Dependencies**: Spring Web (REST), Spring Data JPA, Flyway, Lombok (API); React, react-router, axios (app)
**Storage**: PostgreSQL único; migrations Flyway `V<N>__descricao_snake_case.sql` forward-only (CONVENTIONS §8)
**Testing**: JUnit 5 + Mockito (Service/Controller); `@SpringBootTest` para integração; Vitest + React Testing Library (app)
**Target Platform**: Navegador web (desktop e tablet) consumindo a REST API (RNF-011)
**Project Type**: Full-stack web: REST API (Spring Boot) + SPA (React/Vite)
**Performance Goals**: Confirmação de carregamento com resposta imediata; lançamento de venda posterior < 2s (RNF-003)
**Constraints**: Transação atômica estoque + caixa (RNF-005); lock pessimista por produto (RNF-008); validação de devolução de vazios contra o pátio (RDN-003); sem DELETE de movimento, cancelamento com reversão (RNF-007); mensagens de erro estruturadas em pt-BR (CONVENTIONS §6); retenção de histórico de 12 meses (RNF-010)
**Scale/Scope**: Revenda única, 1 a 5 usuários simultâneos, histórico de 12 meses (RNF-010); carregamentos eventuais (alguns por semana), um ou poucos itens por chegada

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate | Verificação | Resultado |
|---|---|---|
| §I-A | spec.md, plan.md e tasks.md existem e são consistentes com a HU-006 antes de qualquer código | OK |
| §II | Vocabulário canônico: carregamento, cheio, vazio, pátio, carga, vasilhame; proibidos "chegada/compra" como termo técnico e sinônimos de carga/vasilhame | OK |
| §III | Invariantes de estoque: todo vasilhame em exatamente um estado; estoque nunca negativo; movimentação altera estoque na mesma transação do registro; entrada nunca apagada, apenas estornada | OK |
| §V | CT fechado exige prova (teste ou evidência registrada); plano precede o código | OK |
| §VI | Sem DELETE de movimento; sem atualização de estoque fora de transação atômica; sem regra fora de requisitos; sem referência a ferramenta de IA | OK |
| §VII | Documentação em pt-BR; hífen normal, proibido travessão (em-dash) | OK |
| §X | Rastreabilidade RF-010/RF-011/RF-012 e HU-006 nos commits e na documentação do módulo | OK |
| §XI | CTs provados por teste automatizado ou evidência; regras de estoque com prioridade máxima de cobertura; erro de validação não colapsa em erro de sistema | OK |

**GATE: PASS** - a feature não viola nenhum gate da Constituição.

## Project Structure

### Documentation (this feature)

```text
specs/HU-006-Chegada-Caminhao/
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
│   │       ├── carregamento/                 # Módulo HU-006
│   │       │   ├── CarregamentoController.java   # GET/POST /api/carregamentos
│   │       │   ├── CarregamentoService.java      # @Transactional, validações e custo médio
│   │       │   ├── CarregamentoRepository.java
│   │       │   ├── Carregamento.java
│   │       │   ├── CarregamentoItem.java
│   │       │   └── dto/
│   │       │       ├── CarregamentoRequest.java
│   │       │       └── CarregamentoResponse.java
│   │       ├── produto/                      # Produto.java, ProdutoRepository (@Lock PESSIMISTIC_WRITE), EstoqueProduto.java
│   │       ├── fornecedor/                   # Fornecedor.java, FornecedorRepository.java (leitura)
│   │       └── estoque/                      # EstoqueController.java (GET /api/estoque, saldos)
│   ├── src/main/resources/db/migration/
│   │   ├── V1__... a V9__...                 # Schema existente (tab_carga, tab_vasilhame, tab_produto, tab_fornecedor, tab_carregamento, tab_carregamento_item, tab_configuracao)
│   │   └── V10__criar_tab_movimentacao_estoque.sql   # Nova (rastro de movimentação, HU-006)
│   └── src/test/java/com/gerenciador/estoque/business/carregamento/   # CarregamentoServiceTest, CarregamentoControllerTest
├── gerenciador_estoque_app/                  # Frontend: React 18 + TypeScript + Vite
│   └── src/
│       ├── api/
│       │   ├── client.ts                     # Cliente HTTP tipado (axios)
│       │   ├── carregamentos.ts              # Endpoints de carregamento (HU-006)
│       │   ├── fornecedores.ts               # Leitura de fornecedores para o formulário
│       │   ├── produtos.ts                   # Leitura de produtos para o formulário
│       │   └── estoque.ts                    # GET saldos (validação de vazios)
│       └── features/
│           └── carregamentos/
│               ├── CarregamentosPage.tsx     # Formulário de chegada + lista de carregamentos
│               └── components/
│                   └── FormularioChegada.tsx
├── gerenciador_estoque_infra/                # Infraestrutura, PostgreSQL, deploy
├── gerenciador_estoque_audi/                 # Documentação e auditoria (este plano)
└── gerenciador_estoque_jar/                  # JAR executável (privado)
```

**Structure Decision**: Monorepo em 5 repositórios irmãos (api, app, infra, audi, jar) conforme §I da Constituição. Na API, a feature vive no módulo `com.gerenciador.estoque.business.carregamento` com camadas Controller (REST), Service (`@Transactional`), Repository (Spring Data JPA) e dto/ (Request/Response), reusando `core` e os módulos `produto`, `fornecedor` e `estoque` para saldos e validação sob lock. No app, `src/features/carregamentos/` com cliente HTTP tipado em `src/api/carregamentos.ts`, sem fetch solto (CONVENTIONS §7). Migrations Flyway forward-only em `src/main/resources/db/migration/` (CONVENTIONS §8).

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| Nenhuma violação | - | - |
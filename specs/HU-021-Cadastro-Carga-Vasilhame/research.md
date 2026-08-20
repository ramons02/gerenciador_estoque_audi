# Research: HU-021 - Cadastro de Carga e Vasilhame

**HU**: HU-021 | **Feature**: Cadastro de Carga e Vasilhame | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Requisitos vinculados**: FR-001 a FR-006 (CT-001 a CT-006), RDN-001, RF-021-A

Fase 0 - decisões de design registradas antes de qualquer código. Cada decisão referencia a convenção, constituição ou requisito que a justifica.

---

## Decisão 1: Validação de nome duplicado no Service, via hook validar do BaseService

**Decision**: `CargaService` e `VasilhameService` sobrescrevem o hook `validar(entidade)` do `BaseService`, consultando `findByNome(nome)` e lançando `BusinessException` com mensagem em pt-BR quando o nome já existe (ignorando o próprio registro na edição). Nome vazio também é rejeitado.

**Rationale**: A coluna `nome` de `tab_carga` e `tab_vasilhame` é `UNIQUE` desde a V1, mas a violação de constraint sozinha viraria erro 500 genérico (GlobalExceptionHandler, handler genérico). A validação no Service devolve 422 com mensagem clara (CONVENTIONS §6), segue o padrão de camadas do projeto (regra no Service, nunca no controller) e o mesmo padrão já usado por `ClienteService` e `VendaService`.

**Alternatives considered**: Capturar `DataIntegrityViolationException` no controller - rejeitado por concentrar regra fora do Service e depender da mensagem do driver; validar só no frontend - rejeitado por deixar a API aceitar duplicado fora da tela.

---

## Decisão 2: Duplicidade por nome exato, sem normalização de maiúsculas

**Decision**: A consulta `findByNome(nome)` usa igualdade exata (case-sensitive), e o nome é apenas aparado (`trim`) antes da comparação. "Gas" e "gas" são tratados como nomes diferentes.

**Rationale**: O spec HU-021 (Edge Cases) define que o sistema trata pelo nome exato informado, e a constraint UNIQUE do banco é case-sensitive. Normalizar maiúsculas criaria divergência entre a regra de aplicação e o banco (CONVENTIONS §8).

**Alternatives considered**: `findByNomeIgnoreCase` - rejeitado por divergir da constraint UNIQUE do banco e do edge case documentado no spec.

---

## Decisão 3: Criação de carga/vasilhame inline no formulário de produto

**Decision**: No app, os seletores de Carga e Vasilhame do formulário de produto ganham as opções "Nova carga..." e "Novo vasilhame...". Ao escolhê-las, abre um formulário inline (input de nome e, para vasilhame, preço do casco) com botões Criar/Cancelar. Ao criar, a lista é recarregada e o item novo fica selecionado no formulário para continuar o cadastro do produto.

**Rationale**: FR-003 exige que o item criado fique selecionado e o cadastro continue sem sair da tela (CT-003). Reusa os endpoints já existentes `POST /api/cargas` e `POST /api/vasilhames` sem alteração de contrato.

**Alternatives considered**: Tela separada de manutenção de cargas/vasilhames - rejeitada por estar fora do escopo da versão (Assumptions do spec) e aumentar passos para o cadastro; digitar nome livre com criação automática - rejeitado por esconder a criação e dificultar a correção de erros de digitação.

---

## Decisão 4: Nenhuma migration nova

**Decision**: A feature não adiciona migration: `tab_carga` (nome UNIQUE) e `tab_vasilhame` (nome UNIQUE, preco_casco) existem desde a V1, e os endpoints de criação existem desde o início do projeto.

**Rationale**: Sem mudança de schema, não há necessidade de migration Flyway (CONVENTIONS §8); adicionar migration sem mudança real inflaria o histórico sem benefício.

**Alternatives considered**: Migration para constraint adicional - rejeitada por redundância com o UNIQUE já existente.

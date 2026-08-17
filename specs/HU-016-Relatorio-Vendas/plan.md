# Plano de Implementação - HU-016 — Relatório de Vendas (Diário/Mensal)

**HU de origem:** HU-016
**Status:** Em elaboração

## 1. Escopo

- 3 critérios de aceitação a implementar e provar.
- Requisitos vinculados: RF-041, RF-040, RF-044, RGN-008.

## 2. Decisões de arquitetura

- **API (Java/Spring Boot):** módulo em `com.gerenciador.estoque.business.*`, herdando
  `core` (BaseModel, BaseRepository, BaseService) para reuso (SOLID).
- **Banco (PostgreSQL):** migration Flyway `V<N>__*.sql` forward-only (CONVENTIONS §8).
- **App (React/TypeScript):** tela por HU em pasta de feature, cliente HTTP tipado.

## 3. Etapas

1. Modelagem de dados (migration Flyway).
2. Entidade + Repository + Service + Controller na API.
3. Testes de regra de negócio (caixa/estoque prioridade máxima).
4. Tela no app consumindo a API.
5. Prova dos critérios de aceitação (CT) e evidência.

## 4. Definição de pronto

- Todos os CTs provados com evidência registrada (Constituição §V.2).
- Sem `DELETE` físico em movimento; exclusão lógica via `ativo = false`.

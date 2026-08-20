# Data Model: HU-021 - Cadastro de Carga e Vasilhame

**HU**: HU-021 | **Feature**: Cadastro de Carga e Vasilhame | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Requisitos vinculados**: FR-001 a FR-006 (CT-001 a CT-006), RDN-001

Sem mudanças de schema nesta feature (Decisão 4 do research). Entidades envolvidas, já existentes:

---

## tab_carga

| Campo | Tipo | Restrições | Observação |
|---|---|---|---|
| id | BIGSERIAL | PK | |
| nome | VARCHAR(50) | NOT NULL, UNIQUE | "Gas", "Agua" na seed (V8); novos nomes validados no Service (CT-004) |
| criado_em | TIMESTAMP | NOT NULL | PrePersist do BaseModel |
| atualizado_em | TIMESTAMP | NOT NULL | PreUpdate do BaseModel |
| ativo | BOOLEAN | NOT NULL, DEFAULT TRUE | exclusão é lógica (BaseService.excluirLogico) |

**Entidade JPA**: `Carga` (módulo `business.carga`). **Regras de validação** (CargaService.validar): nome obrigatório (CT-005); nome único por igualdade exata após trim (CT-004; Decisão 2 do research).

## tab_vasilhame

| Campo | Tipo | Restrições | Observação |
|---|---|---|---|
| id | BIGSERIAL | PK | |
| nome | VARCHAR(50) | NOT NULL, UNIQUE | "P13", "Galão 20L" na seed (V8); novos nomes validados no Service (CT-004) |
| preco_casco | NUMERIC(12,2) | NOT NULL, DEFAULT 0 | preço do casco; zero permitido na criação (Assumptions do spec) |
| criado_em | TIMESTAMP | NOT NULL | PrePersist do BaseModel |
| atualizado_em | TIMESTAMP | NOT NULL | PreUpdate do BaseModel |
| ativo | BOOLEAN | NOT NULL, DEFAULT TRUE | exclusão é lógica (BaseService.excluirLogico) |

**Entidade JPA**: `Vasilhame` (módulo `business.vasilhame`). **Regras de validação** (VasilhameService.validar): nome obrigatório (CT-005); nome único por igualdade exata após trim (CT-004).

## Relacionamentos (existentes, sem alteração)

- `tab_produto.carga_id -> tab_carga.id` (N:1)
- `tab_produto.vasilhame_id -> tab_vasilhame.id` (N:1)
- `tab_produto`: constraint `uk_produto_carga_vasilhame UNIQUE (carga_id, vasilhame_id)` (RDN-001) - combinação única garantida no banco; carga/vasilhame novos viabilizam combinações novas (CT-006).
- Exclusão de carga/vasilhame com produto vinculado: `excluirLogico` apenas desativa (`ativo = FALSE`), preservando o histórico (RNF-007); listagens usam `findByAtivoTrue`.

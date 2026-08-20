# Modelo de Dados: Cadastro de Produtos (Carga + Vasilhame)

**HU**: HU-001 - Cadastro de Produtos (Carga + Vasilhame)
**Fase**: 1 (modelo de dados)
**Branch**: `HU-001-Cadastro-Produtos`

Tabelas criadas pela migration `V1__criar_tabelas_base.sql` (CONVENTIONS §8). Todas as
tabelas herdam as colunas de auditoria do `BaseModel`: `criado_em TIMESTAMP NOT NULL`,
`atualizado_em TIMESTAMP NOT NULL` e `ativo BOOLEAN NOT NULL DEFAULT TRUE`. Os `id` usam
`BIGSERIAL`, que cria a sequência implícita da tabela (convenção `seq_<entidade>`).

---

## 1. tab_carga (Carga/Conteúdo)

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGSERIAL | sim | PK | Sequência implícita |
| nome | VARCHAR(50) | sim | - | Único (`UNIQUE`); ex.: Gas, Agua |
| criado_em | TIMESTAMP | sim | - | Auditoria |
| atualizado_em | TIMESTAMP | sim | - | Auditoria |
| ativo | BOOLEAN | sim | - | Default TRUE |

**Regras derivadas dos requisitos:**
- A carga é a referência do Conteúdo vendido (RDN-001); a capacidade fixa é definida pela
  distribuidora e não é editável na revenda (RDN-007).
- Dados iniciais de fábrica: `Gas` e `Agua` (migrations V1 e V8, idempotente).

## 2. tab_vasilhame (Vasilhame/Casco)

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGSERIAL | sim | PK | Sequência implícita |
| nome | VARCHAR(50) | sim | - | Único (`UNIQUE`); ex.: P13, P45, Galão 20L |
| preco_casco | NUMERIC(12,2) | sim | - | Default 0; usado na venda de vasilhame novo (RGN-010) |
| criado_em | TIMESTAMP | sim | - | Auditoria |
| atualizado_em | TIMESTAMP | sim | - | Auditoria |
| ativo | BOOLEAN | sim | - | Default TRUE |

**Regras derivadas dos requisitos:**
- O vasilhame é o casco retornável; "P45" foi descontinuado via `ativo = FALSE` (migration V5).
- O preço do casco é configurado à parte do produto (RGN-010), alimentando a venda de
  vasilhame novo (RF-024).

## 3. tab_produto (item vendido = carga + vasilhame)

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGSERIAL | sim | PK | Sequência implícita |
| carga_id | BIGINT | sim | FK tab_carga(id) | Composição do item (RDN-001) |
| vasilhame_id | BIGINT | sim | FK tab_vasilhame(id) | Composição do item (RDN-001) |
| preco_custo | NUMERIC(12,2) | sim | - | Tratado pela HU-002 (RF-002, RGN-005) |
| preco_venda | NUMERIC(12,2) | sim | - | Tratado pela HU-002 (RF-002, RGN-005) |
| limite_minimo | INTEGER | sim | - | Default 0; tratado pela HU-003 (RF-003) |
| estoque_cheios | INTEGER | sim | - | Default 0; saldo de cheios (HU-003, HU-006, HU-007) |
| estoque_vazios | INTEGER | sim | - | Default 0; saldo de vazios no pátio |
| criado_em | TIMESTAMP | sim | - | Auditoria |
| atualizado_em | TIMESTAMP | sim | - | Auditoria |
| ativo | BOOLEAN | sim | - | Default TRUE |

**Constraints:**
- `uk_produto_carga_vasilhame UNIQUE (carga_id, vasilhame_id)`: garante que a combinação
  carga + vasilhame seja única (CT-003), também sob concorrência (RNF-008).

**Regras de validação (RF-001, RDN-001, CT-001, CT-003):**
- Carga e vasilhame são obrigatórios; produto sem qualquer um dos dois não é salvo.
- A combinação carga + vasilhame é única; tentativa de duplicar é bloqueada com mensagem
  pt-BR (erro 422).
- Um mesmo vasilhame pode existir com cargas diferentes (ex.: "Gás P13" e "Água P13") e uma
  mesma carga com vasilhames diferentes (ex.: "Gás P13" e "Gás P45"): a unicidade é da
  combinação, não de cada parte.
- O nome exibido é derivado: `carga.nome + " " + vasilhame.nome` (CT-002), não persistido.

## 4. Transições de estado

**Produto (tab_produto):**

| De | Para | Gatilho | Regra |
|---|---|---|---|
| ativo = true | ativo = false | DELETE /api/produtos/{id} | Só permitido quando não houver movimentações vinculadas (CT-004) |
| ativo = true | ativo = true (editado) | PUT /api/produtos/{id} | Mantém unicidade da nova combinação (CT-004) |
| ativo = false | ativo = true | - | Não existe reativação via API; novo cadastro cria registro novo |

**Bloqueio de exclusão:** a exclusão é bloqueada quando existirem registros em `tab_venda`
(vendas vinculadas ao produto) ou em `tab_carregamento_item` (carregamentos vinculados).
Essas tabelas são criadas pelas features HU-006 (carregamento) e HU-007 (venda); até que
existam, a consulta de movimentações retorna vazio e a exclusão fica liberada (ressalva
registrada no task.md).

## 5. Relacionamento entre entidades

```text
tab_carga (1) --- (N) tab_produto (N) --- (1) tab_vasilhame
```

- `tab_produto.carga_id` referencia `tab_carga.id`.
- `tab_produto.vasilhame_id` referencia `tab_vasilhame.id`.
- `tab_venda.produto_id` e `tab_carregamento_item.produto_id` (criadas por HU-006 e HU-007)
  referenciam `tab_produto.id` e são a origem do bloqueio de exclusão (CT-004, RNF-007).

**Nota de consistência:** as entidades seguem a nomenclatura canônica do projeto
(`tab_<entidade>`, colunas snake_case pt-BR) conforme CONVENTIONS §4; o nome canônico do item
vendido é "Produto" (composição carga + vasilhame), e os termos Carga e Vasilhame seguem o
vocabulário da Constituição §II.
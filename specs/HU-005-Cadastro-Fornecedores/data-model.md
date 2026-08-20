# Modelo de Dados: Cadastro de Fornecedores (Distribuidoras)

**HU**: HU-005 - Cadastro de Fornecedores (Distribuidoras)
**Fase**: 1 (modelo de dados)
**Branch**: `HU-005-Cadastro-Fornecedores`

Tabela `tab_fornecedor` criada pela migration `V2__criar_tab_fornecedor.sql`. A feature não
cria outras tabelas: o vínculo com o carregamento usa a coluna `fornecedor_id` de
`tab_carregamento` (migration `V3`, implementada na HU-006). Herda as colunas de auditoria
do `BaseModel` (`criado_em`, `atualizado_em`, `ativo`).

---

## 1. tab_fornecedor

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGSERIAL | sim | PK | Sequência implícita |
| nome | VARCHAR(100) | sim | - | Único (`UNIQUE`); ex.: Ultragaz, Nacional (CT-001) |
| contato | VARCHAR(100) | não | - | Contato da distribuidora (CT-001) |
| criado_em | TIMESTAMP | sim | - | Auditoria |
| atualizado_em | TIMESTAMP | sim | - | Auditoria |
| ativo | BOOLEAN | sim | - | Default TRUE |

**Regras de validação (RF-005, CT-001):**
- `nome` é obrigatório; fornecedor sem nome não é salvo ("Informe o nome do fornecedor.",
  422).
- `nome` é único entre ativos; duplicidade é bloqueada com "Já existe um fornecedor com este
  nome." (422), evitando duplicados na seleção do carregamento (edge case da spec).
- `contato` é opcional e não interfere na seleção do carregamento.

## 2. tab_carregamento (coluna de vínculo usada pela feature)

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGSERIAL | sim | PK | Sequência implícita |
| fornecedor_id | BIGINT | sim | FK tab_fornecedor(id) | Origem da carga (RF-010) |
| criado_em | TIMESTAMP | sim | - | Auditoria |
| atualizado_em | TIMESTAMP | sim | - | Auditoria |
| ativo | BOOLEAN | sim | - | Default TRUE |

**Regra derivada (CT-002, RF-010):**
- Todo carregamento possui um fornecedor cadastrado vinculado; a FK `fornecedor_id` é
  obrigatória (`NOT NULL`).
- Fornecedor não cadastrado não pode ser usado em um registro de carregamento (edge case da
  spec).

## 3. Transições de estado

**Fornecedor (tab_fornecedor):**

| De | Para | Gatilho | Regra |
|---|---|---|---|
| ativo = true | ativo = false | DELETE /api/fornecedores/{id} | Exclusão lógica; carregamentos históricos preservados (RNF-007) |
| ativo = true | ativo = true (editado) | PUT /api/fornecedores/{id} | Edição livre; mantém unicidade do nome |

**Consequência para carregamentos:** carregamentos existentes mantêm o vínculo e o relatório
(RF-042) continua exibindo o nome do fornecedor; novos carregamentos não listam fornecedores
inativos.

## 4. Relacionamento entre entidades

```text
tab_fornecedor (1) --- (N) tab_carregamento
```

- `tab_carregamento.fornecedor_id` referencia `tab_fornecedor.id`.
- A seleção de fornecedor no registro de carregamento (HU-006) usa a listagem de ativos de
  `tab_fornecedor`.

**Nota de consistência:** o relatório de carregamentos (RF-042, HU-017) usa o nome do
fornecedor via este vínculo; manter o registro ativo ou inativo não altera o histórico.
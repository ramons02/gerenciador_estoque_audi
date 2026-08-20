# Modelo de Dados: Cadastro de Preços (Custo e Venda)

**HU**: HU-002 - Cadastro de Preços (Custo e Venda)
**Fase**: 1 (modelo de dados)
**Branch**: `HU-002-Cadastro-Precos`

A feature não introduz tabela nova: os preços são colunas de `tab_produto` (migration
`V1__criar_tabelas_base.sql`), e o congelamento do preço da venda usa a coluna existente em
`tab_venda` (migration `V3`, criada no fluxo de venda HU-007).

---

## 1. tab_produto (colunas de preço usadas pela feature)

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGSERIAL | sim | PK | Sequência implícita |
| carga_id | BIGINT | sim | FK tab_carga(id) | Composição do item (RDN-001) |
| vasilhame_id | BIGINT | sim | FK tab_vasilhame(id) | Composição do item (RDN-001) |
| preco_custo | NUMERIC(12,2) | sim | - | Valor de custo do item (RF-002) |
| preco_venda | NUMERIC(12,2) | sim | - | Valor de venda do item (RF-002) |
| limite_minimo | INTEGER | sim | - | Default 0; tratado pela HU-003 |
| estoque_cheios | INTEGER | sim | - | Saldo de cheios (HU-003, HU-006, HU-007) |
| estoque_vazios | INTEGER | sim | - | Saldo de vazios no pátio |
| criado_em | TIMESTAMP | sim | - | Auditoria |
| atualizado_em | TIMESTAMP | sim | - | Auditoria |
| ativo | BOOLEAN | sim | - | Default TRUE |

**Regras de validação (RF-002, RGN-005, CT-001, CT-002):**
- `preco_custo` e `preco_venda` são obrigatórios e maiores que zero (valores zerados,
  negativos ou não numéricos são bloqueados).
- `preco_venda >= preco_custo`: preço de venda igual ao custo é permitido; abaixo do custo é
  bloqueado sem autorização do administrador (RGN-005).
- Comparação monetária sempre com `NUMERIC`/`BigDecimal`, nunca ponto flutuante.

## 2. tab_venda (coluna de snapshot usada pela feature)

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGSERIAL | sim | PK | Sequência implícita |
| produto_id | BIGINT | sim | FK tab_produto(id) | Produto vendido |
| cliente_id | BIGINT | não | FK tab_cliente(id) | Opcional; obrigatório em vasilhame novo |
| quantidade | INTEGER | sim | - | Quantidade vendida |
| tipo | VARCHAR(20) | sim | - | BALCAO ou ENTREGA |
| forma_pagamento | VARCHAR(20) | sim | - | Dinheiro, PIX ou Cartão |
| valor_unitario | NUMERIC(12,2) | sim | - | Snapshot do preço no lançamento (CT-003) |
| total | NUMERIC(12,2) | sim | - | Total da venda |
| status | VARCHAR(20) | sim | - | CONCLUIDA, CANCELADA |
| vasilhame_novo | BOOLEAN | sim | - | Default FALSE (RF-024) |
| criado_em / atualizado_em / ativo | - | sim | - | Auditoria (RNF-007) |

**Regra derivada (CT-003):**
- `valor_unitario` é preenchido no momento do lançamento da venda com o `preco_venda`
  vigente da `tab_produto`; alterações posteriores do preço do produto não recalculam vendas
  existentes.
- Relatórios (RF-041) e fechamento de caixa (RF-043) usam `valor_unitario` e `total` da
  venda, nunca o preço atual do produto.

## 3. Transições de estado

**Preço do produto (tab_produto):**

| De | Para | Gatilho | Regra |
|---|---|---|---|
| preço A | preço B | PUT /api/produtos/{id} | A qualquer momento (CT-003); vale para lançamentos futuros |
| preço B | preço A | PUT /api/produtos/{id} | Sem restrição, desde que venda >= custo (RGN-005) |

**Vendas existentes:** não sofrem transição: o `valor_unitario` gravado é imutável
(CT-003); apenas o cancelamento de venda (RGN-007, HU-020) altera o registro, via status.

## 4. Relacionamento entre entidades

```text
tab_produto (1) --- (N) tab_venda
```

- `tab_venda.valor_unitario` guarda o preço vigente no lançamento (snapshot), desacoplado do
  `preco_venda` atual de `tab_produto`.
- A feature HU-002 apenas mantém o cadastro dos preços; o uso na venda é responsabilidade da
  HU-007.

**Nota de consistência:** a decisão de manter os preços como colunas de `tab_produto` segue
a alternativa permitida pelas DECISÕES TÉCNICAS do projeto ("colunas na tab_produto:
preco_custo, preco_venda"), sem criar `tab_produto_preco` nesta fase.
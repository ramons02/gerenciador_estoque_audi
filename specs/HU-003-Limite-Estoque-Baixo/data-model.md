# Modelo de Dados: Limite Mínimo de Estoque (Alerta)

**HU**: HU-003 - Limite Mínimo de Estoque (Alerta)
**Fase**: 1 (modelo de dados)
**Branch**: `HU-003-Limite-Estoque-Baixo`

A feature não introduz tabela nova: usa `limite_minimo` e `estoque_cheios` da `tab_produto`
(migration `V1__criar_tabelas_base.sql`). O alerta é derivado, sem armazenamento próprio.

---

## 1. tab_produto (colunas usadas pela feature)

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGSERIAL | sim | PK | Sequência implícita |
| carga_id | BIGINT | sim | FK tab_carga(id) | Composição do item (RDN-001) |
| vasilhame_id | BIGINT | sim | FK tab_vasilhame(id) | Composição do item (RDN-001) |
| preco_custo | NUMERIC(12,2) | sim | - | HU-002 |
| preco_venda | NUMERIC(12,2) | sim | - | HU-002 |
| limite_minimo | INTEGER | sim | - | Default 0 = sem alerta (RF-003) |
| estoque_cheios | INTEGER | sim | - | Default 0; saldo de cheios, movimentado por HU-006/HU-007 |
| estoque_vazios | INTEGER | sim | - | Default 0; saldo de vazios no pátio |
| criado_em | TIMESTAMP | sim | - | Auditoria |
| atualizado_em | TIMESTAMP | sim | - | Auditoria |
| ativo | BOOLEAN | sim | - | Default TRUE |

## 2. Regras de validação (RF-003, RF-032, RGN-004, CT-001)

- `limite_minimo` é um inteiro maior ou igual a zero; valor negativo é bloqueado com a
  mensagem "O limite mínimo de estoque não pode ser negativo." (422).
- O limite se refere ao estoque de **cheios**, nunca a vazios ou "em rua" (assumption da
  spec, RF-003).
- `limite_minimo = 0` significa "limite não definido": o produto não gera alerta, mesmo com
  saldo zerado (edge case da spec).

## 3. Condição do alerta (derivada, sem tabela)

```text
alerta_ativo(produto) = limite_minimo > 0 AND estoque_cheios <= limite_minimo
```

| Caso | estoque_cheios | limite_minimo | Alerta |
|---|---|---|---|
| Saldo acima do limite | 25 | 20 | não |
| Saldo exatamente no limite | 20 | 20 | sim (condição "atingir ou ficar abaixo", CT-002) |
| Saldo abaixo do limite | 15 | 20 | sim |
| Limite não definido | 0 | 0 | não (edge case da spec) |

**Dispensa:** o alerta deixa de existir quando um carregamento (HU-006) eleva
`estoque_cheios` acima de `limite_minimo` na mesma transação (RGN-004, CT-003, RNF-005).
Carregamento parcial que não eleve o saldo acima do limite mantém o alerta. Alterar o limite
não dispensa nem ativa alerta retroativamente.

## 4. Transições de estado

**Limite do produto (tab_produto):**

| De | Para | Gatilho | Regra |
|---|---|---|---|
| limite A | limite B | PUT /api/produtos/{id}/limite-minimo | A qualquer momento; apenas `>= 0` |
| sem alerta | com alerta | Venda baixa o saldo para <= limite (HU-007) | Derivado; não é evento persistido |
| com alerta | sem alerta | Carregamento eleva saldo acima do limite (HU-006) | Única via de dispensa (RGN-004) |

## 5. Relacionamento entre entidades

```text
tab_produto (limite_minimo, estoque_cheios)  --condicao-->  alerta derivado
tab_venda  (baixa estoque_cheios)   HU-007
tab_carregamento_item  (eleva estoque_cheios)  HU-006
```

- O saldo de cheios é movimentado exclusivamente por vendas (HU-007) e carregamentos
  (HU-006), sempre na mesma transação do movimento (RNF-005, CONVENTIONS §5).
- O alerta é consumido pelo painel de estoque (HU-012) e pelo dashboard (RF-053/HU-019).

**Nota de consistência:** o projeto prevê `tab_estoque` (saldo por produto) como modelo
possível, mas o schema vigente mantém `estoque_cheios`, `estoque_vazios` e `limite_minimo`
na `tab_produto` (migrations V1 e V3).
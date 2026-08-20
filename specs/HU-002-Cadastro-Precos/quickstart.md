# Validação End-to-End: Cadastro de Preços (Custo e Venda)

**HU**: HU-002 - Cadastro de Preços (Custo e Venda)
**Branch**: `HU-002-Cadastro-Precos`

Guia para provar os critérios de aceitação (CT-001 a CT-003) executando o sistema completo.
Detalhes de contratos em `contracts/api.md` e `contracts/ui.md`; estrutura do banco em
`data-model.md`.

## Pré-requisitos

| Item | Valor esperado |
|---|---|
| PostgreSQL | Rodando em `localhost:5432`, banco `gerenciador_estoque` (ou variáveis do `gerenciador_estoque_infra/env/dev.env`) |
| API | `http://localhost:8080` (`mvn spring-boot:run` no `gerenciador_estoque_api`) |
| App | `http://localhost:5173` (`npm run dev` no `gerenciador_estoque_app`, proxy `/api` para a API) |

**Setup:**

```bash
# 1. Banco
createdb gerenciador_estoque

# 2. API (aplica as migrations Flyway na subida)
cd ../gerenciador_estoque_api
mvn spring-boot:run

# 3. App
cd ../gerenciador_estoque_app
npm install
npm run dev
```

As migrations V1 e V8 criam os produtos "Gás P13" e "Água Galão 20L" com preços zerados,
prontos para preenchimento na tela.

## Cenários de prova (dado → quando → esperado)

### CT-001 - Cadastro aceita preço de custo e preço de venda

1. Dado o app aberto no cadastro de produtos.
2. Quando o usuário informa preço de custo R$ 50,00 e preço de venda R$ 80,00 em um produto.
3. Então o sistema salva ambos os valores junto ao produto e a listagem os exibe.

**Prova complementar (API):** `POST /api/produtos` com `"precoCusto":"50.00",
"precoVenda":"80.00"` retorna 200 com os valores no corpo. `GET /api/produtos/{id}` exibe os
preços vigentes.

**Prova de dado (CONSTITUICAO §IV.1):** `SELECT preco_custo, preco_venda FROM tab_produto
WHERE id = {id}` retorna 50.00 e 80.00.

### CT-002 - Validação de preço de venda não inferior ao custo

1. Dado um produto com preço de custo R$ 50,00.
2. Quando o usuário informa preço de venda inferior a R$ 50,00.
3. Então o sistema bloqueia o salvamento com a mensagem "O preço de venda não pode ser menor
   que o preço de custo." (422, RGN-005).
4. Quando o usuário informa preço de venda igual ou superior a R$ 50,00, então o salvamento é
   aceito (inclui venda igual ao custo).

**Prova complementar (API):** `PUT /api/produtos/{id}` com `"precoCusto":"50.00",
"precoVenda":"49.00"` retorna 422; com `"precoVenda":"50.00"` retorna 200.

### CT-003 - Alteração de preço sem recalcular vendas antigas

1. Dado um produto com preço de venda R$ 80,00 e uma venda lançada com esse valor (venda
   lançada via HU-007).
2. Quando o usuário altera o preço de venda para R$ 90,00.
3. Então a venda já lançada permanece com o preço de R$ 80,00 e uma nova venda usa o preço de
   R$ 90,00.

**Prova de dado (CONSTITUICAO §IV.1):**

```sql
-- venda antiga permanece com o valor unitario original
SELECT valor_unitario FROM tab_venda WHERE id = {vendaAntiga};  -- 80.00
-- produto com o preco novo vigente
SELECT preco_venda FROM tab_produto WHERE id = {produtoId};     -- 90.00
```

**Prova complementar (API):** `GET /api/produtos/{id}` retorna `precoVenda: "90.00"` após a
alteração, e a venda antiga (GET /api/vendas ou relatório) mantém `valorUnitario: "80.00"`.

## Roteiro resumido

| Passo | Ação | Resultado esperado |
|---|---|---|
| 1 | Cadastrar produto com custo R$ 50,00 e venda R$ 80,00 | Salvo com os dois preços |
| 2 | Tentar salvar venda R$ 49,00 com custo R$ 50,00 | Bloqueado (RGN-005) |
| 3 | Salvar venda R$ 50,00 (igual ao custo) | Aceito |
| 4 | Lançar uma venda com o preço vigente (HU-007) | Venda grava o valor unitário |
| 5 | Alterar preço de venda para R$ 90,00 | Venda antiga mantém o valor; cadastro mostra R$ 90,00 |

## Fora do escopo

- Uso dos preços no lançamento de venda (HU-007) e nos relatórios (HU-015 a HU-018): este
  guia apenas prova o cadastro e o congelamento via dado.
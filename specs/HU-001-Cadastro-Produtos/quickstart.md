# Validação End-to-End: Cadastro de Produtos (Carga + Vasilhame)

**HU**: HU-001 - Cadastro de Produtos (Carga + Vasilhame)
**Branch**: `HU-001-Cadastro-Produtos`

Guia para provar os critérios de aceitação (CT-001 a CT-004) executando o sistema completo.
Detalhes de contratos em `contracts/api.md` e `contracts/ui.md`; estrutura do banco em
`data-model.md`.

## Pré-requisitos

| Item | Valor esperado |
|---|---|
| PostgreSQL | Rodando em `localhost:5432`, banco `gerenciador_estoque` (ou usar as variáveis do `gerenciador_estoque_infra/env/dev.env`) |
| API | `http://localhost:8080` (`mvn spring-boot:run` no `gerenciador_estoque_api`) |
| App | `http://localhost:5173` (`npm run dev` no `gerenciador_estoque_app`, proxy `/api` para a API) |

**Setup:**

```bash
# 1. Banco (variaveis locais ou as do dev.env)
createdb gerenciador_estoque

# 2. API (aplica as migrations Flyway V1 em diante na subida)
cd ../gerenciador_estoque_api
mvn spring-boot:run

# 3. App
cd ../gerenciador_estoque_app
npm install
npm run dev
```

As migrations V1 e V8 criam as cargas (`Gas`, `Agua`) e os vasilhames (`P13`, `Galão 20L`)
de fábrica; o vasilhame `P45` fica inativo (V5).

## Cenários de prova (dado → quando → esperado)

### CT-001 - Cadastro aceita carga e vasilhame

1. Dado o app aberto na tela "Produtos", quando o usuário clica em "Novo produto".
2. Quando o usuário seleciona a carga "Gás" e o vasilhame "P13" e informa preços válidos.
3. Então o produto é salvo e aparece na listagem.

**Prova complementar (API):** `POST /api/produtos` com `{"carga":{"id":1},"vasilhame":{"id":1},
"precoCusto":"50.00","precoVenda":"80.00","limiteMinimo":0}` retorna 200 com o produto criado.
Sem carga ou sem vasilhame, retorna 422 com a mensagem correspondente (contracts/api.md).

### CT-002 - Listagem pelo nome combinado

1. Dado produtos cadastrados (ex.: "Gás P13", "Gás P45" inativo, "Água Galão 20L").
2. Quando o usuário consulta a listagem de produtos.
3. Então cada item exibe o nome combinado de carga + vasilhame (ex.: "Gás P13", "Água Galão 20L").

**Prova complementar (API):** `GET /api/produtos` retorna o campo `nome` combinado em cada item.

### CT-003 - Bloqueio de produto duplicado

1. Dado o produto "Gás P13" já cadastrado.
2. Quando o usuário tenta cadastrar novamente carga "Gás" + vasilhame "P13".
3. Então o sistema bloqueia com a mensagem "Já existe um produto com a combinação de carga e
   vasilhame informada." (422) e nenhum registro duplicado é gravado.

**Prova de dado (CONSTITUICAO §IV.1):** `SELECT count(*) FROM tab_produto WHERE carga_id =
{cargaGas} AND vasilhame_id = {vasilhameP13}` permanece em 1 após a tentativa.

### CT-004 - Edição e exclusão condicionada

1. Dado um produto sem movimentações vinculadas.
2. Quando o usuário edita a carga ou o vasilhame para uma combinação nova.
3. Então o sistema salva e a listagem exibe a nova combinação.
4. Quando o usuário exclui o produto sem movimentações, então a exclusão é aceita (204) e o
   produto some da listagem (permanece `ativo = false` no banco).
5. Dado um produto com vendas ou carregamentos vinculados (depende das HU-006 e HU-007), quando
   o usuário tenta excluí-lo, então o sistema bloqueia com 422 "Não é possível excluir o
   produto {nome} porque ele possui movimentações vinculadas."

**Prova de dado:** `SELECT count(*) FROM tab_produto WHERE ativo = false` confirma a exclusão
lógica; nenhuma linha é removida fisicamente (RNF-007).

## Roteiro resumido

| Passo | Ação | Resultado esperado |
|---|---|---|
| 1 | Cadastrar "Gás P13" com preços válidos | Produto salvo e listado como "Gás P13" |
| 2 | Tentar cadastrar "Gás P13" novamente | Bloqueado com mensagem de duplicidade |
| 3 | Cadastrar "Água Galão 20L" | Aceito (combinação diferente) |
| 4 | Editar "Água Galão 20L" para "Água P13" | Salvo; listagem atualizada |
| 5 | Excluir um produto sem movimentações | Excluído (lógico), some da listagem |

## Fora do escopo

- Preços (HU-002), limite mínimo e alerta (HU-003), exclusão com movimentações reais
  (exige as tabelas de venda HU-007 e carregamento HU-006 para a prova completa do bloqueio).
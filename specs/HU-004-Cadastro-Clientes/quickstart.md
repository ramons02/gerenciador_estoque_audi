# Validação End-to-End: Cadastro de Clientes

**HU**: HU-004 - Cadastro de Clientes
**Branch**: `HU-004-Cadastro-Clientes`

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

# 2. API (aplica as migrations Flyway na subida, inclui tab_cliente da V3)
cd ../gerenciador_estoque_api
mvn spring-boot:run

# 3. App
cd ../gerenciador_estoque_app
npm install
npm run dev
```

## Cenários de prova (dado → quando → esperado)

### CT-001 - Cadastro captura nome, telefone, endereço e documento (opcional)

1. Dado a tela de cadastro de clientes.
2. Quando o vendedor informa nome, telefone e endereço.
3. Então o sistema salva o cliente com todos os dados e o exibe na listagem.

**Prova complementar (API):** `POST /api/clientes` com `{"nome":"Maria Souza",
"telefone":"9888-7777","endereco":"Av. Central, 45"}` retorna 200. Sem nome, retorna 422
"Informe o nome do cliente.". Cadastro com apenas nome (telefone e endereço vazios) é aceito.

**Prova de dado (CONSTITUICAO §IV.1):** `SELECT nome, telefone, endereco FROM tab_cliente
WHERE id = {id}` retorna os valores informados.

**Observação:** o campo "documento" não existe no schema (removido por LGPD na migration
V6); o cadastro aceita o cliente sem documento, conforme o edge case da spec.

### CT-002 - Busca do cliente por nome ou telefone no momento da venda

1. Dado um cliente cadastrado ("Maria Souza", telefone "9888-7777").
2. Quando o vendedor digita parte do nome ("mar") ou o telefone ("9888") na busca da venda.
3. Então o sistema retorna o cliente correspondente na mesma tela e permite selecioná-lo.

**Prova complementar (API):**
```
GET /api/clientes?q=mar      → retorna "Maria Souza"
GET /api/clientes?q=9888     → retorna "Maria Souza"
GET /api/clientes            → retorna todos os ativos
```

### CT-003 - Cliente obrigatório na venda de vasilhame novo; sem Fiado

1. Dado uma venda de vasilhame novo em andamento (HU-007).
2. Quando o vendedor tenta confirmar sem selecionar um cliente.
3. Então o sistema bloqueia a confirmação por cliente obrigatório.
4. Quando o vendedor seleciona o cliente e confirma, então o sistema aceita a venda e vincula
   o vasilhame ao cliente ("em rua", RF-024/RF-026/RF-028).
5. Dado o lançamento de venda, quando o vendedor consulta as formas de pagamento, então a
   forma Fiado não existe nas opções (RGN-002).

**Prova de dado (CONSTITUICAO §IV.1):** `SELECT quantidade FROM tab_cliente_vasilhame WHERE
cliente_id = {id}` cresce após a venda de vasilhame novo.

## Roteiro resumido

| Passo | Ação | Resultado esperado |
|---|---|---|
| 1 | Cadastrar "Maria Souza" com telefone e endereço | Salvo e exibido na listagem |
| 2 | Cadastrar cliente com apenas o nome | Aceito (demais campos opcionais) |
| 3 | Buscar por "mar" e por "9888" na tela de venda | Cliente encontrado e selecionável |
| 4 | Confirmar venda de vasilhame novo sem cliente | Bloqueado (cliente obrigatório) |
| 5 | Confirmar a mesma venda com cliente | Aceita; vasilhame vinculado como "em rua" |
| 6 | Conferir as formas de pagamento da venda | Fiado não existe (RGN-002) |

## Fora do escopo

- Fluxo completo da venda (HU-007) e devolução de vazios (HU-011): este guia prova o
  cadastro, a busca e a regra de obrigatoriedade; os fluxos de venda e devolução são
  cobertos pelos quickstarts daquelas HU.
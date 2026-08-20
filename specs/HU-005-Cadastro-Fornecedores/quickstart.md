# Validação End-to-End: Cadastro de Fornecedores (Distribuidoras)

**HU**: HU-005 - Cadastro de Fornecedores (Distribuidoras)
**Branch**: `HU-005-Cadastro-Fornecedores`

Guia para provar os critérios de aceitação (CT-001 e CT-002) executando o sistema completo.
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

# 2. API (aplica as migrations Flyway na subida, inclui tab_fornecedor da V2)
cd ../gerenciador_estoque_api
mvn spring-boot:run

# 3. App
cd ../gerenciador_estoque_app
npm install
npm run dev
```

## Cenários de prova (dado → quando → esperado)

### CT-001 - Cadastro captura nome da distribuidora e contato

1. Dado a tela de cadastro de fornecedores.
2. Quando o administrador informa o nome da distribuidora ("Ultragaz") e o contato.
3. Então o sistema salva o fornecedor com os dados informados e o exibe na listagem.

**Prova complementar (API):** `POST /api/fornecedores` com `{"nome":"Ultragaz",
"contato":"0800-1234"}` retorna 200. Sem nome, retorna 422 "Informe o nome do fornecedor.".
Cadastro de "Ultragaz" novamente retorna 422 "Já existe um fornecedor com este nome."
(duplicidade na seleção do carregamento).

**Prova de dado (CONSTITUICAO §IV.1):** `SELECT nome, contato, ativo FROM tab_fornecedor
WHERE id = {id}` retorna os valores informados com `ativo = true`.

### CT-002 - Fornecedor disponível na seleção do registro de carregamento

1. Dado um fornecedor cadastrado ("Ultragaz").
2. Quando o administrador abre o registro de carregamento (HU-006).
3. Então o fornecedor aparece disponível na seleção de fornecedor.
4. Quando o administrador seleciona o fornecedor e salva o carregamento, então o sistema
   registra a entrada vinculada ao fornecedor selecionado.
5. Sem fornecedor selecionado, o carregamento é rejeitado (carregamento sem fornecedor não
   existe, RF-010).

**Prova complementar (API):**
```
GET /api/fornecedores            → lista inclui "Ultragaz" (fonte da seleção)
POST /api/carregamentos          → sem fornecedor.id retorna 422
POST /api/carregamentos          → com fornecedor.id retorna 200 (implementação HU-006)
```

**Prova de dado (CONSTITUICAO §IV.1):** `SELECT fornecedor_id FROM tab_carregamento WHERE
id = {id}` retorna o id do fornecedor cadastrado.

## Roteiro resumido

| Passo | Ação | Resultado esperado |
|---|---|---|
| 1 | Cadastrar "Ultragaz" com contato | Salvo e exibido na listagem |
| 2 | Cadastrar "Ultragaz" novamente | Bloqueado (nome duplicado) |
| 3 | Cadastrar "Nacional" | Aceito |
| 4 | Abrir o registro de carregamento | "Ultragaz" e "Nacional" na seleção |
| 5 | Tentar salvar carregamento sem fornecedor | Bloqueado (fornecedor obrigatório) |
| 6 | Salvar carregamento com fornecedor | Vinculado ao fornecedor selecionado |

## Fora do escopo

- Fluxo completo do carregamento (HU-006), incluindo itens, custos e movimentação de
  estoque: este guia prova o cadastro e a disponibilidade na seleção; o registro em si é
  coberto pelo quickstart da HU-006.
# Contrato da API: Cadastro de Produtos (Carga + Vasilhame)

**HU**: HU-001 - Cadastro de Produtos (Carga + Vasilhame)
**Branch**: `HU-001-Cadastro-Produtos`

Base URL: `/api`. Erros estruturados em JSON conforme `GlobalExceptionHandler` (CONVENTIONS §6).

## Formato de erro padrão

```json
{
  "timestamp": "2026-08-20T10:00:00",
  "status": 422,
  "message": "mensagem de erro em pt-BR"
}
```

| HTTP | Situação |
|---|---|
| 200 | Sucesso (recurso ou lista) |
| 204 | Sucesso sem corpo (DELETE) |
| 400 | Parâmetro obrigatório ausente |
| 404 | Recurso não encontrado ou inativo |
| 422 | Validação de negócio (mensagem pt-BR) |
| 500 | Erro interno não tratado |

---

## 1. Cargas (`/api/cargas`)

Módulo de apoio: lista as cargas disponíveis para montar a composição do produto (CT-001).

### GET /api/cargas

Lista as cargas ativas.

**Resposta 200**:
```json
[
  { "id": 1, "nome": "Gas" },
  { "id": 2, "nome": "Agua" }
]
```

## 2. Vasilhames (`/api/vasilhames`)

Módulo de apoio: lista os vasilhames disponíveis para montar a composição do produto (CT-001).

### GET /api/vasilhames

Lista os vasilhames ativos.

**Resposta 200**:
```json
[
  { "id": 1, "nome": "P13", "precoCasco": "0.00" },
  { "id": 3, "nome": "Galão 20L", "precoCasco": "0.00" }
]
```

## 3. Produtos (`/api/produtos`)

### GET /api/produtos

Lista os produtos ativos, cada um com o nome combinado derivado de carga + vasilhame (CT-002).

**Resposta 200**:
```json
[
  {
    "id": 1,
    "carga": { "id": 1, "nome": "Gas" },
    "vasilhame": { "id": 1, "nome": "P13" },
    "nome": "Gas P13",
    "precoCusto": "50.00",
    "precoVenda": "80.00",
    "limiteMinimo": 0,
    "estoqueCheios": 0,
    "estoqueVazios": 0,
    "ativo": true
  }
]
```

### GET /api/produtos/{id}

Obtém um produto ativo por id.

**Resposta 200**: corpo igual ao item da listagem.

**Erros**: 404 `Produto não encontrado(a) com id {id}.`

### POST /api/produtos

Cria um produto (CT-001).

**Request**:
```json
{
  "carga": { "id": 1 },
  "vasilhame": { "id": 1 },
  "precoCusto": "50.00",
  "precoVenda": "80.00",
  "limiteMinimo": 0
}
```

**Resposta 200**: produto criado (corpo igual ao item da listagem, com `id` e `nome`).

**Erros (422)**:
- `Informe a carga do produto.` (carga ausente)
- `Informe o vasilhame do produto.` (vasilhame ausente)
- `O preço de custo deve ser maior que zero.`
- `O preço de venda deve ser maior que zero.`
- `O preço de venda não pode ser menor que o preço de custo.` (RGN-005)
- `Já existe um produto com a combinação de carga e vasilhame informada.` (CT-003)

### PUT /api/produtos/{id}

Atualiza um produto existente (CT-004). A nova combinação deve manter a unicidade.

**Request**: mesmo corpo do POST.

**Resposta 200**: produto atualizado.

**Erros**: 404 (produto não encontrado) e 422 (mesmas mensagens do POST, incluindo duplicidade).

### DELETE /api/produtos/{id}

Marca o produto como inativo (`ativo = false`), exclusão lógica.

**Resposta 204**: sem corpo.

**Erros**:
- 404: `Produto não encontrado(a) com id {id}.`
- 422: `Não é possível excluir o produto {nome} porque ele possui movimentações vinculadas.`
  (CT-004, quando existirem vendas ou carregamentos vinculados)

---

## Rastreabilidade

- Endpoints: RF-001 (composição carga + vasilhame), CT-001 a CT-004.
- Commits e documentação do módulo referenciam HU-001 (CONSTITUICAO §X, CONVENTIONS §9).
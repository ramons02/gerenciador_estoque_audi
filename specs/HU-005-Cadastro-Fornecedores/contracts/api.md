# Contrato da API: Cadastro de Fornecedores (Distribuidoras)

**HU**: HU-005 - Cadastro de Fornecedores (Distribuidoras)
**Branch**: `HU-005-Cadastro-Fornecedores`

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

## 1. Fornecedores (`/api/fornecedores`)

### GET /api/fornecedores

Lista os fornecedores ativos. É a fonte da seleção de fornecedor no registro de carregamento
(CT-002, RF-010).

**Resposta 200**:
```json
[
  {
    "id": 1,
    "nome": "Ultragaz",
    "contato": "0800-1234",
    "ativo": true
  }
]
```

### GET /api/fornecedores/{id}

Obtém um fornecedor ativo por id.

**Erros**: 404 `Fornecedor não encontrado(a) com id {id}.`

### POST /api/fornecedores

Cria um fornecedor (CT-001).

**Request**:
```json
{
  "nome": "Ultragaz",
  "contato": "0800-1234"
}
```

**Resposta 200**: fornecedor criado com `id` e `ativo: true`.

**Erros (422):**
- `Informe o nome do fornecedor.` (nome ausente ou vazio)
- `Já existe um fornecedor com este nome.` (duplicidade entre ativos)

### PUT /api/fornecedores/{id}

Atualiza um fornecedor existente.

**Request**: mesmo corpo do POST.

**Resposta 200**: fornecedor atualizado.

**Erros**: 404 (fornecedor não encontrado) e 422 (mesmas mensagens do POST).

### DELETE /api/fornecedores/{id}

Marca o fornecedor como inativo (`ativo = false`), exclusão lógica. Carregamentos históricos
permanecem vinculados (RNF-007).

**Resposta 204**: sem corpo.

**Erros**: 404 `Fornecedor não encontrado(a) com id {id}.`

## 2. Uso no registro de carregamento (consumo pela HU-006)

O registro de carregamento (HU-006) consome `GET /api/fornecedores` para a seleção de
fornecedor e envia `fornecedor.id` no payload do carregamento. O carregamento é rejeitado sem
fornecedor cadastrado (422), pois `tab_carregamento.fornecedor_id` é obrigatório (RF-010).

---

## Rastreabilidade

- Endpoints: RF-005 (cadastro de fornecedores/distribuidoras), RF-010 (disponibilidade na
  seleção do carregamento), CT-001 e CT-002.
- Commits e documentação do módulo referenciam HU-005 (CONSTITUICAO §X, CONVENTIONS §9).
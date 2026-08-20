# Contrato da API: Cadastro de Clientes

**HU**: HU-004 - Cadastro de Clientes
**Branch**: `HU-004-Cadastro-Clientes`

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

## 1. Clientes (`/api/clientes`)

### GET /api/clientes

Lista os clientes ativos. Com o parâmetro `q`, busca por nome ou telefone (CT-002).

**Request**:
```
GET /api/clientes
GET /api/clientes?q=joao
GET /api/clientes?q=9999-8888
```

**Resposta 200**:
```json
[
  {
    "id": 1,
    "nome": "João da Silva",
    "telefone": "9999-8888",
    "endereco": "Rua das Flores, 123",
    "ativo": true
  }
]
```

**Observação:** não há campo `documento` (removido por LGPD, migration V6).

### GET /api/clientes/{id}

Obtém um cliente ativo por id.

**Erros**: 404 `Cliente não encontrado(a) com id {id}.`

### POST /api/clientes

Cria um cliente (CT-001).

**Request**:
```json
{
  "nome": "Maria Souza",
  "telefone": "9888-7777",
  "endereco": "Av. Central, 45"
}
```

**Resposta 200**: cliente criado com `id` e `ativo: true`.

**Erros (422):**
- `Informe o nome do cliente.` (nome ausente ou vazio)

### PUT /api/clientes/{id}

Atualiza um cliente existente.

**Request**: mesmo corpo do POST.

**Resposta 200**: cliente atualizado.

**Erros**: 404 (cliente não encontrado) e 422 (nome obrigatório).

### DELETE /api/clientes/{id}

Marca o cliente como inativo (`ativo = false`), exclusão lógica. O histórico de vendas e de
vasilhames "em rua" é preservado (RNF-007).

**Resposta 204**: sem corpo.

**Erros**: 404 `Cliente não encontrado(a) com id {id}.`

## 2. Busca na venda (consumo pela HU-007)

O lançamento de venda (HU-007) consome `GET /api/clientes?q=<termo>` para encontrar e
selecionar o cliente na mesma tela (CT-002, RNF-001). Na venda de vasilhame novo, a venda
exige `cliente.id` no payload e rejeita a confirmação sem ele (CT-003, RF-024/RF-026). A
forma de pagamento Fiado não existe na venda (RGN-002).

---

## Rastreabilidade

- Endpoints: RF-004 (cadastro), RF-028 (vínculo com "em rua", na venda/devolução), RGN-002
  (sem Fiado), CT-001 a CT-003.
- Commits e documentação do módulo referenciam HU-004 (CONSTITUICAO §X, CONVENTIONS §9).
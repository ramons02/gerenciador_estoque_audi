# Contrato da API: Cadastro de Preços (Custo e Venda)

**HU**: HU-002 - Cadastro de Preços (Custo e Venda)
**Branch**: `HU-002-Cadastro-Precos`

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
| 200 | Sucesso (recurso) |
| 400 | Parâmetro obrigatório ausente |
| 404 | Recurso não encontrado ou inativo |
| 422 | Validação de negócio (mensagem pt-BR) |
| 500 | Erro interno não tratado |

---

## 1. Cadastro e alteração de preços

Os preços são atributos do produto e são informados no cadastro e na atualização do produto
(RF-002). Não existe endpoint dedicado de preço.

### POST /api/produtos

Cria um produto com os preços de custo e venda (CT-001).

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

**Resposta 200**: produto criado, com `precoCusto` e `precoVenda` no corpo.

**Erros (422)**:
- `O preço de custo deve ser maior que zero.`
- `O preço de venda deve ser maior que zero.`
- `O preço de venda não pode ser menor que o preço de custo.` (RGN-005, CT-002)

### PUT /api/produtos/{id}

Altera o preço de custo e/ou de venda a qualquer momento (CT-003). O corpo é o cadastro
completo do produto com os novos valores.

**Request**:
```json
{
  "carga": { "id": 1 },
  "vasilhame": { "id": 1 },
  "precoCusto": "52.00",
  "precoVenda": "90.00",
  "limiteMinimo": 0
}
```

**Resposta 200**: produto atualizado com os novos preços.

**Erros**: 404 (`Produto não encontrado(a) com id {id}.`) e 422 (mesmas mensagens do POST;
RGN-005 continua valendo na alteração).

## 2. Leitura dos preços

### GET /api/produtos

Lista os produtos com os preços vigentes (CT-001: exibição do cadastro).

**Resposta 200**: itens com `precoCusto` e `precoVenda` atuais.

### GET /api/produtos/{id}

Obtém um produto com os preços vigentes.

**Erros**: 404 quando o produto não existe ou está inativo.

## 3. Congelamento do preço na venda (consumo pela HU-007)

O lançamento de venda (POST /api/vendas, implementado na HU-007) lê o `precoVenda` vigente
no momento e grava `valorUnitario` e `total` na venda. Este contrato apenas registra a
dependência: vendas antigas não são recalculadas após alteração de preço (CT-003).

---

## Rastreabilidade

- Endpoints: RF-002 (preços no cadastro do produto), RGN-005 (venda não inferior ao custo),
  CT-001 a CT-003.
- Commits e documentação do módulo referenciam HU-002 (CONSTITUICAO §X, CONVENTIONS §9).
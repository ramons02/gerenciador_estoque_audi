# Contract API: Bloqueio de Venda sem Estoque

**HU**: HU-013 - Bloqueio de Venda sem Estoque
**Módulo**: `com.gerenciador.estoque.venda` | **Base**: `/api/vendas`

Endpoint de criação de venda com validação de estoque (RF-031). O bloqueio vale para todos os fluxos (balcão, entrega, troca, vasilhame novo - CT-003).

## POST /api/vendas

### Request

```json
{
  "tipo": "BALCAO",
  "formaPagamento": "DINHEIRO",
  "idCliente": null,
  "itens": [
    {
      "idProduto": 1,
      "quantidade": 3,
      "tipoOperacao": "TROCA"
    }
  ]
}
```

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| tipo | string | sim | BALCAO ou ENTREGA (RF-020) |
| formaPagamento | string | sim | DINHEIRO, PIX ou CARTAO; nunca FIADO (RGN-002, RF-021) |
| idCliente | number | nao | Cliente (obrigatório para entrega e vasilhame novo) |
| itens[].idProduto | number | sim | Produto existente |
| itens[].quantidade | integer | sim | Maior que zero e menor ou igual ao saldo de cheios (RF-031) |
| itens[].tipoOperacao | string | sim | NORMAL, TROCA, VASILHAME_NOVO ou AVULSA (CT-003) |

### Response 201 Created

```json
{
  "id": 205,
  "tipo": "BALCAO",
  "formaPagamento": "DINHEIRO",
  "total": 150.00,
  "dataHora": "2026-08-20T14:30:00-03:00",
  "idUsuario": 2,
  "itens": [
    {
      "idProduto": 1,
      "produto": "Gás P13",
      "quantidade": 3,
      "tipoOperacao": "TROCA",
      "valorUnitario": 50.00
    }
  ]
}
```

A venda é confirmada somente quando TODOS os itens passam na validação de estoque (RNF-005).

### Erros

| HTTP | codigo | mensagem (pt-BR) |
|---|---|---|
| 400 | VALIDACAO | "Informe ao menos um item de venda." |
| 400 | VALIDACAO | "A quantidade deve ser maior que zero." |
| 400 | FORMA_PAGAMENTO_INVALIDA | "A forma de pagamento não é aceita. Formas aceitas: Dinheiro, PIX, Cartão." (RGN-002) |
| 404 | PRODUTO_NAO_ENCONTRADO | "Produto não encontrado." |
| 404 | CLIENTE_NAO_ENCONTRADO | "Cliente não encontrado." |
| 409 | ESTOQUE_INSUFICIENTE | "Estoque insuficiente para 3 unidade(s) de Gás P13. Disponível: 1." (CT-001) |
| 409 | ESTOQUE_INSUFICIENTE | "Estoque insuficiente para 2 unidade(s) de Água Galão 20L. Disponível: 0." |

Regras dos erros:

- O bloqueio por estoque é sempre 409 com `ESTOQUE_INSUFICIENTE`, nunca 500 (Constituição §XI.3) e nunca erro de validação de formulário (CONVENTIONS §6).
- Erros estruturados em JSON, sempre com `codigo` e `mensagem` em pt-BR.

## PUT /api/vendas/{id}/cancelamento

Cancelamento de venda com reversão de estoque e caixa (RGN-007, RNF-007). Após o cancelamento, o saldo liberado volta a permitir novas vendas (Edge Case do spec).

### Request

```json
{
  "motivo": "Cliente devolveu o vasilhame cheio sem uso"
}
```

### Response 200

```json
{
  "id": 205,
  "status": "CANCELADA",
  "dataHora": "2026-08-20T15:00:00-03:00",
  "idUsuario": 2,
  "motivo": "Cliente devolveu o vasilhame cheio sem uso"
}
```

### Erros

| HTTP | codigo | mensagem (pt-BR) |
|---|---|---|
| 404 | VENDA_NAO_ENCONTRADA | "Venda não encontrada." |
| 409 | VENDA_JA_CANCELADA | "A venda já está cancelada." |
| 400 | VALIDACAO | "Informe o motivo do cancelamento." |

## Notas de implementação

- `POST /api/vendas` e `PUT /api/vendas/{id}/cancelamento` são `@Transactional`: estoque + caixa + movimentações na mesma transação (RNF-005).
- Leitura do saldo com lock `PESSIMISTIC_WRITE` por produto (RNF-008, CONVENTIONS §8); sem lock, o CT-002 falha.
- Bloqueio considera apenas `qtd_cheios`; falta de vazios ou em rua nunca bloqueia (Edge Case do spec).
- Quantidade igual ao saldo é permitida (Edge Case do spec).
- Nenhuma venda é apagada fisicamente (RNF-007).
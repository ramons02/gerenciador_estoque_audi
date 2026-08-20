# API Contract: HU-010 - Venda de Vasilhame Novo (Casco + Carga)

**HU**: HU-010 | **Feature**: Venda de Vasilhame Novo | **Date**: 2026-08-20 | **Spec**: [spec.md](../spec.md)
**Requisitos vinculados**: RF-024, RF-026, RF-028, RDN-008, RNF-005, RNF-008, RGN-010
**Data model**: [data-model.md](../data-model.md)
**Base comum**: [api.md HU-007](../HU-007-Lancamento-Venda/contracts/api.md)

A HU-010 é variação do `POST /api/vendas` da HU-007: o item usa `tipoItem = CASCO_NOVO` e o request exige `idCliente`. Não existe rota separada de venda de vasilhame novo.

---

## POST /api/vendas (vasilhame novo)

```json
{
  "tipo": "BALCAO",
  "formaPagamento": "PIX",
  "idUsuario": 1,
  "idCliente": 5,
  "itens": [
    { "idProduto": 1, "quantidade": 2, "tipoItem": "CASCO_NOVO" }
  ]
}
```

**Regras adicionais** (Service):
- `tipoItem = CASCO_NOVO`: o Service calcula `precoUnitario = precoCasco (tab_vasilhame) + precoVenda (carga)` por item e grava `precoCasco` no item (CT-002; RGN-010). Campos de preço não são aceitos no request.
- `idCliente` obrigatório quando há item CASCO_NOVO (CT-004; RF-028): sem cliente, 422.
- Ao confirmar: baixa N cheios e registra N vasilhames "em rua" para o cliente (incrementa `tab_cliente_vasilhame`), na mesma transação (CT-003; RF-026). Movimentações SAIDA_CHEIO e EM_RUA com o mesmo `id_referencia`.
- Validação de cheios igual à HU-007, sob lock (RF-031; RNF-008).
- Acréscimo do cartão (se CARTAO) aplica-se somente a itens de carga Gás; itens de carga Água usam preço normal (RF-021-A, RGN-002 - regra da HU-007).

**Response 201** - venda de vasilhame novo:

```json
{
  "id": 45,
  "dataHora": "2026-08-20T12:00:00-03:00",
  "tipo": "BALCAO",
  "formaPagamento": "PIX",
  "taxaEntrega": null,
  "acrescimoCartao": null,
  "total": 460.00,
  "status": "ATIVA",
  "idUsuario": 1,
  "idCliente": 5,
  "itens": [
    { "id": 4, "idProduto": 1, "nomeProduto": "Gás P13", "quantidade": 2, "precoUnitario": 230.00, "precoCasco": 115.00, "tipoItem": "CASCO_NOVO" }
  ]
}
```

(ex.: preço da carga 115,00 + casco 115,00 = 230,00 por unidade; total 460,00)

**Erros adicionais**:
- `422` - "Informe o cliente para venda de vasilhame novo." (CT-004)
- `404` - "Cliente não encontrado."

---

## GET /api/clientes

Busca de clientes para o seletor do lançamento (RF-004); cadastro rápido antes de concluir (spec.md Edge Cases) usa o `POST /api/clientes` da HU-004.

---

## GET /api/produtos

Lista produtos incluindo o `precoCasco` do vasilhame, para a tela exibir o preço composto (CT-002).

---

## Contrato de erros (comum)

Igual ao da HU-007, acrescido de:

| Status | Código | Mensagem padrão (pt-BR) |
|---|---|---|
| 404 | NAO_ENCONTRADO | "Cliente não encontrado." |
| 422 | CLIENTE_OBRIGATORIO | "Informe o cliente para venda de vasilhame novo." |

---

## Nota de contrato

O cancelamento da venda de vasilhame novo (HU-020) reverte: +N cheios, -N em rua e -N em `tab_cliente_vasilhame` do cliente, com novo rastro (RGN-007, RNF-007). A devolução posterior do vazio (RF-028) é a HU-011, que decrementa o mesmo controle com referência à devolução.
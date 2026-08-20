# API Contract: HU-009 - Troca de Vasilhame (Venda Normal)

**HU**: HU-009 | **Feature**: Troca de Vasilhame | **Date**: 2026-08-20 | **Spec**: [spec.md](../spec.md)
**Requisitos vinculados**: RF-023, RF-025, RDN-004, RDN-005, RNF-005, RNF-008
**Data model**: [data-model.md](../data-model.md)
**Base comum**: [api.md HU-007](../HU-007-Lancamento-Venda/contracts/api.md)

A HU-009 é variação do `POST /api/vendas` da HU-007: a troca é marcada no request (campo `troca: true` no item), sem rota separada (`/api/vendas/troca` não existe - research.md, Decisão 1).

---

## POST /api/vendas (venda com troca)

Mesmo request da HU-007, com o item marcado como troca:

```json
{
  "tipo": "BALCAO",
  "formaPagamento": "DINHEIRO",
  "idUsuario": 1,
  "itens": [
    { "idProduto": 1, "quantidade": 2, "tipoItem": "CHEIO", "troca": true }
  ]
}
```

**Regras adicionais** (Service):
- `troca: true`: o Service baixa N cheios E adiciona N vazios ao pátio na mesma transação (CT-001; RF-025; RDN-004). Movimentações SAIDA_CHEIO e ENTRADA_VAZIO com o mesmo `id_referencia`.
- A troca NÃO altera o total: valor calculado como qualquer venda (CT-003; RF-023). O vazio recebido não é item cobrado.
- Validação de cheios igual à HU-007, sob lock (RF-031; RNF-008).
- `troca` ausente ou false: comportamento padrão da HU-007 (sem entrada de vazio no pátio).

**Response 201** - venda com troca (mesma estrutura da HU-007, com o item trazendo `troca: true`):

```json
{
  "id": 44,
  "dataHora": "2026-08-20T11:30:00-03:00",
  "tipo": "BALCAO",
  "formaPagamento": "DINHEIRO",
  "taxaEntrega": null,
  "acrescimoCartao": null,
  "total": 230.00,
  "status": "ATIVA",
  "idUsuario": 1,
  "idCliente": null,
  "itens": [
    { "id": 3, "idProduto": 1, "nomeProduto": "Gás P13", "quantidade": 2, "precoUnitario": 115.00, "precoCasco": null, "tipoItem": "CHEIO", "troca": true }
  ]
}
```

**Erros**: mesmos da HU-007 (400 campos, 404 produto, 409 estoque insuficiente, 422 forma desabilitada).

---

## GET /api/estoque

Saldos por produto, incluindo `qtdVazios` (pátio), para conferir o efeito da troca (CT-004). Estrutura em [api.md HU-006](../HU-006-Chegada-Caminhao/contracts/api.md).

---

## Contrato de erros (comum)

Igual ao da HU-007 ([api.md HU-007](../HU-007-Lancamento-Venda/contracts/api.md)): 400/404/409/422/500 com mensagens em pt-BR. Nenhum código novo; a troca reusa o bloqueio de estoque da base comum.

---

## Nota de contrato

O cancelamento da venda com troca (HU-020) reverte a movimentação dupla: +N cheios e -N vazios, com novo rastro (RGN-007, RNF-007). Os dados gravados (SAIDA_CHEIO + ENTRADA_VAZIO com `id_referencia`) são a base dessa reversão.
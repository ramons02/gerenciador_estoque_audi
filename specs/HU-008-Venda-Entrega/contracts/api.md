# API Contract: HU-008 - Venda com Entrega (Taxa de Entrega)

**HU**: HU-008 | **Feature**: Venda com Entrega | **Date**: 2026-08-20 | **Spec**: [spec.md](../spec.md)
**Requisitos vinculados**: RF-020, RF-022, RF-041, RF-052, RGN-001
**Data model**: [data-model.md](../data-model.md)
**Base comum**: [api.md HU-007](../HU-007-Lancamento-Venda/contracts/api.md)

A HU-008 é variação do `POST /api/vendas` da HU-007. Não cria rota nova de venda: documenta os campos específicos da entrega e o endpoint de configuração da taxa.

---

## POST /api/vendas (tipo ENTREGA)

Mesmo request da HU-007, com `tipo = "ENTREGA"`:

```json
{
  "tipo": "ENTREGA",
  "formaPagamento": "DINHEIRO",
  "idUsuario": 1,
  "itens": [
    { "idProduto": 1, "quantidade": 1, "tipoItem": "CHEIO" }
  ]
}
```

**Regras adicionais** (Service):
- O Service lê `taxa_entrega` de `tab_configuracao` e soma ao total automaticamente (CT-001; RF-022). O campo `taxaEntrega` NÃO é aceito no request (valor vem da configuração).
- Chave `taxa_entrega` ausente: venda ENTREGA bloqueada (422, edge case do spec).
- `taxaEntrega` gravada no cabeçalho da venda (valores históricos preservados).

**Response 201** - venda com entrega:

```json
{
  "id": 43,
  "dataHora": "2026-08-20T11:00:00-03:00",
  "tipo": "ENTREGA",
  "formaPagamento": "DINHEIRO",
  "taxaEntrega": 10.00,
  "acrescimoCartao": null,
  "total": 125.00,
  "status": "ATIVA",
  "idUsuario": 1,
  "idCliente": null,
  "itens": [
    { "id": 2, "idProduto": 1, "nomeProduto": "Gás P13", "quantidade": 1, "precoUnitario": 115.00, "precoCasco": null, "tipoItem": "CHEIO" }
  ]
}
```

**Erros adicionais**:
- `422` - "A taxa de entrega não está configurada. Defina o valor em Configurações."

---

## GET /api/configuracoes

Retorna `taxaEntrega` vigente junto com formas de pagamento e acréscimo (RF-052):

```json
{
  "formasPagamento": ["DINHEIRO", "PIX", "CARTAO"],
  "acrescimoCartao": 5.00,
  "taxaEntrega": 10.00
}
```

---

## PUT /api/configuracoes

Altera configurações pelo administrador (CT-002; RF-052). Body parcial: apenas `taxaEntrega` para esta HU.

```json
{ "taxaEntrega": 15.00 }
```

**Response 200** - configurações atualizadas (mesma estrutura do GET).

**Erros**:
- `400` - "O valor da taxa de entrega deve ser maior ou igual a zero."
- `403` - papel sem permissão de administrador.

---

## Relatório de Vendas (CT-003)

Relatório discriminando tipo e total com taxa (RF-041, CONVENTIONS §10). Colunas: Data/Hora, Produto, Qtd, Valor Unitário, Total (R$), Forma de Pagamento, Tipo (Balcão/Entrega). Exportação CSV UTF-8 com cabeçalhos pt-BR (RNF-009). A implementação completa do relatório é da HU-016; a HU-008 garante que os dados (tipo e taxaEntrega) estejam gravados para ele.

---

## Contrato de erros (comum)

| Status | Código | Mensagem padrão (pt-BR) |
|---|---|---|
| 400 | VALIDACAO | "O valor da taxa de entrega deve ser maior ou igual a zero." |
| 403 | SEM_PERMISSAO | "Operação restrita ao administrador." |
| 422 | CONFIGURACAO_INVALIDA | "A taxa de entrega não está configurada. Defina o valor em Configurações." |
| 500 | ERRO_INTERNO | Sem exposição de detalhe técnico (§XI.3) |
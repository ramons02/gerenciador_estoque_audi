# API Contract: HU-007 - Lançamento Rápido de Venda (Balcão/Entrega)

**HU**: HU-007 | **Feature**: Lançamento Rápido de Venda | **Date**: 2026-08-20 | **Spec**: [spec.md](../spec.md)
**Requisitos vinculados**: RF-020, RF-021, RF-021-A, RF-031, RF-052, RNF-001, RNF-005, RNF-006, RNF-008, RGN-002
**Data model**: [data-model.md](../data-model.md)

Módulo REST `vendas` (pacote `com.gerenciador.estoque.business.venda`). Todas as respostas de erro são JSON estruturado `{ "codigo": "...", "mensagem": "..." }` em pt-BR (CONVENTIONS §6). Este contrato é a base comum das HU-008 (entrega), HU-009 (troca) e HU-010 (vasilhame novo), que documentam seus campos específicos.

---

## POST /api/vendas

Lançamento de venda. Operação transacional: valida saldo sob lock, calcula total, persiste venda + itens + movimentação e atualiza saldos (RNF-005, RNF-008).

**Request** (body JSON):

```json
{
  "tipo": "BALCAO",
  "formaPagamento": "DINHEIRO",
  "idUsuario": 1,
  "itens": [
    {
      "idProduto": 1,
      "quantidade": 2,
      "tipoItem": "CHEIO"
    }
  ]
}
```

**Regras** (Service):
- Campos obrigatórios: `tipo`, `formaPagamento`, `idUsuario` e ao menos 1 item com `idProduto` e `quantidade > 0` (CT-001).
- `tipo`: BALCAO ou ENTREGA. `formaPagamento`: DINHEIRO, PIX ou CARTAO, habilitada nas Configurações no momento da confirmação (CT-003); FIADO rejeitado (RGN-002).
- `total` e `precoUnitario` são calculados pelo Service (quantidade × preco_venda; acréscimo do cartão por unidade quando CARTAO e carga do produto é Gás - CT-002, CT-003-A). Não são aceitos no request.
- O acréscimo do cartão aplica-se somente a itens cujo produto tem carga Gás; itens de carga Água usam preço normal mesmo pagos com Cartão (RF-021-A, RGN-002).
- `quantidade <= qtdCheios` do produto, validada sob lock pessimista (CT-004; RNF-008).
- `idCliente` opcional nesta HU (sem devolução de vazio exige cliente - HU-010).

**Response 201** - venda criada:

```json
{
  "id": 42,
  "dataHora": "2026-08-20T10:15:00-03:00",
  "tipo": "BALCAO",
  "formaPagamento": "DINHEIRO",
  "taxaEntrega": null,
  "acrescimoCartao": null,
  "total": 230.00,
  "status": "ATIVA",
  "idUsuario": 1,
  "idCliente": null,
  "itens": [
    { "id": 1, "idProduto": 1, "nomeProduto": "Gás P13", "quantidade": 2, "precoUnitario": 115.00, "precoCasco": null, "tipoItem": "CHEIO" }
  ]
}
```

**Erros**:
- `400` - "Informe o produto, a quantidade, o tipo e a forma de pagamento." / "A quantidade deve ser maior que zero."
- `409` - "Estoque insuficiente para 3 unidade(s) de Gás P13. Disponível: 1." (CT-004, RF-031)
- `422` - "A forma de pagamento CARTAO não está habilitada nas Configurações." (CT-003)
- `404` - "Produto não encontrado." / "Usuário não encontrado."

---

## GET /api/vendas

Lista as vendas (histórico do dia e consultas). Suporte a filtro por data e status.

**Response 200** - lista de vendas (mesma estrutura do POST, com `status`).

---

## PUT /api/vendas/{id}/cancelar

Cancela a venda (comportamento da HU-020; contrato de rota mantido aqui para a base comum). Reverte estoque e caixa na mesma transação, registra rastro (quem, quando, motivo) e marca `status = CANCELADA` (RGN-007, RNF-007).

---

## GET /api/configuracoes

Formas de pagamento habilitadas, acréscimo do cartão e taxa de entrega (RF-052), usadas pela tela de lançamento.

**Response 200**:

```json
{
  "formasPagamento": ["DINHEIRO", "PIX", "CARTAO"],
  "acrescimoCartao": 5.00,
  "taxaEntrega": 10.00
}
```

---

## GET /api/estoque

Saldos por produto, para exibir disponibilidade e validar bloqueio na tela (RF-031). Estrutura em [api.md HU-006](../HU-006-Chegada-Caminhao/contracts/api.md).

---

## Contrato de erros (comum)

| Status | Código | Mensagem padrão (pt-BR) |
|---|---|---|
| 400 | VALIDACAO | "Informe o produto, a quantidade, o tipo e a forma de pagamento." etc. |
| 404 | NAO_ENCONTRADO | "Produto não encontrado." / "Usuário não encontrado." |
| 409 | ESTOQUE_INSUFICIENTE | "Estoque insuficiente para {n} unidade(s) de {produto}. Disponível: {m}." |
| 422 | CONFIGURACAO_INVALIDA | "A forma de pagamento {forma} não está habilitada nas Configurações." |
| 500 | ERRO_INTERNO | Sem exposição de detalhe técnico; validação nunca colapsa em erro de sistema (§XI.3) |
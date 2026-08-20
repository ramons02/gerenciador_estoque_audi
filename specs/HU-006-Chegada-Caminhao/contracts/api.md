# API Contract: HU-006 - Registro de Chegada de Caminhão (Entrada)

**HU**: HU-006 | **Feature**: Chegada de Caminhão (Carregamento) | **Date**: 2026-08-20 | **Spec**: [spec.md](../spec.md)
**Requisitos vinculados**: RF-010, RF-011, RF-012, RDN-003, RDN-009, RNF-005, RNF-008
**Data model**: [data-model.md](../data-model.md)

Módulo REST `carregamentos` (pacote `com.gerenciador.estoque.business.carregamento`). Todas as respostas de erro são JSON estruturado `{ "codigo": "...", "mensagem": "..." }` em pt-BR (CONVENTIONS §6).

---

## GET /api/carregamentos

Lista os carregamentos registrados (para conferência e para o Relatório de Carregamentos - CT-005).

**Response 200** - lista de carregamentos:

```json
[
  {
    "id": 1,
    "idFornecedor": 3,
    "nomeFornecedor": "Ultragaz",
    "dataHora": "2026-08-20T09:30:00-03:00",
    "itens": [
      {
        "id": 1,
        "idProduto": 1,
        "nomeProduto": "Gás P13",
        "qtdCheiosEntraram": 100,
        "qtdVaziosSairam": 20,
        "custoTotal": 3000.00,
        "valorUnitario": 30.00
      }
    ]
  }
]
```

**Erros**: 401 não autenticado; 403 sem papel de administrador.

---

## POST /api/carregamentos

Confirma a chegada de caminhão. Operação transacional: grava carregamento, itens, movimentações e atualiza saldos e custo médio em uma única transação (RNF-005).

**Request** (body JSON):

```json
{
  "idFornecedor": 3,
  "dataHora": "2026-08-20T09:30:00-03:00",
  "itens": [
    {
      "idProduto": 1,
      "qtdCheiosEntraram": 100,
      "qtdVaziosSairam": 20,
      "custoTotal": 3000.00
    }
  ]
}
```

**Regras** (Service):
- `valorUnitario` é calculado pelo Service (`custoTotal ÷ qtdCheiosEntraram`, 2 casas) e não é aceito no request (CT-001).
- `qtdVaziosSairam` validada contra o saldo de vazios do pátio sob lock pessimista por produto (RDN-003, RNF-008). Limite exato permitido.
- `qtdCheiosEntraram > 0`, obrigatório; `custoTotal >= 0`.
- `dataHora` opcional: quando ausente, usa o horário do servidor (Brasília).

**Response 201** - carregamento criado (mesma estrutura do GET, incluindo `valorUnitario` calculado).

**Erros**:
- `400` - campos ausentes ou inválidos: "Informe o fornecedor." / "Informe ao menos um item com produto e quantidade de cheios." / "A quantidade de cheios deve ser maior que zero."
- `404` - "Fornecedor não encontrado." / "Produto não encontrado."
- `409` - conflito com estado atual do estoque: "Devolução de 15 vazios excede o saldo do pátio. Disponível: 10." (CT-002)

---

## GET /api/estoque

Saldos por produto (usado pela tela para conferir o efeito da chegada e pelo app na validação).

**Response 200**:

```json
[
  { "idProduto": 1, "nomeProduto": "Gás P13", "qtdCheios": 150, "qtdVazios": 40, "qtdEmRua": 12, "limiteMinimo": 30 }
]
```

---

## GET /api/produtos e GET /api/fornecedores

Leitura para preencher o formulário de chegada (RF-005, RF-001). `GET /api/produtos` inclui `custoMedio` e saldos; `GET /api/fornecedores` lista ativos.

---

## Contrato de erros (comum)

| Status | Código | Mensagem padrão (pt-BR) |
|---|---|---|
| 400 | VALIDACAO | "Informe o fornecedor." etc. (por campo) |
| 404 | NAO_ENCONTRADO | "Fornecedor não encontrado." / "Produto não encontrado." |
| 409 | ESTOQUE_INSUFICIENTE | "Devolução de {n} vazios excede o saldo do pátio. Disponível: {m}." |
| 500 | ERRO_INTERNO | Sem exposição de detalhe técnico; nunca colapsa validação em erro de sistema (§XI.3) |
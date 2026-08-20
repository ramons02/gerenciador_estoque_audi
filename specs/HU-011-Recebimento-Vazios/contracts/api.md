# Contract API: Recebimento de Vasilhames Vazios Avulsos

**HU**: HU-011 - Recebimento de Vasilhames Vazios Avulsos
**Módulo**: `com.gerenciador.estoque.devolucao` | **Base**: `/api/devolucoes`

## POST /api/devolucoes

Lançamento de recebimento de vazios avulsos (devolução fora de venda, RF-027).

### Request

```json
{
  "idProduto": 1,
  "quantidade": 3,
  "idCliente": null
}
```

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| idProduto | number | sim | Produto existente (RF-001) |
| quantidade | integer | sim | Maior que zero |
| idCliente | number | null | Opcional (CT-001); se informado, deve existir (RF-004) |

### Response 201 Created

```json
{
  "id": 101,
  "idProduto": 1,
  "produto": "Gás P13",
  "quantidade": 3,
  "idCliente": null,
  "cliente": null,
  "dataHora": "2026-08-20T14:30:00-03:00",
  "idUsuario": 2,
  "saldos": {
    "vazios": 18,
    "emRua": 7,
    "emRuaCliente": null
  }
}
```

| Campo | Descrição |
|---|---|
| id | id do lançamento em tab_devolucao_vazio |
| produto | nome do produto para exibição |
| dataHora | data/hora gerada pelo servidor (CT-004) |
| idUsuario | usuário autenticado responsável (CT-004) |
| saldos.vazios | pátio de vazios atualizado (+N, CT-002) |
| saldos.emRua | em rua global atualizado (-M quando houve baixa) |
| saldos.emRuaCliente | em rua do cliente após baixa, null quando sem cliente |

### Erros

| HTTP | codigo | mensagem (pt-BR) |
|---|---|---|
| 400 | VALIDACAO | "Informe o produto e a quantidade." |
| 400 | VALIDACAO | "A quantidade deve ser maior que zero." |
| 404 | PRODUTO_NAO_ENCONTRADO | "Produto não encontrado." |
| 404 | CLIENTE_NAO_ENCONTRADO | "Cliente não encontrado." |

Erros sempre estruturados em JSON, nunca texto puro (CONVENTIONS §6). Bloqueio de negócio nunca retorna 500 (Constituição §XI.3).

## GET /api/devolucoes

Lista lançamentos para auditoria (CT-004, RNF-007).

### Query params

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| inicio | date (YYYY-MM-DD) | nao | Data inicial do filtro |
| fim | date (YYYY-MM-DD) | nao | Data final do filtro |
| idProduto | number | nao | Filtro por produto |
| pagina | integer | nao | Paginação (padrão 1) |
| tamanho | integer | nao | Tamanho da página (padrão 50) |

### Response 200

```json
{
  "total": 1,
  "pagina": 1,
  "itens": [
    {
      "id": 101,
      "idProduto": 1,
      "produto": "Gás P13",
      "quantidade": 3,
      "idCliente": null,
      "cliente": null,
      "dataHora": "2026-08-20T14:30:00-03:00",
      "idUsuario": 2
    }
  ]
}
```

## GET /api/devolucoes/{id}

### Response 200

Mesma estrutura do item da listagem.

### Erros

| HTTP | codigo | mensagem (pt-BR) |
|---|---|---|
| 404 | DEVOLUCAO_NAO_ENCONTRADA | "Lançamento de recebimento não encontrado." |

## Notas de implementação

- `POST /api/devolucoes` é `@Transactional`: lançamento + movimentação ENTRADA_VAZIO + saldos na mesma transação (RNF-005, CONVENTIONS §5.1).
- Leitura do saldo em tab_estoque com lock `PESSIMISTIC_WRITE` por produto (RNF-008, CONVENTIONS §8).
- Baixa de comodato limitada ao saldo em rua do cliente (RDN-005): excedente entra só no pátio (Edge Case do spec).
- Movimentos nunca são apagados (RNF-007); GET existe para auditoria e prova do CT-004.
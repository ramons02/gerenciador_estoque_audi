# Contract API: Dashboard (Resumo do Dia)

**HU**: HU-019 - Dashboard (Resumo do Dia)
**Módulo**: `com.gerenciador.estoque.dashboard` | **Base**: `/api/dashboard`

Endpoint de LEITURA (RF-050 a RF-053). Alimenta a tela inicial do app com uma única resposta.

## GET /api/dashboard/resumo-dia

Resumo do dia atual: total faturado, totais por forma de pagamento, unidades vendidas e alertas de estoque baixo (CT-001 a CT-004).

### Response JSON 200

```json
{
  "data": "2026-08-20",
  "totalFaturado": 285.00,
  "totaisPorForma": {
    "dinheiro": 120.00,
    "pix": 90.00,
    "cartao": 75.00
  },
  "unidadesPorProduto": [
    {
      "produto": "Gás P13",
      "quantidade": 4
    },
    {
      "produto": "Água 20L",
      "quantidade": 2
    }
  ],
  "totalUnidades": 6,
  "alertasEstoque": [
    {
      "produto": "Gás P45",
      "qtdCheios": 3,
      "limiteMinimo": 5
    }
  ]
}
```

## Erros

| HTTP | codigo | mensagem (pt-BR) |
|---|---|---|
| 401 | NAO_AUTENTICADO | "Usuário não autenticado. Faça login para continuar." |
| 403 | SEM_PERMISSAO | "Acesso negado. Perfil administrador é necessário." |

## Notas de implementação

- Dia sem vendas: totais zerados, `unidadesPorProduto` vazio e `totalUnidades = 0`, sem erro (Edge Case do spec).
- Forma de pagamento sem movimentação no dia: valor zero no campo correspondente (Edge Case do spec).
- Vendas canceladas (status CANCELADA) não entram em nenhum total (FR-005, RGN-007).
- Alertas derivados do saldo materializado `tab_estoque.qtd_cheios <= limite_minimo` (RF-032, RF-053).
- Resposta em menos de 2 segundos (RNF-003).
- Erros sempre estruturados em JSON (CONVENTIONS §6).
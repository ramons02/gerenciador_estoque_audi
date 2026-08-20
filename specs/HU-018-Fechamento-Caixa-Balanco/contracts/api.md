# Contract API: Fechamento de Caixa e Balanço de Estoque

**HU**: HU-018 - Fechamento de Caixa e Balanço de Estoque
**Módulo**: `com.gerenciador.estoque.caixa` | **Base**: `/api/caixa`

O fechamento de caixa é ação de escrita (RGN-006); o balanço de estoque é relatório de leitura exportado pelo endpoint consolidado da HU-015.

## GET /api/caixa/fechamento?data=YYYY-MM-DD

Resumo do dia para a tela de fechamento: totais por forma de pagamento, status do fechamento e vendas pendentes de conciliação (RGN-006, CT-002).

### Response JSON 200

```json
{
  "data": "2026-08-20",
  "status": "ABERTO",
  "totais": {
    "dinheiro": 120.00,
    "pix": 90.00,
    "cartao": 75.00,
    "geral": 285.00
  },
  "vendasPendentes": [],
  "fechamentoExistente": false,
  "dataHoraFechamento": null,
  "usuarioFechamento": null
}
```

Quando há fechamento concluído: `status = FECHADO`, `fechamentoExistente = true`, `dataHoraFechamento` e `usuarioFechamento` preenchidos. Quando há vendas não conciliadas: `vendasPendentes` traz os identificadores e horários das vendas em aberto.

## POST /api/caixa/fechar

Conclui o fechamento do dia (RGN-006, CT-002). Recalcula os totais das vendas ATIVA do dia na transação e grava `tab_fechamento_caixa` com status FECHADO (data-model.md).

### Request JSON

```json
{
  "data": "2026-08-20"
}
```

### Response JSON 200

```json
{
  "data": "2026-08-20",
  "status": "FECHADO",
  "totais": {
    "dinheiro": 120.00,
    "pix": 90.00,
    "cartao": 75.00,
    "geral": 285.00
  },
  "dataHoraFechamento": "2026-08-20T19:05:00-03:00",
  "usuarioFechamento": "Ana (admin)"
}
```

## Exportação do Balanço (RF-044, CT-003)

O balanço de estoque (RF-043, CT-001) usa o endpoint da HU-015: `GET /api/relatorios/balanco?periodo=...` com `Accept: text/csv` ou `?formato=csv`. Colunas EXATAS (CONVENTIONS §10, RF-043):

```
Produto;Estoque Inicial;(+) Entradas;(-) Vendas;Estoque Final;Vazios em Pátio
Gás P13;8;50;48;10;16
```

## Erros

| HTTP | codigo | mensagem (pt-BR) |
|---|---|---|
| 400 | VALIDACAO | "Informe a data do fechamento." |
| 409 | CAIXA_JA_FECHADO | "O caixa deste dia já foi fechado." |
| 422 | VENDAS_PENDENTES | "Existem vendas do dia pendentes de conciliação. Concilie antes de fechar o caixa." (RGN-006) |
| 401 | NAO_AUTENTICADO | "Usuário não autenticado. Faça login para continuar." |
| 403 | SEM_PERMISSAO | "Acesso negado. Perfil administrador é necessário." |
| 400 | PERIODO_INVALIDO | "Período inválido. Use HOJE, 7DIAS, MES ou PERSONALIZADO." (balanço) |

## Notas de implementação

- O fechamento é transacional: cálculo dos totais e gravação do registro na mesma transação (CONVENTIONS §5.1, RGN-006).
- Vendas canceladas (status CANCELADA) não entram nos totais do dia (FR-005, RGN-007).
- Balanço em período sem movimentações: produtos cadastrados com valores zerados, sem erro (Edge Case do spec).
- Erros sempre estruturados em JSON (CONVENTIONS §6).
# Contract API: Relatório de Vendas (Diário/Mensal)

**HU**: HU-016 - Relatório de Vendas (Diário/Mensal)
**Módulo**: `com.gerenciador.estoque.venda` | **Base**: `/api/vendas`

Endpoints de LEITURA (RF-040, RF-041, RGN-008). O relatório em JSON para a tela fica no módulo `venda`; a exportação CSV reutiliza o contrato consolidado da HU-015 (`GET /api/relatorios/vendas`), sem duplicação.

## Parâmetros comuns

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| periodo | string | sim | HOJE, 7DIAS, MES ou PERSONALIZADO (RF-040, CT-002) |
| inicio | date (YYYY-MM-DD) | quando periodo=PERSONALIZADO | Data inicial do intervalo |
| fim | date (YYYY-MM-DD) | quando periodo=PERSONALIZADO | Data final do intervalo (>= inicio) |

## GET /api/vendas/relatorio

Relatório de vendas por período (RF-041, CT-001, CT-002). Consulta vendas `status = ATIVA` dentro do intervalo (FR-005, RGN-007).

### Response JSON 200

```json
{
  "periodo": "HOJE",
  "inicio": "2026-08-20",
  "fim": "2026-08-20",
  "geradoEm": "2026-08-20T14:30:00-03:00",
  "linhas": [
    {
      "dataHora": "2026-08-20T10:15:00-03:00",
      "produto": "Gás P13",
      "quantidade": 1,
      "valorUnitario": 50.00,
      "total": 50.00,
      "formaPagamento": "DINHEIRO",
      "tipo": "BALCAO"
    }
  ],
  "totalPeriodo": 50.00,
  "totaisPorForma": {
    "dinheiro": 50.00,
    "pix": 0.00,
    "cartao": 0.00
  }
}
```

## Exportação CSV

A exportação (RF-044, CT-003) usa o endpoint da HU-015: `GET /api/relatorios/vendas?periodo=...` com `Accept: text/csv` ou `?formato=csv`. Colunas EXATAS (CONVENTIONS §10, RF-041):

```
Data/Hora;Produto;Qtd;Valor Unitário;Total (R$);Forma de Pagamento;Tipo (Balcão/Entrega)
2026-08-20 10:15:00;Gás P13;1;50,00;50,00;Dinheiro;Balcão
```

## Erros

| HTTP | codigo | mensagem (pt-BR) |
|---|---|---|
| 400 | PERIODO_INVALIDO | "Período inválido. Use HOJE, 7DIAS, MES ou PERSONALIZADO." |
| 400 | VALIDACAO | "Informe as datas inicial e final para o período personalizado." |
| 400 | VALIDACAO | "A data final deve ser posterior ou igual à data inicial." (Edge Case do spec) |
| 401 | NAO_AUTENTICADO | "Usuário não autenticado. Faça login para continuar." |
| 403 | SEM_PERMISSAO | "Acesso negado. Perfil administrador é necessário." |

## Notas de implementação

- Período sem vendas retorna 200 com `linhas` vazias e `totalPeriodo = 0`, sem erro (Edge Case do spec, SC-001).
- Vendas canceladas (status CANCELADA) são excluídas da consulta e dos totais (FR-005, RGN-007).
- Total (R$) da linha reflete valores gravados na venda, com acréscimo de cartão (somente carga Gás, RF-021-A) e taxa de entrega embutidos (RF-022).
- Erros sempre estruturados em JSON (CONVENTIONS §6).
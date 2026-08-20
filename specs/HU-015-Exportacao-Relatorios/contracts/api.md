# Contract API: Exportação de Relatórios (Período Personalizado)

**HU**: HU-015 - Exportação de Relatórios (Período Personalizado)
**Módulo**: `com.gerenciador.estoque.relatorio` | **Base**: `/api/relatorios`

Endpoints de LEITURA (RF-040 a RF-044). Cada endpoint responde JSON por padrão (painel) e CSV quando o cliente envia `Accept: text/csv` ou `?formato=csv` (exportação).

## Parâmetros comuns

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| periodo | string | sim | HOJE, 7DIAS, MES ou PERSONALIZADO (RF-040, CT-001) |
| inicio | date (YYYY-MM-DD) | quando periodo=PERSONALIZADO | Data inicial do intervalo |
| fim | date (YYYY-MM-DD) | quando periodo=PERSONALIZADO | Data final do intervalo (>= inicio) |
| formato | string | nao | "csv" força exportação em CSV |

## GET /api/relatorios/vendas

Relatório de Vendas (RF-041).

### Response JSON 200

```json
{
  "periodo": "PERSONALIZADO",
  "inicio": "2026-08-13",
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
  "totalPeriodo": 50.00
}
```

### Response CSV 200 (Accept: text/csv ou ?formato=csv)

- `Content-Type: text/csv; charset=UTF-8`
- `Content-Disposition: attachment; filename="relatorio-vendas-2026-08-20.csv"`
- BOM UTF-8 no início do arquivo; separador ponto e vírgula.

Colunas EXATAS (CONVENTIONS §10, RF-041):

```
Data/Hora;Produto;Qtd;Valor Unitário;Total (R$);Forma de Pagamento;Tipo (Balcão/Entrega)
2026-08-20 10:15:00;Gás P13;1;50,00;50,00;Dinheiro;Balcão
```

## GET /api/relatorios/carregamentos

Relatório de Carregamentos (RF-042).

### Response JSON 200

```json
{
  "periodo": "MES",
  "inicio": "2026-08-01",
  "fim": "2026-08-20",
  "geradoEm": "2026-08-20T14:30:00-03:00",
  "linhas": [
    {
      "dataHora": "2026-08-15T08:00:00-03:00",
      "fornecedor": "Ultragaz",
      "produto": "Gás P13",
      "qtdCheiosEntraram": 50,
      "qtdVaziosSairam": 12,
      "custoTotal": 2500.00
    }
  ]
}
```

### Response CSV 200

Colunas EXATAS (CONVENTIONS §10, RF-042):

```
Data;Fornecedor;Produto;Qtd Cheios Entraram;Qtd Vazios Saíram;Custo Total
2026-08-15;Ultragaz;Gás P13;50;12;2500,00
```

## GET /api/relatorios/balanco

Fechamento de Caixa e Balanço de Estoque (RF-043).

### Response JSON 200

```json
{
  "periodo": "HOJE",
  "inicio": "2026-08-20",
  "fim": "2026-08-20",
  "geradoEm": "2026-08-20T14:30:00-03:00",
  "linhas": [
    {
      "produto": "Gás P13",
      "estoqueInicial": 8,
      "entradas": 50,
      "vendas": 48,
      "estoqueFinal": 10,
      "vaziosPatio": 16
    }
  ]
}
```

### Response CSV 200

Colunas EXATAS (CONVENTIONS §10, RF-043):

```
Produto;Estoque Inicial;(+) Entradas;(-) Vendas;Estoque Final;Vazios em Pátio
Gás P13;8;50;48;10;16
```

## Erros

| HTTP | codigo | mensagem (pt-BR) |
|---|---|---|
| 400 | PERIODO_INVALIDO | "Período inválido. Use HOJE, 7DIAS, MES ou PERSONALIZADO." |
| 400 | VALIDACAO | "Informe as datas inicial e final para o período personalizado." |
| 400 | VALIDACAO | "A data final deve ser posterior ou igual à data inicial." (Edge Case do spec) |
| 400 | FORMATO_INVALIDO | "Formato de exportação inválido. Use csv." |
| 401 | NAO_AUTENTICADO | "Usuário não autenticado. Faça login para continuar." |
| 403 | SEM_PERMISSAO | "Acesso negado. Perfil administrador é necessário." |

## Notas de implementação

- Período sem movimentações gera CSV/JSON válidos com cabeçalho e zero linhas (Edge Case do spec, SC-004).
- Vendas canceladas (status CANCELADA) são excluídas da consolidação (RGN-007).
- Caracteres acentuados preservados via UTF-8 com BOM (RNF-009, Edge Case do spec).
- Erros sempre estruturados em JSON (CONVENTIONS §6).
# Contract API: Relatório de Carregamentos (Entradas)

**HU**: HU-017 - Relatório de Carregamentos (Entradas)
**Módulo**: `com.gerenciador.estoque.carregamento` | **Base**: `/api/carregamentos`

Endpoints de LEITURA (RF-040, RF-042). O relatório em JSON para a tela fica no módulo `carregamento`; a exportação CSV reutiliza o contrato consolidado da HU-015 (`GET /api/relatorios/carregamentos`), sem duplicação.

## Parâmetros comuns

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| periodo | string | sim | HOJE, 7DIAS, MES ou PERSONALIZADO (RF-040, CT-002) |
| inicio | date (YYYY-MM-DD) | quando periodo=PERSONALIZADO | Data inicial do intervalo |
| fim | date (YYYY-MM-DD) | quando periodo=PERSONALIZADO | Data final do intervalo (>= inicio) |

## GET /api/carregamentos/relatorio

Relatório de carregamentos por período (RF-042, CT-001, CT-002). Consolida somente carregamentos confirmados (FR-004).

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

## Exportação CSV

A exportação (RF-044, CT-003) usa o endpoint da HU-015: `GET /api/relatorios/carregamentos?periodo=...` com `Accept: text/csv` ou `?formato=csv`. Colunas EXATAS (CONVENTIONS §10, RF-042):

```
Data;Fornecedor;Produto;Qtd Cheios Entraram;Qtd Vazios Saíram;Custo Total
2026-08-15;Ultragaz;Gás P13;50;12;2500,00
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

- Período sem carregamentos retorna 200 com `linhas` vazias, sem erro (Edge Case do spec, SC-001).
- Uma linha por produto dentro de cada carregamento (RF-042 tem Produto como coluna).
- A invariante `qtd_vazios_sairam <= saldo de vazios do pátio` é garantida na escrita da chegada (RDN-003); o relatório apenas reflete os valores gravados.
- Erros sempre estruturados em JSON (CONVENTIONS §6).
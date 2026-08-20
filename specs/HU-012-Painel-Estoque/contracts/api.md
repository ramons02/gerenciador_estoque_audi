# Contract API: Painel de Estoque em Tempo Real (Pátio)

**HU**: HU-012 - Painel de Estoque em Tempo Real (Pátio)
**Módulo**: `com.gerenciador.estoque.estoque` | **Base**: `/api/estoque`

Endpoints de LEITURA: não escrevem dados (RF-030). Os saldos são atualizados pelas features de venda, carregamento e devolução (RNF-005).

## GET /api/estoque

Saldos por produto: Cheios, Vazios (pátio) e Em rua (RF-030, CT-001), com flag de alerta de estoque baixo (RF-032, CT-003).

### Response 200

```json
{
  "geradoEm": "2026-08-20T14:30:00-03:00",
  "produtos": [
    {
      "idProduto": 1,
      "nome": "Gás P13",
      "qtdCheios": 10,
      "qtdVazios": 4,
      "qtdEmRua": 6,
      "limiteMinimo": 5,
      "alertaEstoqueBaixo": false
    },
    {
      "idProduto": 3,
      "nome": "Água Galão 20L",
      "qtdCheios": 2,
      "qtdVazios": 9,
      "qtdEmRua": 3,
      "limiteMinimo": 5,
      "alertaEstoqueBaixo": true
    }
  ]
}
```

| Campo | Descrição |
|---|---|
| idProduto | id em tab_produto |
| nome | nome do produto (carga + vasilhame) |
| qtdCheios | vasilhames cheios prontos para venda |
| qtdVazios | vasilhames vazios no pátio |
| qtdEmRua | vasilhames em rua (clientes) |
| limiteMinimo | limite mínimo configurado (RF-003), null quando não configurado |
| alertaEstoqueBaixo | true quando qtdCheios <= limiteMinimo e limiteMinimo não nulo (RF-032) |

Regras:

- Todo produto ativo aparece, inclusive com saldos zerados (Edge Case do spec).
- Produto sem limite mínimo configurado nunca tem alerta (Edge Case do spec).
- Saldo exatamente igual ao limite mínimo dispara alerta (RGN-004).

## GET /api/estoque/alertas

Produtos com estoque baixo (RF-032), consumido pelo painel (CT-003) e pelo dashboard (RF-053, HU-014).

### Response 200

```json
{
  "geradoEm": "2026-08-20T14:30:00-03:00",
  "alertas": [
    {
      "idProduto": 3,
      "nome": "Água Galão 20L",
      "qtdCheios": 2,
      "qtdVazios": 9,
      "qtdEmRua": 3,
      "limiteMinimo": 5
    }
  ]
}
```

| Campo | Descrição |
|---|---|
| alertas | lista de produtos em alerta (qtdCheios <= limiteMinimo, limite não nulo) |
| alertas[].qtdCheios | saldo atual, para exibição |
| alertas[].limiteMinimo | limite que disparou o alerta |

## Erros

| HTTP | codigo | mensagem (pt-BR) |
|---|---|---|
| 401 | NAO_AUTENTICADO | "Usuário não autenticado. Faça login para continuar." |
| 403 | SEM_PERMISSAO | "Acesso negado. Perfil administrador é necessário." |

Erros sempre estruturados em JSON (CONVENTIONS §6). Como é leitura, não há erro de negócio de estoque; indisponibilidade do banco retorna 503 estruturado, nunca HTML.

## Notas de implementação

- Consulta parte de `tab_produto` (ativo) com LEFT JOIN `tab_estoque` (Edge Case: produtos sem linha de saldo aparecem zerados).
- `alertaEstoqueBaixo` é calculada na API (fonte única da regra RF-032), não no app.
- Resposta em menos de 500 ms com histórico de 12 meses (RNF-010); sem soma de movimentações na leitura (research.md).
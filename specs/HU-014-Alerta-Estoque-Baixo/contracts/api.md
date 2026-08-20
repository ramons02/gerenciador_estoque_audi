# Contract API: Alerta de Estoque Baixo

**HU**: HU-014 - Alerta de Estoque Baixo
**Módulo**: `com.gerenciador.estoque.estoque` | **Base**: `/api/estoque`

Endpoint de LEITURA: não escreve dados. Consumido pelo dashboard (RF-053, CT-002) e pelo painel de estoque (HU-012, CT-002).

## GET /api/estoque/alertas

Produtos em alerta de estoque baixo (RF-032), com sugestão de reposição (RGN-009, CT-004).

### Query params

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| sem | string | nao | Resposta abreviada sem sugestaoReposicao quando informado "sem-sugestao" |

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
      "limiteMinimo": 5,
      "mediaVendasDiarias": 2.4,
      "sugestaoReposicao": 3
    },
    {
      "idProduto": 1,
      "nome": "Gás P13",
      "qtdCheios": 5,
      "qtdVazios": 4,
      "qtdEmRua": 6,
      "limiteMinimo": 5,
      "mediaVendasDiarias": 1.1,
      "sugestaoReposicao": 2
    }
  ]
}
```

| Campo | Descrição |
|---|---|
| idProduto | id em tab_produto |
| nome | nome do produto (carga + vasilhame) |
| qtdCheios | saldo de cheios atual (<= limiteMinimo) |
| qtdVazios | saldo de vazios (informação complementar) |
| qtdEmRua | saldo em rua (informação complementar) |
| limiteMinimo | limite mínimo configurado (RF-003) |
| mediaVendasDiarias | média de vendas diárias dos últimos 30 dias, uma casa decimal (RGN-009) |
| sugestaoReposicao | max(1, arredondaParaCima(mediaVendasDiarias)); sem histórico: max(1, limiteMinimo - qtdCheios) |

Regras:

- Entram no resultado apenas produtos com `qtd_cheios <= limiteMinimo` E `limiteMinimo` não nulo (RF-032, Edge Case).
- Saldo exatamente igual ao limite entra no resultado (RF-032, RGN-004).
- Produto sem histórico de vendas no período ainda retorna com `sugestaoReposicao` calculada pelo limite (Edge Case do spec).

## Erros

| HTTP | codigo | mensagem (pt-BR) |
|---|---|---|
| 401 | NAO_AUTENTICADO | "Usuário não autenticado. Faça login para continuar." |
| 403 | SEM_PERMISSAO | "Acesso negado. Perfil administrador é necessário." |

Erros sempre estruturados em JSON (CONVENTIONS §6). Leitura pura, sem erro de negócio de estoque; indisponibilidade de banco retorna 503 estruturado.

## Notas de implementação

- `mediaVendasDiarias` e `sugestaoReposicao` são calculadas na API (regra centralizada, CONVENTIONS §12), nunca no app.
- O alerta é derivado do saldo (research.md): persiste enquanto a condição valer e some quando uma entrada eleva o saldo acima do limite (RGN-004, CT-003), sem estado persistido.
- O dashboard e o painel usam este mesmo endpoint (fonte única, CT-002).
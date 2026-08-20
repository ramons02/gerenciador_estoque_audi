# Contract UI: Alerta de Estoque Baixo

**HU**: HU-014 - Alerta de Estoque Baixo
**Tela**: Dashboard (Resumo do Dia) e Painel de Estoque | **Rotas**: /dashboard e /estoque

## Descrição

O alerta de estoque baixo (RF-032) é exibido em dois lugares: no dashboard (RF-053, CT-002) e no painel de estoque (CT-002). O mesmo componente `AlertaEstoqueBaixo` é reutilizado nos dois, consumindo `GET /api/estoque/alertas`.

## Componente de alerta (AlertaEstoqueBaixo)

Para cada produto do endpoint:

| Elemento | Comportamento |
|---|---|
| Linha/card do produto | Nome (ex.: Gás P13), saldo de cheios atual e limite mínimo |
| Status visual | Aviso em cor de destaque (amarelo/laranja) com ícone de alerta |
| Sugestão de reposição | "Sugestão de reposição: 3" (CT-004, RGN-009) |
| Sinalizador de baixo | "Estoque baixo" sempre legível (RNF-002) |

### No dashboard (RF-053, CT-002)

- Bloco "Alertas de estoque" no topo do resumo do dia.
- Lista compacta dos produtos em alerta com a sugestão de reposição.
- Reconsulta automática após venda ou entrada confirmada (refetch, sem recarga manual).

### No painel de estoque (CT-002)

- Produtos em alerta aparecem destacados (badge "Estoque baixo" e cor de destaque), conforme `contracts/ui.md` da HU-012.
- Tooltip ou linha adicional com a sugestão de reposição.

## Atualização em tempo real

- O hook `useAlertasEstoque` reconsulta após qualquer mutação confirmada na sessão (venda, carregamento, devolução) e em intervalo periódico curto (30 segundos).
- Venda que derruba o saldo para o limite durante o dia ativa o alerta imediatamente (Edge Case do spec).
- Entrada que eleva o saldo acima do limite desativa o alerta (RGN-004, CT-003).

## Regras de exibição

- Produto sem limite mínimo configurado nunca aparece no alerta (Edge Case do spec).
- Produto com saldo exatamente no limite aparece em alerta (RF-032).
- Produto recém-cadastrado sem histórico de vendas ainda exibe sugestão de reposição calculada pelo limite (Edge Case do spec).
- Mensagens de erro amigáveis em pt-BR, sem quebrar o restante da tela (Constituição §XI.3).
- Consumo da API via cliente HTTP tipado (estoqueApi.ts), nunca fetch solto (CONVENTIONS §7).

## Mensagens

- Nenhum alerta ativo: "Nenhum produto em estoque baixo."
- Erro de rede: "Não foi possível carregar os alertas. Verifique a conexão e tente novamente."
# Contract UI: Dashboard (Resumo do Dia)

**HU**: HU-019 - Dashboard (Resumo do Dia)
**Tela**: Tela inicial (Dashboard) | **Rota**: /

## Descrição

Tela inicial do sistema (RF-050 a RF-053, CONVENTIONS §7), aberta ao entrar no app. Mostra o resumo do dia de um relance: faturamento, totais por forma de pagamento, unidades vendidas e alertas de estoque baixo (CT-001 a CT-004).

## Layout

- Card principal: Total Faturado no Dia (R$) em destaque, números grandes (RF-050, CT-001, RNF-002).
- Cards de formas de pagamento: Dinheiro, PIX, Cartão (crédito + débito somados em um único valor) (RF-051, CT-002).
- Seção "Vasilhames vendidos no dia": tabela por produto + total geral (RF-052, CT-003).
- Bloco de alertas de estoque baixo: produtos com saldo de cheios no limite mínimo ou abaixo (RF-053, CT-004), visível enquanto ativo (RGN-004).

## Campos e ações

| Elemento | Comportamento |
|---|---|
| Card Total Faturado | Valor do dia em R$, vindo de GET /api/dashboard/resumo-dia |
| Cards Dinheiro/PIX/Cartão | Totais do dia por forma de pagamento; forma sem movimento exibe zero (Edge Case do spec) |
| Tabela de unidades vendidas | Soma por produto e total geral (RF-052) |
| Alerta de estoque baixo | Lista produtos com saldo no limite ou abaixo; some quando o produto é reabastecido (CT-004, RGN-004) |
| Atualização | O resumo é reconsultado após lançar venda ou cancelar (DashboardContext) |

## Regras de comportamento

- Dia sem vendas: todos os valores zerados, sem erro (Edge Case do spec).
- Produto sem vendas no dia não aparece nas unidades vendidas (Edge Case do spec).
- Vendas canceladas não entram nos totais (FR-005, RGN-007); ao cancelar uma venda do dia, o resumo é reconsultado.
- Alerta de estoque baixo permanece visível até nova entrada de caminhão (RGN-004).
- Os totais do dashboard coincidem com o relatório de vendas do mesmo dia (SC-002, RGN-008).
- Consumo via cliente HTTP tipado (dashboardApi.ts); nunca fetch solto (CONVENTIONS §7).

## Mensagens

- Erro de rede: "Não foi possível carregar o resumo do dia. Verifique a conexão e tente novamente."
- Erros de autenticação/permissão: mensagens do backend exibidas em alerta (pt-BR).
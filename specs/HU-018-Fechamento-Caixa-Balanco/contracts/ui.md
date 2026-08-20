# Contract UI: Fechamento de Caixa e Balanço de Estoque

**HU**: HU-018 - Fechamento de Caixa e Balanço de Estoque
**Tela**: Fechamento de Caixa | **Rota**: /caixa/fechamento (balanço na seção /relatorios)

## Descrição

Tela de fechamento do dia (RGN-006, CT-002) com o resumo financeiro por forma de pagamento e a ação de encerrar o caixa. O balanço de estoque (RF-043, CT-001/CT-004) é exibido na seção Balanço do painel de relatórios, com o mesmo seletor de período das demais seções (HU-015).

## Layout - Fechamento de Caixa

- Data do dia em destaque; status do fechamento (ABERTO ou FECHADO).
- Cards de totais: Dinheiro, PIX, Cartão (crédito + débito somados) e Total Geral (RGN-008, RF-051).
- Lista de vendas pendentes de conciliação, quando houver (RGN-006).
- Botão "Fechar caixa" (desabilitado quando já FECHADO ou com venda em edição).

## Layout - Balanço de Estoque

- Seletor de período: Hoje, Últimos 7 dias, Mês Atual, Personalizado (RF-040, CT-002).
- Tabela com as colunas EXATAS de CONVENTIONS §10: Produto, Estoque Inicial, (+) Entradas, (-) Vendas, Estoque Final, Vazios em Pátio (RF-043, CT-001).
- Botão "Exportar CSV" (RF-044, CT-003).

## Campos e ações

| Elemento | Comportamento |
|---|---|
| Cards de totais | Vêm de GET /api/caixa/fechamento?data= (totais do dia) |
| Lista de vendas pendentes | Exibida quando o dia tem vendas em aberto; bloco de aviso |
| Botão "Fechar caixa" | Abre modal de confirmação; POST /api/caixa/fechar |
| Modal de confirmação | Confirma a ação; mostra os totais que serão gravados |
| Seletor de período (balanço) | 4 opções; reconsulta o balanço |
| Tabela do balanço | Colunas exatas de CONVENTIONS §10 (mesmas do CSV) |
| Botão "Exportar CSV" (balanço) | Baixa via GET /api/relatorios/balanco com Accept: text/csv (HU-015) |

## Validações e bloqueios

- Fechamento com vendas pendentes: botão desabilitado e aviso "Existem vendas do dia pendentes de conciliação. Concilie antes de fechar o caixa." (RGN-006, CT-002).
- Dia já fechado: botão desabilitado e aviso "O caixa deste dia já foi fechado."; data/hora e usuário do fechamento exibidos.
- Venda em edição (formulário aberto): botão "Fechar caixa" desabilitado até confirmar ou descartar (RGN-006).
- Personalizado sem datas ou com final anterior ao início: validações iguais às da HU-015.

## Regras de comportamento

- Vendas canceladas não entram nos totais do dia nem nas vendas (-) do balanço (FR-005, RGN-007).
- Período sem movimentações: balanço com produtos cadastrados e valores zerados, sem erro (Edge Case do spec).
- O estoque final do balanço do dia confere com o estoque em tempo real do pátio no mesmo instante (SC-003).
- O download usa o cliente HTTP tipado (caixaApi.ts/relatoriosApi.ts); nunca fetch solto (CONVENTIONS §7).

## Mensagens

- Sucesso do fechamento: "Caixa fechado com sucesso." + totais gravados.
- Erro de rede: "Não foi possível concluir a operação. Verifique a conexão e tente novamente."
- Erros de validação/permissão: mensagens do backend exibidas em alerta (pt-BR).
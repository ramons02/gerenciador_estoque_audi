# Contract UI: Cancelamento/Estorno de Venda

**HU**: HU-020 - Cancelamento/Estorno de Venda
**Tela**: Histórico de Vendas - detalhe da venda | **Rota**: /vendas/historico

## Descrição

No histórico de vendas, o administrador abre o detalhe de uma venda ATIVA e a cancela com confirmação e motivo (CT-001 a CT-003). Vendas canceladas ficam visíveis no histórico com status "cancelado" e rastro de auditoria (CT-002, CT-003).

## Layout

- Histórico de vendas com lista das vendas do período, cada uma com status visível (ATIVA ou CANCELADA).
- Detalhe da venda: itens (produto, quantidade, valor unitário), forma de pagamento, tipo (Balcão/Entrega), total, usuário do lançamento (RNF-006).
- Quando CANCELADA: badge "Cancelada", motivo, data/hora e usuário do cancelamento exibidos no detalhe (FR-004, CT-003).
- Botão "Cancelar venda" apenas para vendas ATIVA (FR-006).

## Campos e ações

| Elemento | Comportamento |
|---|---|
| Badge de status | ATIVA ou Cancelada (RNF-007, CT-002) |
| Botão "Cancelar venda" | Abre modal de confirmação; só para venda ATIVA |
| Modal de confirmação | Campo Motivo (obrigatório) + resumo da reversão esperada (itens/estoque e valor) |
| Confirmar cancelamento | PUT /api/vendas/{id}/cancelar; ao concluir, atualiza o histórico e o dashboard (FR-005) |
| Rastro no detalhe | Motivo, data/hora e usuário do cancelamento (FR-004, CT-003) |

## Validações e bloqueios

- Motivo vazio: "Informe o motivo do cancelamento." (bloqueia a confirmação, §XIII.2).
- Venda já cancelada: botão não aparece; tentativa de API retorna "Esta venda já está cancelada." (FR-006, Edge Case do spec).
- Venda inexistente: erro do backend exibido em alerta.
- Somente usuário com perfil administrador vê a ação de cancelar (Assumption do spec, RNF-006).

## Regras de comportamento

- O cancelamento reverte estoque e caixa automaticamente na mesma transação; nenhum ajuste manual é necessário (RGN-007, CT-001).
- A venda cancelada permanece no histórico com todos os dados originais (RNF-007, CT-002).
- Após cancelar, o dashboard do dia reconsulta e remove a venda dos totais (CT-004, FR-005).
- A reversão nunca gera estoque negativo; se houver inconsistência, o cancelamento falha com mensagem clara (RDN-005, Edge Case do spec).
- Consumo via cliente HTTP tipado (vendasApi.ts); nunca fetch solto (CONVENTIONS §7).

## Mensagens

- Sucesso: "Venda cancelada. Estoque e caixa revertidos." + resumo da reversão.
- Erro de rede: "Não foi possível cancelar a venda. Verifique a conexão e tente novamente."
- Erros de validação/permissão: mensagens do backend exibidas em alerta (pt-BR).
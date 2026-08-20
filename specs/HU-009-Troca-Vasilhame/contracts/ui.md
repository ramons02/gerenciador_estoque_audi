# UI Contract: HU-009 - Troca de Vasilhame (Venda Normal)

**HU**: HU-009 | **Feature**: Troca de Vasilhame | **Date**: 2026-08-20 | **Spec**: [spec.md](../spec.md)
**Requisitos vinculados**: RF-023, RF-025, RDN-004, RNF-001, RNF-005
**API**: [api.md](api.md)
**Base comum**: [ui.md HU-007](../HU-007-Lancamento-Venda/contracts/ui.md)

A HU-009 usa o mesmo formulário de lançamento da HU-007, com a opção de troca de vasilhame.

---

## Tela: Vendas (opção "Troca de vasilhame")

No `LancamentoVenda.tsx` (HU-007), por item do lançamento:

| Elemento | Comportamento |
|---|---|
| Opção "Troca de vasilhame" (checkbox/switch por item) | Marca o item como troca (cliente entrega 1 vazio e leva 1 cheio) (CT-001) |
| Indicador "Vazio recebido" | Mostra ao lado do item: "1 vazio por unidade retornará ao pátio" (informativo, sem custo) |
| Total | NÃO muda com a troca: o vazio recebido não altera o valor da venda (CT-003) |

**Validações e mensagens**:
- A troca segue o bloqueio de estoque da HU-007: quantidade acima do saldo de cheios bloqueia com "Estoque insuficiente para N unidade(s) de Gás P13. Disponível: M." antes de qualquer atualização (spec.md Edge Cases).
- A opção de troca não é obrigatória: itens sem a marcação são vendas avulsas da HU-007.

## Painel de Estoque (leitura)

O saldo de vazios do pátio (`GET /api/estoque`, campo `qtdVazios`) atualiza imediatamente após a confirmação da venda com troca, sem ação manual (CT-004). Exibição na tela de estoque (HU-012) ou ao lado do formulário.

## Fluxo por CT

| CT | Fluxo na UI |
|---|---|
| CT-001 | Marcar "Troca de vasilhame" em 1 unidade e confirmar: pátio de vazios +1 e cheios -1 (conferir no painel de estoque); com N unidades, -N cheios e +N vazios |
| CT-002 | Atomicidade: não há indicação visual de transação; a conferência é no dado (painel nunca mostra cheio baixado sem vazio adicionado). Falha simulada de conexão: o lançamento não aparece no histórico e o estoque permanece intacto |
| CT-003 | Total exibido não muda ao marcar a troca (ex.: 2 × 115,00 = R$ 230,00 com ou sem troca) |
| CT-004 | Após confirmar, consultar o painel de estoque: o vazio recebido já está somado ao saldo do pátio, sem nenhuma ação adicional |

## Nota de integração

- HU-010 (vasilhame novo) usa o mesmo formulário com opção oposta (sem devolução de vazio, cliente obrigatório, preço do casco). Troca e vasilhame novo são mutuamente exclusivos por item.
- O cancelamento da venda com troca (HU-020) reverte o pátio automaticamente; a tela não oferece ação manual de ajuste de vazios.
# UI Contract: HU-008 - Venda com Entrega (Taxa de Entrega)

**HU**: HU-008 | **Feature**: Venda com Entrega | **Date**: 2026-08-20 | **Spec**: [spec.md](../spec.md)
**Requisitos vinculados**: RF-020, RF-022, RF-052, RNF-001, RNF-002, RGN-001
**API**: [api.md](api.md)
**Base comum**: [ui.md HU-007](../HU-007-Lancamento-Venda/contracts/ui.md)

A HU-008 usa o mesmo formulário de lançamento da HU-007, com o comportamento de taxa no campo Tipo, e adiciona o campo da taxa na tela de Configurações.

---

## Tela: Vendas (campo Tipo = Entrega)

No `LancamentoVenda.tsx` (HU-007), quando o vendedor seleciona Tipo "Entrega":

| Elemento | Comportamento |
|---|---|
| Selector Tipo (Balcão / Entrega) | Ao marcar "Entrega", o total passa a incluir a taxa vigente de `GET /api/configuracoes` (CT-001) |
| Linha "Taxa de entrega" | Exibida abaixo do total, somente leitura, com o valor aplicado (ex.: "Taxa de entrega: R$ 10,00") |
| Total | Atualizado em tempo real: itens + taxa (e acréscimo do cartão quando Cartão e carga Gás, HU-007) |

**Validações e mensagens**:
- Entrega sem taxa configurada (chave ausente): botão desabilitado e aviso "A taxa de entrega não está configurada. Defina o valor em Configurações." (edge case; RF-022).
- Taxa zero: a linha exibe "Taxa de entrega: R$ 0,00" e o total segue sem acréscimo (edge case).
- Trocar para "Balcão": a taxa some do total imediatamente (CT-001; RF-022).
- Alteração da taxa depois de vendas do dia: o formulário passa a usar o novo valor apenas nas próximas confirmações; vendas já lançadas mostram o valor gravado no histórico (edge case).

## Tela: Configurações (campo da taxa)

Na `ConfiguracoesPage.tsx` (`src/features/configuracoes/`):

| Campo | Tipo | Validação | Comportamento |
|---|---|---|---|
| Taxa de entrega (R$) | Money | >= 0 | Default do `GET /api/configuracoes`; salva via `PUT /api/configuracoes` (CT-002; RF-052) |

**Mensagens**:
- Sucesso: "Configurações salvas com sucesso." (toast)
- Erro 400: "O valor da taxa de entrega deve ser maior ou igual a zero." (inline)
- Erro 403: "Operação restrita ao administrador." (tela não abre para vendedor - RNF-006)

## Fluxo por CT

| CT | Fluxo na UI |
|---|---|
| CT-001 | Tipo "Entrega" com taxa 10,00 e produto 115,00: total exibe R$ 125,00 e linha "Taxa de entrega: R$ 10,00"; mudar para Balcão: total exibe R$ 115,00 |
| CT-002 | Admin altera a taxa para 15,00 em Configurações e salva: próximo lançamento Entrega usa R$ 15,00 |
| CT-003 | Relatório de Vendas (HU-016) mostra Tipo "Entrega" com Total R$ 125,00 (taxa incluída) e "Balcão" sem taxa |

## Nota de integração

A taxa é aplicada no Service (fonte da verdade); a UI apenas exibe o valor vigente lido de `GET /api/configuracoes`. O campo `taxaEntrega` não é editável no lançamento (RF-022: automático).
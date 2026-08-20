# Data Model: HU-008 - Venda com Entrega (Taxa de Entrega)

**HU**: HU-008 | **Feature**: Venda com Entrega | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Requisitos vinculados**: RF-020, RF-022, RF-041, RF-052, RGN-001, RNF-005, RNF-007

Entidades tocadas pela feature (somente as que ela usa ou altera). Estrutura base de venda em [data-model HU-007](../HU-007-Lancamento-Venda/data-model.md); esta HU documenta apenas o que adiciona.

---

## tab_venda (alteração)

Campos já definidos na HU-007, com o acréscimo da HU-008:

| Campo | Tipo | Obrigatório | Observação |
|---|---|---|---|
| tipo | VARCHAR(20) | sim | BALCAO ou ENTREGA (RF-020); discriminação do relatório (CT-003) |
| taxa_entrega | NUMERIC(12,2) | quando tipo ENTREGA | Valor configurado aplicado na venda (RF-022); gravado para conferência histórica |
| total | NUMERIC(12,2) | sim | Soma dos itens + taxa_entrega (quando Entrega) + acréscimo do cartão (quando Cartão e carga do produto é Gás, HU-007) |

*Situação atual (V3/V4)*: `tab_venda` existe sem `taxa_entrega` e sem `acrescimo_cartao`. Migration V10+ adiciona as colunas ao cabeçalho da venda.

---

## tab_configuracao (leitura e escrita)

| Chave | Valor | Uso na HU-008 |
|---|---|---|
| taxa_entrega | NUMERIC(12,2) | Aplicada automaticamente em vendas ENTREGA (CT-001); editável pelo admin (CT-002) |

*Situação atual (V3/V9)*: chave `taxa_entrega` já inserida com valor 10.00; `PUT /api/configuracoes` atualiza o valor. Taxa nunca hardcoded no código (CONVENTIONS §8, RF-052).

---

## tab_venda_item (sem alteração)

Estrutura da HU-007; a taxa não é item, é atributo do cabeçalho da venda (valor por entrega, não por unidade - RGN-001).

---

## tab_movimentacao_estoque (sem alteração)

A venda com entrega movimenta estoque como qualquer venda (SAIDA_CHEIO por item, HU-007). A taxa afeta apenas o caixa (total), não as quantidades.

---

## Regras de validação derivadas dos requisitos

- `tipo = ENTREGA` exige chave `taxa_entrega` presente em `tab_configuracao`; ausência bloqueia a confirmação com mensagem "A taxa de entrega não está configurada. Defina o valor em Configurações." (spec.md Edge Cases; RF-022).
- Taxa zero é válida e não acresce nada ao total (spec.md Edge Cases; RGN-001).
- Vendas BALCAO nunca aplicam a taxa (CT-001, CT-002; RF-022).
- Alteração da taxa afeta apenas vendas confirmadas depois da mudança; vendas anteriores mantêm o valor gravado em `tab_venda.taxa_entrega` (spec.md Edge Cases).
- `total = soma(itens) + taxa_entrega` (quando ENTREGA), persistido na mesma transação do estoque (RNF-005).
- Regra de negócio sem RF correspondente não pode ser codificada; se necessária, registrar em REQUISITOS.md na mesma entrega (Constituição §X.2).

## Transições de estado

- `tab_venda.status`: ATIVA → CANCELADA (RNF-007, RGN-007), comportamento da HU-020. O cancelamento reverte o total (incluindo a taxa) e o estoque na mesma transação, mantendo o rastro.
- `tab_configuracao.taxa_entrega`: valor vigente trocado pelo `PUT`; histórico de valores anteriores preservado nas vendas que os aplicaram.
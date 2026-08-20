# Data Model: HU-009 - Troca de Vasilhame (Venda Normal)

**HU**: HU-009 | **Feature**: Troca de Vasilhame | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Requisitos vinculados**: RF-023, RF-025, RDN-003, RDN-004, RDN-005, RNF-005, RNF-007, RNF-008

Entidades tocadas pela feature (somente as que ela usa ou altera). Estrutura base de venda em [data-model HU-007](../HU-007-Lancamento-Venda/data-model.md); esta HU documenta apenas o que a troca adiciona.

---

## tab_venda_item (uso)

Item com `tipoItem = CHEIO` marcado como troca (campo/flag de indicação, canônico: `troca` no item ou no cabeçalho - decisão de implementação a registrar; este plano assume flag no request mapeada no item).

| Campo | Tipo | Obrigatório | Observação |
|---|---|---|---|
| id_produto | BIGINT | sim | FK → tab_produto(id) |
| quantidade | INTEGER | sim | N unidades vendidas com troca |
| preco_unitario | NUMERIC(12,2) | sim | Preço de venda; a troca não altera o preço (CT-003) |
| tipo_item | VARCHAR(20) | sim | CHEIO |

---

## tab_estoque (alteração dupla)

| Campo | Tipo | Obrigatório | Efeito na troca (por unidade) |
|---|---|---|---|
| qtd_cheios | INTEGER | sim | -1 (RF-025); nunca negativa (RDN-005) |
| qtd_vazios | INTEGER | sim | +1 no pátio (RF-025); nunca negativa (RDN-003) |
| qtd_em_rua | INTEGER | sim | Inalterada (troca não gera comodato - RDN-008) |

*Situação atual (V3)*: saldos em `tab_produto` (`estoque_cheios`, `estoque_vazios`); ver nota de conciliação em [data-model HU-006](../HU-006-Chegada-Caminhao/data-model.md).

---

## tab_movimentacao_estoque (uso)

A troca gera, no mesmo commit, dois registros por item (RDN-004):

| Registro | Tipo | Quantidade | id_referencia |
|---|---|---|---|
| 1 | SAIDA_CHEIO | N | id da venda |
| 2 | ENTRADA_VAZIO | N | id da venda |

Ambos com o mesmo `id_referencia`, permitindo reconstituir a operação completa e a reversão no cancelamento (RGN-007, HU-020). Modelo completo da tabela em [data-model HU-006](../HU-006-Chegada-Caminhao/data-model.md).

---

## tab_produto (leitura, lock)

Consultado com `@Lock(PESSIMISTIC_WRITE)` para validar cheios e serializar o pátio na transação (RNF-008; CONVENTIONS §5.3). Estrutura em [data-model HU-006](../HU-006-Chegada-Caminhao/data-model.md).

---

## Regras de validação derivadas dos requisitos

- `quantidade <= qtd_cheios` do produto, validada sob lock antes de qualquer atualização (RF-031, RDN-005; spec.md Edge Cases). Limite exato permitido.
- A troca é 1:1: 1 vazio adicionado ao pátio por unidade vendida (RF-023; spec.md Assumptions).
- Total da venda sem o vazio recebido: a troca não tem custo para o cliente (CT-003, FR-003).
- O saldo de vazios do pátio reflete o recebido imediatamente após a confirmação, sem ação manual (CT-004; RDN-003).
- Venda com troca de múltiplas unidades: N cheios baixados e N vazios adicionados na mesma operação (spec.md Edge Cases).
- Pátio com saldo prévio: o vazio recebido apenas incrementa o saldo (spec.md Edge Cases).
- Regra de negócio sem RF correspondente não pode ser codificada; se necessária, registrar em REQUISITOS.md na mesma entrega (Constituição §X.2).

## Transições de estado

- Saldos de `tab_estoque`: `qtd_cheios` -N e `qtd_vazios` +N aplicados juntos ou nenhum (CT-002; RNF-005; RDN-004). Sem estado parcial visível: baixar cheio sem adicionar vazio é defeito grave (§III.3).
- `tab_venda.status`: ATIVA → CANCELADA (RNF-007, RGN-007), comportamento da HU-020. A reversão da troca é o espelho: +N cheios e -N vazios, com novo rastro de movimentação.
- Vasilhame trocado: o vazio recebido entra no estado Vazio (pátio); o cheio vendido sai do estado Cheio. Todo vasilhame permanece em exatamente um estado (RDN-002).
# Contract UI: Bloqueio de Venda sem Estoque

**HU**: HU-013 - Bloqueio de Venda sem Estoque
**Tela**: Lançamento de Venda | **Rota**: /venda (lançamento rápido)

## Descrição

Tela de lançamento rápido de venda (RNF-001) que valida o estoque de cheios no momento da confirmação (RF-031). Quando a quantidade excede o saldo, a venda é bloqueada com mensagem clara e o estoque não é baixado (CT-001).

## Campos

| Campo | Tipo | Obrigatório | Comportamento |
|---|---|---|---|
| Produto | select com busca | sim | Lista produtos ativos com saldo de cheios visível |
| Quantidade | input numérico grande | sim | Inteiros positivos; validação inline "maior que zero" |
| Tipo de operação | seletor | sim | NORMAL, TROCA, VASILHAME_NOVO, AVULSA (CT-003) |
| Tipo de venda | seletor | sim | Balcão ou Entrega (RF-020) |
| Forma de pagamento | seletor | sim | Dinheiro, PIX, Cartão (RF-021); nunca Fiado (RGN-002) |
| Confirmar | botão primário | - | Envia POST /api/vendas |

## Fluxo

1. Abrir a tela de Lançamento de Venda.
2. Adicionar itens (produto + quantidade + tipo de operação).
3. Selecionar tipo de venda e forma de pagamento.
4. Clicar em "Confirmar venda".

## Bloqueio por estoque insuficiente (CT-001)

**Dado** produto com saldo de cheios 1,
**Quando** o vendedor informa quantidade 3 e confirma,
**Então**:

- A venda não é confirmada.
- O erro inline no item exibe: "Estoque insuficiente para 3 unidade(s) de Gás P13. Disponível: 1." (mensagem do backend, CONVENTIONS §6).
- O estoque não é baixado (verificação no banco, `data-model.md`).

Quantidade igual ao saldo (ex.: 1) é permitida (Edge Case do spec).

## Validações inline (antes do POST)

- Quantidade vazia ou zero: "Informe uma quantidade maior que zero."
- Sem itens: "Informe ao menos um item de venda."
- Forma de pagamento ausente: "Selecione a forma de pagamento."

## Regras de comportamento

- O bloqueio vale para todos os fluxos: balcão, entrega, troca e vasilhame novo (CT-003).
- O bloqueio considera apenas cheios; falta de vazios ou de em rua nunca bloqueia (Edge Case do spec).
- Erro de bloqueio é exibido como erro de negócio (alerta/inline destacado), nunca como "sistema indisponível" (Constituição §XI.3).
- Após o cancelamento de uma venda (tela de histórico), o saldo liberado permite nova venda (Edge Case do spec).
- Consumo da API via cliente HTTP tipado (vendasApi.ts), nunca fetch solto (CONVENTIONS §7).

## Mensagens

- Sucesso: "Venda confirmada." com resumo do total.
- Bloqueio: mensagem do 409 ESTOQUE_INSUFICIENTE exibida destacada.
- Erro de rede: "Não foi possível confirmar a venda. Verifique a conexão e tente novamente."
# Contract UI: Recebimento de Vasilhames Vazios Avulsos

**HU**: HU-011 - Recebimento de Vasilhames Vazios Avulsos
**Tela**: Recebimento de Vazios (devolução avulsa) | **Rota**: /recebimento-vazios (acesso por menu "Pátio")

## Descrição

Tela para o vendedor registrar devoluções de vazios fora de venda (RF-027). Lançamento rápido, com números grandes (RNF-002) e o mínimo de passos (RNF-001).

## Campos

| Campo | Tipo | Obrigatório | Comportamento |
|---|---|---|---|
| Produto | select com busca (ProdutoSelect) | sim | Lista produtos ativos; exibe "carga + vasilhame" (ex.: Gás P13) |
| Quantidade | input numérico grande | sim | Apenas inteiros positivos; teclado numérico em tablet |
| Cliente (opcional) | select com busca (ClienteSelect) + botão "Limpar" | nao | Busca por nome; vazio significa devolução sem identificação (CT-001) |
| Confirmar | botão primário | - | Confirma o lançamento (POST /api/devolucoes) |

## Fluxo

1. Abrir a tela "Recebimento de Vazios".
2. Selecionar produto, informar quantidade e, opcionalmente, o cliente.
3. Clicar em "Confirmar recebimento".
4. O sistema exibe o resumo da confirmação com os saldos atualizados (ver Resumo).

## Validações inline (antes do POST)

- Produto não selecionado: "Informe o produto." (desabilita o botão Confirmar).
- Quantidade vazia ou zero: "Informe uma quantidade maior que zero."
- Cliente informado e não encontrado na busca: não permite seleção inexistente.

## Resumo após confirmação

Após o 201, exibir cartão de confirmação com:

- "3 x Gás P13 recebido(s) como vazio."
- "Pátio de vazios: 18" (+N, CT-002).
- "Em rua do cliente: 2" (somente quando houve baixa de comodato, RF-028, CT-003).
- Data/hora e usuário do lançamento (CT-004).

## Mensagens de erro (do backend, exibidas em alerta amigável)

- "Produto não encontrado."
- "Cliente não encontrado."
- "A quantidade deve ser maior que zero."

## Regras de comportamento

- Cliente é opcional: o formulário funciona sem ele e não bloqueia (CT-001).
- A devolução nunca altera o estoque de cheios (Edge Case do spec).
- Quando o cliente devolve mais do que possui em rua, o excedente entra no pátio e o comodato baixa até zero, sem erro (Edge Case do spec, RDN-005).
- Após confirmar, o formulário limpa quantidade e mantém produto/cliente para lançamento em sequência (lançamento rápido, RNF-001).
- Consumo da API via cliente HTTP tipado (devolucoesApi.ts), nunca fetch solto (CONVENTIONS §7).
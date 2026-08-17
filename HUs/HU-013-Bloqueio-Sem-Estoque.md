# HU-013 — Bloqueio de Venda sem Estoque

**História:** Como vendedor, quero que o sistema bloqueie a venda quando não houver estoque cheio suficiente, para nunca vender o que não tenho.

## Critérios de Aceitação

- **CT-001** Venda com quantidade maior que o saldo de cheios é bloqueada com mensagem clara (RF-031).
- **CT-002** Vendas simultâneas não podem gerar estoque negativo (RNF-008).
- **CT-003** O bloqueio vale para qualquer fluxo (balcão, entrega, troca, vasilhame novo).

## Requisitos
- RF-031, RDN-005, RNF-008

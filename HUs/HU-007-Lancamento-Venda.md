# HU-007 — Lançamento Rápido de Venda (Balcão/Entrega)

**História:** Como vendedor, quero lançar uma venda rapidamente com produto, quantidade, tipo e forma de pagamento, para atender o cliente da portaria sem demora.

## Critérios de Aceitação

- **CT-001** O lançamento exige: produto, quantidade, tipo (Balcão/Entrega) e forma de pagamento.
- **CT-002** O total é calculado automaticamente (qtd × preço de venda).
- **CT-003** Formas de pagamento: Dinheiro, PIX e Cartão (Crédito/Débito como forma única). Apenas as formas habilitadas nas Configurações aparecem (RF-052).
- **CT-003-A** Venda paga com Cartão soma o acréscimo configurado (R$ por unidade) ao preço; Dinheiro e PIX usam o preço normal (RF-021-A).
- **CT-004** Venda com quantidade maior que o estoque de cheios é bloqueada (RF-031).
- **CT-005** O lançamento completo deve ser executável em poucos segundos (RNF-001).
- **CT-006** A venda registra data/hora e usuário responsável.

## Requisitos
- RF-020, RF-021, RF-031, RNF-001, RNF-006, RGN-002

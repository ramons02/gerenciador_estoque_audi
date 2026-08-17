# HU-009 — Troca de Vasilhame (Venda Normal)

**História:** Como vendedor, quero registrar a venda com troca (cliente entrega 1 vazio e leva 1 cheio), para que o pátio de vazios seja atualizado automaticamente.

## Critérios de Aceitação

- **CT-001** Na venda com troca, o sistema baixa 1 cheio do estoque e adiciona 1 vazio ao pátio por unidade vendida (RF-025).
- **CT-002** A operação é atômica — nunca pode haver estado parcial (RNF-005).
- **CT-003** O vazio recebido do cliente não altera o total da venda (a troca não tem custo para o cliente).
- **CT-004** O saldo de vazios em pátio passa a refletir o recebido imediatamente.

## Requisitos
- RF-023, RF-025, RDN-004, RNF-005

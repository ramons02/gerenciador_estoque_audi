# Spec - HU-009 — Troca de Vasilhame (Venda Normal)

**HU de origem:** HU-009
**Status:** Em elaboração

## 1. Objetivo

Como vendedor, quero registrar a venda com troca (cliente entrega 1 vazio e leva 1 cheio), para que o pátio de vazios seja atualizado automaticamente.

## 2. Critérios de Aceitação

- **CT-001** Na venda com troca, o sistema baixa 1 cheio do estoque e adiciona 1 vazio ao pátio por unidade vendida (RF-025).
- **CT-002** A operação é atômica — nunca pode haver estado parcial (RNF-005).
- **CT-003** O vazio recebido do cliente não altera o total da venda (a troca não tem custo para o cliente).
- **CT-004** O saldo de vazios em pátio passa a refletir o recebido imediatamente.

## 3. Requisitos vinculados

- RF-023
- RF-025
- RDN-004
- RNF-005

## 4. Regras de negócio aplicáveis

- Seguir o vocabulário do domínio (Constituição §II).
- Não criar regra fora de requisitos/HUs (Constituição §X, §VI.5).

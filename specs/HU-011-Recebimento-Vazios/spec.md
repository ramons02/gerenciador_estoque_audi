# Spec - HU-011 — Recebimento de Vasilhames Vazios Avulsos

**HU de origem:** HU-011
**Status:** Em elaboração

## 1. Objetivo

Como vendedor, quero registrar devoluções de vazios fora de venda (cliente devolve vasilhame sem comprar), para controlar o saldo do pátio.

## 2. Critérios de Aceitação

- **CT-001** O sistema permite lançar recebimento de vazios avulsos por produto e cliente (opcional).
- **CT-002** Ao confirmar, o pátio de vazios +N.
- **CT-003** Se o cliente tinha vasilhames "em rua", a devolução baixa o comodato dele (RF-028).
- **CT-004** O lançamento registra data/hora e usuário.

## 3. Requisitos vinculados

- RF-027
- RF-028

## 4. Regras de negócio aplicáveis

- Seguir o vocabulário do domínio (Constituição §II).
- Não criar regra fora de requisitos/HUs (Constituição §X, §VI.5).

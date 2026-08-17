# Spec - HU-020 — Cancelamento/Estorno de Venda

**HU de origem:** HU-020
**Status:** Em elaboração

## 1. Objetivo

Como administrador, quero cancelar/estornar uma venda com reversão automática de estoque e caixa, para corrigir lançamentos errados mantendo o histórico.

## 2. Critérios de Aceitação

- **CT-001** O cancelamento reverte automaticamente o estoque (cheio volta, vazio sai do pátio) e o caixa (RGN-007).
- **CT-002** A venda cancelada permanece no histórico com status "cancelado" (RNF-007).
- **CT-003** O cancelamento registra data/hora e usuário responsável (RNF-006).
- **CT-004** Relatórios e dashboard excluem vendas canceladas dos totais.

## 3. Requisitos vinculados

- RF-043
- RGN-007
- RNF-006
- RNF-007

## 4. Regras de negócio aplicáveis

- Seguir o vocabulário do domínio (Constituição §II).
- Não criar regra fora de requisitos/HUs (Constituição §X, §VI.5).

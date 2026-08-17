# HU-020 — Cancelamento/Estorno de Venda

**História:** Como administrador, quero cancelar/estornar uma venda com reversão automática de estoque e caixa, para corrigir lançamentos errados mantendo o histórico.

## Critérios de Aceitação

- **CT-001** O cancelamento reverte automaticamente o estoque (cheio volta, vazio sai do pátio) e o caixa (RGN-007).
- **CT-002** A venda cancelada permanece no histórico com status "cancelado" (RNF-007).
- **CT-003** O cancelamento registra data/hora e usuário responsável (RNF-006).
- **CT-004** Relatórios e dashboard excluem vendas canceladas dos totais.

## Requisitos
- RF-043, RGN-007, RNF-006, RNF-007

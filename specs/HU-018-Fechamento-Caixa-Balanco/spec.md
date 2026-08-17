# Spec - HU-018 — Fechamento de Caixa e Balanço de Estoque

**HU de origem:** HU-018
**Status:** Em elaboração

## 1. Objetivo

Como administrador, quero gerar o balanço de estoque e fechamento de caixa por período, para saber o estoque final e conferir o caixa do dia.

## 2. Critérios de Aceitação

- **CT-001** O balanço lista por produto: Estoque Inicial, (+) Entradas, (−) Vendas, Estoque Final, Vazios em Pátio (RF-043).
- **CT-002** O fechamento de caixa do dia exige todas as vendas do dia conciliadas (RGN-006).
- **CT-003** O balanço pode ser exportado em Excel/CSV (RF-044).
- **CT-004** Estoque inicial do período é calculado com base nas movimentações anteriores ao período.

## 3. Requisitos vinculados

- RF-043
- RF-044
- RGN-006

## 4. Regras de negócio aplicáveis

- Seguir o vocabulário do domínio (Constituição §II).
- Não criar regra fora de requisitos/HUs (Constituição §X, §VI.5).

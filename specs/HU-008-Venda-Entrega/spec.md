# Spec - HU-008 — Venda com Entrega (Taxa de Entrega)

**HU de origem:** HU-008
**Status:** Em elaboração

## 1. Objetivo

Como vendedor, quero lançar vendas do tipo Entrega com taxa de entrega automática, para que o total cobrado inclua a entrega sem cálculo manual.

## 2. Critérios de Aceitação

- **CT-001** Ao selecionar tipo "Entrega", a taxa de entrega configurada é somada automaticamente ao total.
- **CT-002** O administrador pode configurar o valor da taxa de entrega (RGN-001).
- **CT-003** O relatório de vendas discrimina o tipo (Balcão/Entrega) e o total com taxa.

## 3. Requisitos vinculados

- RF-020
- RF-022
- RGN-001

## 4. Regras de negócio aplicáveis

- Seguir o vocabulário do domínio (Constituição §II).
- Não criar regra fora de requisitos/HUs (Constituição §X, §VI.5).

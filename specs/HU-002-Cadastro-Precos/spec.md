# Spec - HU-002 — Cadastro de Preços (Custo e Venda)

**HU de origem:** HU-002
**Status:** Em elaboração

## 1. Objetivo

Como administrador, quero cadastrar preço de custo e preço de venda por produto, para que as vendas e relatórios usem os valores corretos.

## 2. Critérios de Aceitação

- **CT-001** O cadastro do produto aceita preço de custo e preço de venda (R$).
- **CT-002** O sistema valida que o preço de venda não seja inferior ao preço de custo (RGN-005).
- **CT-003** O preço pode ser alterado a qualquer momento; o sistema mantém o preço da venda no momento em que ela é lançada (não recalcula vendas antigas).

## 3. Requisitos vinculados

- RF-002
- RGN-005

## 4. Regras de negócio aplicáveis

- Seguir o vocabulário do domínio (Constituição §II).
- Não criar regra fora de requisitos/HUs (Constituição §X, §VI.5).

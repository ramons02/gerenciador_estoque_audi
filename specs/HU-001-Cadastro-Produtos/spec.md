# Spec - HU-001 — Cadastro de Produtos (Carga + Vasilhame)

**HU de origem:** HU-001
**Status:** Em elaboração

## 1. Objetivo

Como administrador, quero cadastrar produtos compostos por Carga/Conteúdo e Vasilhame/Casco (ex.: Gás P13 = carga Gás + casco P13), para que o estoque e as vendas reflitam o item real vendido.

## 2. Critérios de Aceitação

- **CT-001** O cadastro permite informar a carga (ex.: Gás, Água) e o vasilhame (ex.: P13, P45, Galão 20L) do produto.
- **CT-002** O sistema lista o produto pelo nome combinado (ex.: "Gás P13").
- **CT-003** Não é possível cadastrar produto duplicado com a mesma combinação carga + vasilhame.
- **CT-004** O produto só pode ser editado; exclusão só é permitida se não houver movimentações vinculadas.

## 3. Requisitos vinculados

- RF-001
- RF-004
- RF-005

## 4. Regras de negócio aplicáveis

- Seguir o vocabulário do domínio (Constituição §II).
- Não criar regra fora de requisitos/HUs (Constituição §X, §VI.5).

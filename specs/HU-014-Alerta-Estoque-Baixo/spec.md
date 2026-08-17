# Spec - HU-014 — Alerta de Estoque Baixo

**HU de origem:** HU-014
**Status:** Em elaboração

## 1. Objetivo

Como administrador, quero ser avisado visualmente quando o estoque cheio de um produto estiver no limite mínimo, para repor a tempo.

## 2. Critérios de Aceitação

- **CT-001** O alerta aparece quando cheios ≤ limite mínimo configurado (RF-032).
- **CT-002** O alerta é visível no dashboard e no painel de estoque.
- **CT-003** O alerta persiste até o estoque subir acima do limite com nova entrada (RGN-004).
- **CT-004** O sistema sugere quantidade de reposição com base na média de vendas diárias (RGN-009).

## 3. Requisitos vinculados

- RF-032
- RF-053
- RGN-004
- RGN-009

## 4. Regras de negócio aplicáveis

- Seguir o vocabulário do domínio (Constituição §II).
- Não criar regra fora de requisitos/HUs (Constituição §X, §VI.5).

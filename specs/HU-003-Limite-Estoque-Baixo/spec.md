# Spec - HU-003 — Limite Mínimo de Estoque (Alerta)

**HU de origem:** HU-003
**Status:** Em elaboração

## 1. Objetivo

Como administrador, quero definir um limite mínimo de estoque de cheios por produto, para que o sistema me avise quando for hora de comprar mais.

## 2. Critérios de Aceitação

- **CT-001** O cadastro do produto aceita um limite mínimo de cheios.
- **CT-002** Ao atingir ou ficar abaixo do limite, o sistema emite alerta visual persistente (RF-032).
- **CT-003** O alerta só é dispensado após nova entrada de caminhão elevar o estoque acima do limite.

## 3. Requisitos vinculados

- RF-003
- RF-032
- RGN-004

## 4. Regras de negócio aplicáveis

- Seguir o vocabulário do domínio (Constituição §II).
- Não criar regra fora de requisitos/HUs (Constituição §X, §VI.5).

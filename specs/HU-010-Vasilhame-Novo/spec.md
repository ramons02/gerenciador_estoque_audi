# Spec - HU-010 — Venda de Vasilhame Novo (Casco + Carga)

**HU de origem:** HU-010
**Status:** Em elaboração

## 1. Objetivo

Como vendedor, quero vender vasilhame novo (cliente compra o casco + a carga), para registrar a saída do casco do estoque da loja.

## 2. Critérios de Aceitação

- **CT-001** O sistema permite marcar a venda como "vasilhame novo" (sem devolução de vazio).
- **CT-002** O preço do vasilhame novo é o preço do casco + preço da carga (RGN-010).
- **CT-003** Ao confirmar: baixa 1 cheio e registra 1 vasilhame "em rua" para o cliente (RF-026).
- **CT-004** O sistema solicita o cliente quando não há devolução de vazio, para o controle de comodato.

## 3. Requisitos vinculados

- RF-024
- RF-026
- RF-028
- RDN-008
- RGN-010

## 4. Regras de negócio aplicáveis

- Seguir o vocabulário do domínio (Constituição §II).
- Não criar regra fora de requisitos/HUs (Constituição §X, §VI.5).

# HU-006 — Registro de Chegada de Caminhão (Entrada)

**História:** Como administrador, quero registrar a chegada de caminhão com cheios recebidos, vazios devolvidos e custo total, para que o estoque e o caixa reflitam a compra da carga.

## Critérios de Aceitação

- **CT-001** O registro captura: fornecedor, produto, data, quantidade de cheios recebidos, quantidade de vazios devolvidos, custo total e valor unitário (calculado automaticamente = custo total ÷ cheios).
- **CT-002** A quantidade de vazios devolvidos não pode exceder o saldo de vazios do pátio (RDN-003).
- **CT-003** Ao confirmar: cheios +N e vazios −N automaticamente (RF-011).
- **CT-004** O sistema recalcula o custo médio do produto após a entrada (RF-012).
- **CT-005** O registro fica visível no Relatório de Carregamentos (RF-042).

## Requisitos
- RF-010, RF-011, RF-012, RDN-003, RDN-009

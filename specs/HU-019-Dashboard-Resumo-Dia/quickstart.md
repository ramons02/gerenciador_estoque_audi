# Quickstart: Dashboard (Resumo do Dia)

**HU**: HU-019 - Dashboard (Resumo do Dia)
**Fase**: 1 - Validação end-to-end
**Data**: 2026-08-20

## Pré-requisitos

- PostgreSQL rodando, banco `gerenciador_estoque` criado e migrations aplicadas (schema conforme `data-model.md`).
- API no ar na porta 8080.
- App no ar na porta 5173.
- Dados de apoio: vendas ATIVA de hoje em Dinheiro, PIX e Cartão em `tab_venda` (ex.: R$ 120 em Dinheiro, R$ 90 em PIX, R$ 75 em Cartão), com itens em `tab_venda_item`; uma venda CANCELADA de hoje (valida FR-005); um produto com `tab_estoque.qtd_cheios <= limite_minimo` (valida RF-053); produtos com limite configurado (RF-003).

## Setup

```bash
# API (em gerenciador_estoque_api/)
mvn spring-boot:run

# App (em gerenciador_estoque_app/)
npm install
npm run dev
```

## Cenários roteirizados

### CT-001 - Total Faturado no Dia

**Dado** vendas registradas no dia,
**Quando** o administrador abre a tela inicial (GET /api/dashboard/resumo-dia, contrato em `contracts/api.md`),
**Então** o dashboard exibe o Total Faturado no Dia em R$, igual à soma das vendas ATIVA do dia (RF-050).

### CT-002 - Totais por forma de pagamento

**Dado** vendas registradas no dia em Dinheiro, PIX e Cartão,
**Quando** o administrador abre a tela inicial,
**Então** o dashboard exibe o total por forma de pagamento, com Cartão representando crédito e débito somados em um único valor (RF-051).

**Dado** um dia sem vendas em uma forma de pagamento,
**Quando** o administrador abre a tela inicial,
**Então** essa forma é exibida com valor zero, sem erro (Edge Case do spec).

### CT-003 - Unidades vendidas no dia

**Dado** vendas do dia de múltiplos produtos,
**Quando** o administrador abre a tela inicial,
**Então** o dashboard exibe o total de vasilhames vendidos por produto e o total geral do dia (RF-052).

**Dado** um produto sem vendas no dia,
**Quando** o administrador abre a tela inicial,
**Então** o produto não aparece na lista de unidades vendidas (Edge Case do spec).

### CT-004 - Alertas de estoque baixo

**Dado** um produto com saldo de cheios no limite mínimo configurado ou abaixo,
**Quando** o administrador abre a tela inicial,
**Então** o dashboard exibe o alerta de estoque baixo desse produto (RF-053, RF-032).

**Dado** um alerta de estoque baixo ativo e uma nova entrada de caminhão,
**Quando** o administrador abre a tela inicial,
**Então** o alerta do produto reabastecido deixa de ser exibido (RGN-004).

## Verificação adicional

- Dia sem vendas: todos os totais zerados, sem erro (Edge Case do spec).
- Conferir que os totais do dashboard batem com o relatório de vendas do mesmo dia (SC-002, RGN-008) e com a soma direta em `tab_venda` (Constituição §IV.2).
- Vendas CANCELADA fora de todos os totais (FR-005, RGN-007); cancelar uma venda do dia e reconsultar o resumo para conferir a exclusão.
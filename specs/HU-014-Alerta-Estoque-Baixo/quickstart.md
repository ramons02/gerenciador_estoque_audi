# Quickstart: Alerta de Estoque Baixo

**HU**: HU-014 - Alerta de Estoque Baixo
**Fase**: 1 - Validação end-to-end
**Data**: 2026-08-20

## Pré-requisitos

- PostgreSQL rodando, banco `gerenciador_estoque` criado e migrations aplicadas (schema conforme `data-model.md`).
- API no ar na porta 8080.
- App no ar na porta 5173.
- Dados de apoio: produto com `limite_minimo` configurado (ex.: Gás P13, limite 5) e saldo de cheios variável em `tab_estoque`; vendas registradas em `tab_venda_item` para o cálculo da média (RGN-009).

## Setup

```bash
# API (em gerenciador_estoque_api/)
mvn spring-boot:run

# App (em gerenciador_estoque_app/)
npm install
npm run dev
```

## Cenários roteirizados

### CT-001 - Alerta visual no limite mínimo

**Dado** um produto com `limite_minimo` configurado em 5 e `qtd_cheios` igual a 5,
**Quando** o administrador consulta o estoque (GET /api/estoque/alertas, contrato em `contracts/api.md`),
**Então** o produto aparece no alerta (saldo igual ao limite dispara o alerta, RF-032 e Edge Case do spec).

**Dado** o mesmo produto com `qtd_cheios` em 7 (acima do limite),
**Quando** o administrador consulta,
**Então** o produto NÃO aparece no alerta (CT-001).

### CT-002 - Alerta visível no dashboard e no painel

**Dado** um produto em estoque baixo (qtd_cheios <= limite_minimo),
**Quando** o administrador abre o dashboard e o painel de estoque,
**Então** o alerta aparece nas duas telas, com os mesmos valores (fonte única do endpoint, RF-053, CT-002).

### CT-003 - Persistência do alerta até nova entrada

**Dado** um produto em alerta (qtd_cheios = 3, limite 5),
**Quando** nenhuma entrada é registrada,
**Então** o alerta permanece visível em consultas sucessivas (RGN-004).

**Dado** o mesmo produto em alerta,
**Quando** uma entrada de carregamento eleva `qtd_cheios` para 6 (acima do limite),
**Então** o alerta deixa de ser exibido na próxima consulta (RGN-004, CT-003).

**Dado** o mesmo produto em alerta,
**Quando** uma entrada eleva `qtd_cheios` apenas para 5 (igual ao limite),
**Então** o alerta permanece ativo (<= inclui igualdade, RF-032).

Verificação no banco (Constituição §IV):

```sql
SELECT qtd_cheios, limite_minimo FROM tab_estoque WHERE id_produto = 1;
```

### CT-004 - Sugestão de reposição pela média de vendas

**Dado** um produto em alerta com 24 unidades vendidas nos últimos 30 dias (média 0,8 por dia),
**Quando** o administrador visualiza o alerta,
**Então** a resposta contém `mediaVendasDiarias: 0.8` e `sugestaoReposicao: 1` (max(1, arredondaParaCima(0,8))).

**Dado** um produto em alerta recém-cadastrado, sem histórico de vendas no período,
**Quando** o administrador visualiza o alerta,
**Então** `sugestaoReposicao = max(1, limiteMinimo - qtdCheios)` é exibida, sem erro (Edge Case do spec).

## Verificação adicional

- Produto sem `limite_minimo` configurado nunca aparece no alerta (Edge Case do spec).
- Uma venda que derruba o saldo para o limite durante o dia ativa o alerta imediatamente, sem recarga manual (Edge Case do spec).
# Quickstart: Painel de Estoque em Tempo Real (Pátio)

**HU**: HU-012 - Painel de Estoque em Tempo Real (Pátio)
**Fase**: 1 - Validação end-to-end
**Data**: 2026-08-20

## Pré-requisitos

- PostgreSQL rodando, banco `gerenciador_estoque` criado e migrations aplicadas (schema conforme `data-model.md`).
- API no ar na porta 8080.
- App no ar na porta 5173.
- Dados de apoio: produtos cadastrados com linhas em `tab_estoque` (saldos variados) e um produto com `limite_minimo` acima do saldo de cheios (para CT-003).

## Setup

```bash
# API (em gerenciador_estoque_api/)
mvn spring-boot:run

# App (em gerenciador_estoque_app/)
npm install
npm run dev
```

## Cenários roteirizados

### CT-001 - Exibição por produto dos três estados

**Dado** produtos cadastrados com saldos registrados em `tab_estoque`,
**Quando** o administrador abre o painel de estoque (GET /api/estoque, contrato em `contracts/api.md`),
**Então** cada produto ativo exibe Cheios, Vazios e Em rua com os valores de `tab_estoque` (ver `data-model.md`).

**Dado** um produto recém-cadastrado sem linha em `tab_estoque`,
**Quando** o administrador abre o painel,
**Então** o produto aparece com saldos zerados (Edge Case do spec).

### CT-002 - Atualização imediata após movimentações

**Dado** o painel de estoque aberto,
**Quando** uma venda do Gás P13 é confirmada (baixa 1 cheio e, em troca, +1 vazio),
**Então** o painel reflete a mudança imediatamente, sem recarga manual (refetch automático após a mutação).

**Dado** o painel aberto,
**Quando** uma entrada de caminhão ou uma devolução avulsa é confirmada,
**Então** o painel atualiza os valores do produto afetado (CT-002).

Verificação no banco (Constituição §IV):

```sql
SELECT qtd_cheios, qtd_vazios, qtd_em_rua FROM tab_estoque WHERE id_produto = 1;
```

### CT-003 - Destaque de produtos com estoque baixo

**Dado** um produto com `limite_minimo` configurado e `qtd_cheios` igual ou abaixo do limite,
**Quando** o administrador visualiza o painel,
**Então** o produto aparece destacado com o badge "Estoque baixo" (flag `alertaEstoqueBaixo` vinda de GET /api/estoque, RF-032).

**Dado** um produto sem `limite_minimo` configurado,
**Quando** o administrador visualiza o painel,
**Então** nenhum badge é exibido para ele, mesmo com saldo baixo (Edge Case do spec).

## Verificação adicional

- GET /api/estoque/alertas retorna somente os produtos em alerta e é a mesma fonte do dashboard (RF-053), garantindo consistência com a HU-014.
- Medir o tempo de resposta da consulta: deve ficar abaixo de 500 ms (SC-002).
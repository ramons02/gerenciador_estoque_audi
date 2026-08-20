# Quickstart: Cancelamento/Estorno de Venda

**HU**: HU-020 - Cancelamento/Estorno de Venda
**Fase**: 1 - Validação end-to-end
**Data**: 2026-08-20

## Pré-requisitos

- PostgreSQL rodando, banco `gerenciador_estoque` criado e migrations aplicadas, incluindo `V<N>__adiciona_campos_cancelamento_tab_venda.sql` (schema conforme `data-model.md`).
- API no ar na porta 8080.
- App no ar na porta 5173.
- Dados de apoio: uma venda com troca registrada hoje (itens em `tab_venda_item`), uma venda de vasilhame novo registrada hoje (RF-024), saldos coerentes em `tab_estoque` (qtd_cheios, qtd_vazios, qtd_em_rua) e um usuário com perfil administrador autenticado (RNF-006).

## Setup

```bash
# API (em gerenciador_estoque_api/)
mvn spring-boot:run

# App (em gerenciador_estoque_app/)
npm install
npm run dev
```

## Cenários roteirizados

### CT-001 - Reversão automática de estoque e caixa

**Dado** uma venda com troca de vasilhame registrada (qtd_cheios = C, qtd_vazios = V antes da venda),
**Quando** o administrador cancela a venda (PUT /api/vendas/{id}/cancelar, contrato em `contracts/api.md`),
**Então** o estoque é revertido automaticamente: o cheio volta ao estoque de cheios e o vazio sai do pátio (qtd_cheios = C e qtd_vazios = V após o cancelamento), e o valor é estornado do caixa do dia (RGN-007).

**Dado** uma venda de vasilhame novo registrada (casco "em rua" do cliente),
**Quando** o administrador cancela a venda,
**Então** o casco "em rua" volta ao estoque de cheios (Edge Case do spec, RF-024/RF-026).

### CT-002 - Histórico com status "cancelado"

**Dado** uma venda cancelada,
**Quando** o administrador consulta o histórico de vendas,
**Então** a venda permanece no histórico com o status "cancelado" e todos os dados originais, sem ser apagada (RNF-007).

### CT-003 - Rastro de auditoria

**Dado** uma venda cancelada,
**Quando** o administrador consulta os detalhes do cancelamento,
**Então** o registro exibe a data/hora, o usuário responsável e o motivo do cancelamento (FR-004, RNF-006, Constituição §XIII.2).

### CT-004 - Vendas canceladas fora dos totais

**Dado** uma venda cancelada em um período,
**Quando** o administrador gera o relatório de vendas do período e abre o dashboard do dia,
**Então** a venda cancelada não aparece nas linhas nem nos totais do relatório, e o dashboard não inclui a venda cancelada nos totais (RGN-007).

## Verificação adicional

- Cancelamento de venda já cancelada: recusado com "Esta venda já está cancelada." (FR-006, Edge Case do spec).
- Motivo vazio: recusado com "Informe o motivo do cancelamento." (§XIII.2).
- Reversão nunca gera estoque negativo: conferir `tab_estoque` após o cancelamento (RDN-005); conferir também `tab_movimentacao_estoque` com os tipos ESTORNO_* gravados (RNF-007, RF-043).
- Fechamento de caixa do dia (HU-018) não inclui a venda cancelada nos totais (FR-005).
- Toda afirmação de dado conferida no banco (Constituição §IV.2).
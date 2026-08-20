# Quickstart: Bloqueio de Venda sem Estoque

**HU**: HU-013 - Bloqueio de Venda sem Estoque
**Fase**: 1 - Validação end-to-end
**Data**: 2026-08-20

## Pré-requisitos

- PostgreSQL rodando, banco `gerenciador_estoque` criado e migrations aplicadas (schema conforme `data-model.md`).
- API no ar na porta 8080.
- App no ar na porta 5173.
- Dados de apoio: produto com saldo de cheios conhecido (ex.: Gás P13 com `qtd_cheios = 1`), verificado em `tab_estoque`.

## Setup

```bash
# API (em gerenciador_estoque_api/)
mvn spring-boot:run

# App (em gerenciador_estoque_app/)
npm install
npm run dev
```

## Cenários roteirizados

### CT-001 - Venda bloqueada com mensagem clara

**Dado** um produto com saldo de cheios igual a 1 (`tab_estoque.qtd_cheios = 1`),
**Quando** o vendedor confirma uma venda de 3 unidades (POST /api/vendas, contrato em `contracts/api.md`),
**Então** o sistema retorna 409 `ESTOQUE_INSUFICIENTE` com a mensagem "Estoque insuficiente para 3 unidade(s) de Gás P13. Disponível: 1." e nenhuma linha nova em `tab_venda` ou `tab_movimentacao_estoque` é gravada (transação revertida, RNF-005).

**Dado** um produto com saldo de cheios igual a 1,
**Quando** o vendedor confirma uma venda de 1 unidade,
**Então** a venda é aceita (201) e o saldo passa a 0 (quantidade igual ao saldo é permitida, Edge Case do spec).

Verificação no banco (Constituição §IV):

```sql
SELECT qtd_cheios FROM tab_estoque WHERE id_produto = 1;
SELECT count(*) FROM tab_venda WHERE id = <id informado na resposta>;
```

### CT-002 - Vendas simultâneas não geram estoque negativo

**Dado** um saldo de cheios finito de 2 unidades de um produto,
**Quando** duas vendas simultâneas tentam consumir, em conjunto, 3 unidades (teste de integração com threads, `VendaConcorrenciaIntegrationTest`),
**Então** apenas a venda que cabe no saldo é aprovada, a outra recebe 409 ESTOQUE_INSUFICIENTE e `tab_estoque.qtd_cheios` nunca fica negativo (RDN-005, RNF-008).

Execução do teste:

```bash
mvn test -Dtest=VendaConcorrenciaIntegrationTest
```

### CT-003 - Bloqueio em todos os fluxos de venda

**Dado** um produto sem saldo de cheios suficiente,
**Quando** o vendedor tenta a venda por balcão, por entrega, com troca de vasilhame ou de vasilhame novo,
**Então** em todos os quatro fluxos (`tipoOperacao` NORMAL, TROCA, VASILHAME_NOVO, AVULSA) o sistema bloqueia com 409 ESTOQUE_INSUFICIENTE, sem exceção.

## Verificação adicional

- Após o cancelamento de uma venda (PUT /api/vendas/{id}/cancelamento), o saldo é revertido e uma nova venda da quantidade liberada é aceita (RGN-007, Edge Case do spec).
- O bloqueio por falta de vazios ou de em rua nunca ocorre: teste de venda NORMAL com saldo de cheios suficiente, mesmo com pátio de vazios zerado, é aceita (Edge Case do spec).
# Quickstart: Recebimento de Vasilhames Vazios Avulsos

**HU**: HU-011 - Recebimento de Vasilhames Vazios Avulsos
**Fase**: 1 - Validação end-to-end
**Data**: 2026-08-20

## Pré-requisitos

- PostgreSQL rodando, banco `gerenciador_estoque` criado e migrations Flyway aplicadas (schema conforme `data-model.md`).
- API no ar na porta 8080 (`gerenciador_estoque_api`).
- App no ar na porta 5173 (`gerenciador_estoque_app`).
- Dados de apoio: produto cadastrado (ex.: Gás P13) e, para o CT-003, cliente com vasilhames em rua registrados.

## Setup

```bash
# API (em gerenciador_estoque_api/)
mvn spring-boot:run

# App (em gerenciador_estoque_app/)
npm install
npm run dev
```

## Cenários roteirizados

### CT-001 - Lançamento por produto com cliente opcional

**Dado** um produto cadastrado e o vendedor logado na tela de Recebimento de Vazios,
**Quando** o vendedor informa produto e quantidade 3, sem selecionar cliente, e confirma,
**Então** o POST /api/devolucoes responde 201 com `idCliente: null` e a tela confirma o recebimento (contrato em `contracts/api.md` e `contracts/ui.md`).

**Dado** o mesmo cenário, **Quando** o vendedor informa produto, quantidade 2 e seleciona um cliente, **Então** o lançamento é registrado vinculado ao cliente (201 com `idCliente` preenchido).

### CT-002 - Pátio de vazios +N

**Dado** o pátio de vazios do Gás P13 com saldo 15,
**Quando** o vendedor confirma um recebimento avulso de 3 unidades,
**Então** `tab_estoque.qtd_vazios` do produto passa a 18 na mesma transação (ver `data-model.md`) e o resumo da tela exibe "Pátio de vazios: 18".

### CT-003 - Baixa do comodato do cliente

**Dado** o cliente "João da Silva" com 2 vasilhames Gás P13 em rua (linha em `tab_cliente_em_rua` com `qtd_em_rua = 2`),
**Quando** o vendedor lança a devolução de 2 vazios vinculada a esse cliente,
**Então** `qtd_em_rua` do cliente passa a 0 (baixa do comodato, RF-028) e o pátio incrementa em 2 (CT-003).

**Dado** um cliente sem vasilhames em rua,
**Quando** o vendedor lança a devolução vinculada a ele,
**Então** apenas o pátio é incrementado, sem alteração de em rua (CT-003).

**Dado** um cliente com apenas 2 em rua devolvendo 5 vazios,
**Quando** o vendedor confirma,
**Então** o comodato baixa até 0 (nunca negativo, RDN-005) e o pátio recebe os 5 (Edge Case do spec).

### CT-004 - Rastreabilidade com data/hora e usuário

**Dado** um lançamento de recebimento confirmado,
**Quando** o administrador consulta GET /api/devolucoes/{id} ou a listagem,
**Então** o registro exibe `dataHora` (gerada pelo servidor) e `idUsuario` do vendedor autenticado (CT-004, RNF-007).

## Verificação no banco (Constituição §IV)

```sql
SELECT qtd_vazios, qtd_em_rua FROM tab_estoque WHERE id_produto = 1;
SELECT tipo, quantidade, id_referencia FROM tab_movimentacao_estoque
  WHERE id_referencia = <id do lançamento>;
```

Esperado: `ENTRADA_VAZIO` com a quantidade lançada e, quando houver baixa, `SAIDA_EM_RUA` com a quantidade baixada, ambas apontando para o mesmo lançamento.
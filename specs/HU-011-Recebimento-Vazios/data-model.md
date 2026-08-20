# Data Model - Fase 1: Recebimento de Vasilhames Vazios Avulsos

**HU**: HU-011 - Recebimento de Vasilhames Vazios Avulsos
**Fase**: 1 - Modelo de dados
**Data**: 2026-08-20

## Visão geral

A feature não cria nova tabela de estoque: ela consome as tabelas canônicas do projeto e adiciona a tabela de lançamentos `tab_devolucao_vazio` (registro de auditoria, RNF-007). Toda atualização de saldo acontece na mesma transação do lançamento (Constituição §III.3, CONVENTIONS §5.1).

## Tabelas

### tab_devolucao_vazio (nova nesta feature)

Registro do lançamento de recebimento avulso de vazios.

| Campo | Tipo | Obrigatório | PK/FK | Descrição |
|---|---|---|---|---|
| id | bigint | sim | PK (seq_devolucao_vazio) | Identificador do lançamento |
| id_produto | bigint | sim | FK tab_produto.id | Produto devolvido (carga + vasilhame) |
| quantidade | integer | sim | - | Quantidade de vazios recebidos, sempre > 0 |
| id_cliente | bigint | nao | FK tab_cliente.id (null) | Cliente devolvedor, opcional (CT-001) |
| data_hora | timestamp | sim | - | Data/hora gerada pelo servidor (CT-004) |
| id_usuario | bigint | sim | FK tab_usuario.id | Usuário responsável pelo lançamento (CT-004) |

Sequência: `seq_devolucao_vazio` (CONVENTIONS §4).

Regras de validação (derivadas de RF-027, RF-028, CT-001 a CT-004):

- `quantidade` deve ser maior que zero: "A quantidade deve ser maior que zero."
- `id_produto` é obrigatório e deve existir em tab_produto (RF-001/RF-003).
- `id_cliente`, quando informado, deve existir em tab_cliente (RF-004); quando ausente, o lançamento é aceito (CT-001).
- `data_hora` e `id_usuario` sempre preenchidos pelo servidor, nunca aceitos do cliente (CT-004, RNF-007).
- Lançamento é imutável: movimentos nunca são apagados nem editados (RNF-007); correção exige novo lançamento reverso em HU futura.

### tab_estoque (alterada pela feature)

Saldo materializado por produto (já existente, base do projeto).

| Campo | Tipo | Obrigatório | PK/FK | Descrição |
|---|---|---|---|---|
| id_produto | bigint | sim | PK/FK tab_produto.id | Produto |
| qtd_cheios | integer | sim | - | Vasilhames cheios prontos para venda (nao alterado) |
| qtd_vazios | integer | sim | - | Vasilhames vazios no pátio (incrementado em N) |
| qtd_em_rua | integer | sim | - | Vasilhames em rua (decrementado em M, se houver baixa) |
| limite_minimo | integer | nao | - | Limite mínimo de cheios para alerta (RF-003, HU-014) |

Efeito da feature na transação: `qtd_vazios = qtd_vazios + N`; `qtd_em_rua = qtd_em_rua - M` onde M = min(N, saldo em rua do cliente) (RDN-005). Leitura do saldo com lock `PESSIMISTIC_WRITE` dentro da transação (CONVENTIONS §8, RNF-008).

### tab_movimentacao_estoque (usa a tabela)

Histórico canônico de movimentações (base do balanço RF-043 e do painel RF-030).

| Campo | Tipo | Obrigatório | PK/FK | Descrição |
|---|---|---|---|---|
| id | bigint | sim | PK (seq_movimentacao_estoque) | Identificador |
| id_produto | bigint | sim | FK tab_produto.id | Produto |
| tipo | varchar(20) | sim | - | ENTRADA_VAZIO / SAIDA_EM_RUA |
| quantidade | integer | sim | - | Quantidade do movimento (N ou M) |
| id_referencia | bigint | sim | FK tab_devolucao_vazio.id | Lançamento que originou o movimento |
| data_hora | timestamp | sim | - | Data/hora do movimento |

Movimentos gerados por esta feature:

- `ENTRADA_VAZIO` com quantidade N (vazios +N, CT-002).
- `SAIDA_EM_RUA` com quantidade M, somente quando M > 0 (baixa de comodato, RF-028, CT-003).

### tab_cliente_em_rua (nova nesta feature)

Controle de vasilhames em rua POR CLIENTE (RF-028), consumido para a baixa do comodato.

| Campo | Tipo | Obrigatório | PK/FK | Descrição |
|---|---|---|---|---|
| id_cliente | bigint | sim | PK/FK tab_cliente.id | Cliente |
| id_produto | bigint | sim | PK/FK tab_produto.id | Produto |
| qtd_em_rua | integer | sim | - | Vasilhames em rua do cliente, nunca negativa (RDN-005) |

Efeito da feature: `qtd_em_rua = qtd_em_rua - M`, com M limitado ao saldo existente. Cliente sem linha nessa tabela equivale a saldo zero (CT-003: apenas pátio é incrementado).

### Tabelas consultadas (sem alteração)

- `tab_produto`: nome/carga/vasilhame para exibição e validação (RF-001).
- `tab_cliente`: validação de existência quando informado (RF-004).
- `tab_usuario`: usuário autenticado responsável (CT-004, RNF-006).

## Regras de validação (resumo, derivadas de requisitos)

| Regra | Fonte |
|---|---|
| quantidade > 0; produto obrigatório e existente | RF-027, CT-001 |
| Cliente opcional; se informado, deve existir | RF-004, CT-001 |
| Pátio de vazios +N sempre; em rua do cliente -M limitado ao saldo | RDN-005, CT-002, CT-003, Edge Cases |
| Vasilhame devolvido entra como vazio, nunca como cheio | Edge Cases do spec, Constituição §II |
| Data/hora e usuário do servidor em todo lançamento | CT-004, RNF-007 |

## Transições de estado

A feature não possui transição de status: o lançamento é imutável após confirmado (RNF-007). O efeito nos estados do estoque (Constituição §III.1) é:

- Vazio (pátio): +N, sempre.
- Em rua (global e do cliente): -M quando M > 0.
- Cheio: não é alterado nesta feature (Edge Case: devolução avulsa nunca afeta cheios).
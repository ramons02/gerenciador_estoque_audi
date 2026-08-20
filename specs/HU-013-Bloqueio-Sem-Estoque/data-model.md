# Data Model - Fase 1: Bloqueio de Venda sem Estoque

**HU**: HU-013 - Bloqueio de Venda sem Estoque
**Fase**: 1 - Modelo de dados
**Data**: 2026-08-20

## Visão geral

A feature não cria tabela nova: implementa a regra de validação (RF-031) sobre `tab_estoque` no fluxo de venda (`tab_venda`, `tab_venda_item`), registrando movimentações em `tab_movimentacao_estoque` e ajustando `tab_cliente_em_rua` no vasilhame novo. Tudo na mesma transação (RNF-005).

## Tabelas

### tab_estoque (validada e alterada)

| Campo | Tipo | Obrigatório | PK/FK | Uso |
|---|---|---|---|---|
| id_produto | bigint | sim | PK/FK tab_produto.id | Produto da venda |
| qtd_cheios | integer | sim | - | Validado (quantidade <= qtd_cheios) e baixado (N) |
| qtd_vazios | integer | sim | - | +1 por troca (RF-023/RF-025) |
| qtd_em_rua | integer | sim | - | +1 por venda de vasilhame novo ou avulsa (RF-026) |
| limite_minimo | integer | nao | - | Não usado nesta feature (alerta, HU-014) |

Regra: leitura e baixa com lock `PESSIMISTIC_WRITE` na mesma transação (CONVENTIONS §8, RNF-008). Nunca negativo (RDN-005).

### tab_venda (usada e alterada)

| Campo | Tipo | Obrigatório | PK/FK | Descrição |
|---|---|---|---|---|
| id | bigint | sim | PK (seq_venda) | Identificador |
| tipo | varchar(20) | sim | - | BALCAO ou ENTREGA (RF-020) |
| forma_pagamento | varchar(20) | sim | - | DINHEIRO, PIX, CARTAO (RF-021); nunca FIADO (RGN-002) |
| id_cliente | bigint | nao | FK tab_cliente.id | Cliente (entrega/vasilhame novo) |
| data_hora | timestamp | sim | - | Data/hora da venda |
| id_usuario | bigint | sim | FK tab_usuario.id | Usuário responsável (RNF-006) |
| status | varchar(20) | sim | - | ATIVA, CANCELADA (RGN-007, RNF-007) |
| total | numeric(12,2) | sim | - | Total com taxa/acréscimo se aplicável (RF-022; RF-021-A somente carga Gás com Cartão) |

Transição de estado: ATIVA -> CANCELADA (reversão de estoque e caixa, RGN-007). Nenhuma venda é apagada (RNF-007).

### tab_venda_item (usada)

| Campo | Tipo | Obrigatório | PK/FK | Descrição |
|---|---|---|---|---|
| id | bigint | sim | PK (seq_venda_item) | Identificador |
| id_venda | bigint | sim | FK tab_venda.id | Venda |
| id_produto | bigint | sim | FK tab_produto.id | Produto |
| tipo_operacao | varchar(20) | sim | - | NORMAL, TROCA, VASILHAME_NOVO, AVULSA (CT-003) |
| quantidade | integer | sim | - | Quantidade solicitada, validada contra qtd_cheios |
| valor_unitario | numeric(12,2) | sim | - | Preço de venda (RF-002) |

### tab_movimentacao_estoque (usa)

Movimentos gerados por venda:

| Situação | tipo | quantidade | Efeito |
|---|---|---|---|
| Venda em geral (baixa cheio) | SAIDA_CHEIO | N | qtd_cheios -N (RF-025) |
| Venda com troca | ENTRADA_VAZIO | 1 por unidade | qtd_vazios +1 (RF-023/RF-025) |
| Venda sem devolução de vazio | ENTRADA_EM_RUA | 1 por unidade | qtd_em_rua +1 (RF-026) |

`id_referencia` aponta para o id de `tab_venda`. No cancelamento (RGN-007), são gerados os movimentos reversos (ENTRADA_CHEIO, SAIDA_VAZIO, SAIDA_EM_RUA).

### tab_cliente_em_rua (alterada no vasilhame novo)

| Campo | Tipo | Obrigatório | PK/FK | Descrição |
|---|---|---|---|---|
| id_cliente | bigint | sim | PK/FK tab_cliente.id | Cliente que leva o vasilhame |
| id_produto | bigint | sim | PK/FK tab_produto.id | Produto |
| qtd_em_rua | integer | sim | - | Incrementada no vasilhame novo (RF-026); nunca negativa (RDN-005) |

## Regras de validação (derivadas de requisitos)

| Regra | Fonte |
|---|---|
| quantidade > qtd_cheios bloqueia a venda com mensagem clara | RF-031, CT-001, CONVENTIONS §6 |
| quantidade <= qtd_cheios libera a venda | CT-001, Edge Case do spec |
| Nenhum fluxo (balcão, entrega, troca, vasilhame novo) foge da validação | CT-003 |
| Estoque nunca negativo, inclusive sob vendas simultâneas | RDN-005, RNF-008, CT-002 |
| Bloqueio considera apenas cheios; vazios e em rua não bloqueiam | Edge Case do spec, RF-031 |
| Cancelamento reverte estoque, caixa e movimentações, mantendo status cancelada | RGN-007, RNF-007 |

## Transições de estado

- Venda: ATIVA -> CANCELADA (RGN-007). Reversão atômica na mesma transação.
- Estoque: baixa parcial nunca ocorre - se qualquer item falha na validação, a transação inteira é revertida (RNF-005).
# Data Model - Fase 1: Painel de Estoque em Tempo Real (Pátio)

**HU**: HU-012 - Painel de Estoque em Tempo Real (Pátio)
**Fase**: 1 - Modelo de dados
**Data**: 2026-08-20

## Visão geral

Feature de LEITURA: não cria tabela nova e não altera dados. Consome o saldo materializado em `tab_estoque` e o cadastro em `tab_produto` (RF-030). A regra de destaque usa `limite_minimo` já persistido (RF-003, configurado pela HU-003).

## Tabelas

### tab_estoque (consultada)

Saldo materializado por produto, mantido pelas features de escrita na mesma transação de cada movimento (Constituição §III.3, CONVENTIONS §5.1).

| Campo | Tipo | Obrigatório | PK/FK | Uso no painel |
|---|---|---|---|---|
| id_produto | bigint | sim | PK/FK tab_produto.id | Chave de agrupamento |
| qtd_cheios | integer | sim | - | Exibido como "Cheios" (CT-001) |
| qtd_vazios | integer | sim | - | Exibido como "Vazios" (pátio) (CT-001) |
| qtd_em_rua | integer | sim | - | Exibido como "Em rua" (CT-001) |
| limite_minimo | integer | nao | - | Base do destaque: alerta quando qtd_cheios <= limite_minimo (RF-032) |

Regras de leitura derivadas dos requisitos:

- Alerta de estoque baixo = `qtd_cheios <= limite_minimo` (RF-032, RGN-004); inclui saldo exatamente igual ao limite (Edge Case da HU-014).
- `limite_minimo` nulo: sem alerta, sempre (Edge Case do spec da HU-012).
- Saldos nunca negativos (RDN-005), garantido pelas features de escrita.

### tab_produto (consultada)

Cadastro do produto (carga + vasilhame, RF-001), usado para exibir o nome e garantir que todo produto ativo apareça no painel, inclusive sem linha em tab_estoque (Edge Case: produto sem movimentação aparece com saldos zerados).

| Campo | Tipo | Obrigatório | PK/FK | Uso no painel |
|---|---|---|---|---|
| id | bigint | sim | PK (seq_produto) | Chave |
| nome | varchar | sim | - | Nome exibido (ex.: Gás P13) |
| carga | varchar | sim | - | Carga/conteúdo (Gás, Água) |
| vasilhame | varchar | sim | - | Casco (P13, P45, Galão 20L) |
| ativo | boolean | sim | - | Filtro: apenas produtos ativos |

## Consultas da feature (derivadas)

1. **Saldos por produto**: `tab_produto` (ativo) LEFT JOIN `tab_estoque`, retornando os três estados e a flag de alerta calculada (CT-001, CT-003).
2. **Alertas**: mesma origem, filtrando `qtd_cheios <= limite_minimo` com `limite_minimo` não nulo (RF-032, RF-053).

## Transições de estado

A feature não realiza transições de estado (é leitura pura). Os estados Cheio, Vazio e Em rua continuam mudando somente nas transações de venda (RF-025/RF-026), carregamento (RF-011) e devolução (RF-027), que refletem no painel pela reconsulta (CT-002).

## Consistência com outras features

- O destaque do CT-003 usa exatamente a mesma regra da HU-014 (`qtd_cheios <= limite_minimo`), garantida pelo endpoint compartilhado `GET /api/estoque/alertas`.
- A sugestão de reposição (RGN-009, HU-014) não é exibida no painel desta HU, apenas no alerta (contracts/ da HU-014).
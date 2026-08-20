# Modelo de Dados: Cadastro de Clientes

**HU**: HU-004 - Cadastro de Clientes
**Fase**: 1 (modelo de dados)
**Branch**: `HU-004-Cadastro-Clientes`

Tabela `tab_cliente` criada pela migration `V3__estoque_clientes_carregamentos_vendas.sql`;
coluna `documento` removida pela migration `V6__remover_documento_cliente.sql` (LGPD).
Tabela de apoio `tab_cliente_vasilhame` (mesma migration V3) para o controle de "em rua" por
cliente (RF-028). Todas as tabelas herdam as colunas de auditoria do `BaseModel`
(`criado_em`, `atualizado_em`, `ativo`).

---

## 1. tab_cliente

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGSERIAL | sim | PK | Sequência implícita |
| nome | VARCHAR(100) | sim | - | Nome do cliente (CT-001) |
| telefone | VARCHAR(20) | não | - | Buscável (CT-002) |
| endereco | VARCHAR(200) | não | - | Endereço (CT-001) |
| criado_em | TIMESTAMP | sim | - | Auditoria |
| atualizado_em | TIMESTAMP | sim | - | Auditoria |
| ativo | BOOLEAN | sim | - | Default TRUE |

**Nota:** o campo `documento` previsto na spec (CT-001, "documento opcional") foi removido
do schema pela migration V6 por decisão de LGPD; o cadastro captura nome, telefone e
endereço.

**Regras de validação (RF-004, CT-001):**
- `nome` é obrigatório; cliente sem nome não é salvo ("Informe o nome do cliente.", 422).
- `telefone` e `endereco` são opcionais; cliente sem documento é aceito (edge case da spec).

## 2. tab_cliente_vasilhame (vasilhames "em rua" por cliente)

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| id | BIGSERIAL | sim | PK | Sequência implícita |
| cliente_id | BIGINT | sim | FK tab_cliente(id) | Cliente com vasilhame em rua (RF-028) |
| vasilhame_id | BIGINT | sim | FK tab_vasilhame(id) | Vasilhame levado pelo cliente |
| quantidade | INTEGER | sim | - | Default 0; total em rua do par |
| criado_em | TIMESTAMP | sim | - | Auditoria |
| atualizado_em | TIMESTAMP | sim | - | Auditoria |
| ativo | BOOLEAN | sim | - | Default TRUE |

**Constraints:**
- `uk_cliente_vasilhame UNIQUE (cliente_id, vasilhame_id)`: um registro por combinação de
  cliente e vasilhame.

**Regras derivadas dos requisitos:**
- "Em rua" só aumenta quando o cliente leva um vasilhame sem devolver vazio equivalente
  (RDN-008), via venda de vasilhame novo (RF-024, HU-007).
- A baixa acontece quando o vazio é devolvido (RF-028, HU-011), reduzindo a quantidade.
- Todo vasilhame existente está em exatamente um estado (RDN-002): a contagem em rua por
  cliente é parte do estado "Em rua".

## 3. tab_venda (coluna de vínculo usada pela feature)

| Campo | Tipo | Obrigatório | PK/FK | Observação |
|---|---|---|---|---|
| cliente_id | BIGINT | não | FK tab_cliente(id) | Opcional; obrigatório na venda de vasilhame novo (CT-003) |

**Regra derivada (CT-003, RF-024/RF-026):**
- Na venda de vasilhame novo (`vasilhame_novo = true`), `cliente_id` é obrigatório: a venda
  é rejeitada sem cliente associado.
- Na venda com troca (RF-023) e na venda balcão comum, o cliente pode ser informado
  opcionalmente.

## 4. Transições de estado

**Cliente (tab_cliente):**

| De | Para | Gatilho | Regra |
|---|---|---|---|
| ativo = true | ativo = false | DELETE /api/clientes/{id} | Exclusão lógica; histórico de vendas e em rua preservado (RNF-007) |
| ativo = true | ativo = true (editado) | PUT /api/clientes/{id} | Edição livre do cadastro |

**Vasilhame em rua (tab_cliente_vasilhame):**

| De | Para | Gatilho | Regra |
|---|---|---|---|
| quantidade N | quantidade N+1 | Venda de vasilhame novo (HU-007) | RDN-008; vincula o vasilhame ao cliente |
| quantidade N | quantidade N-1 | Devolução de vazio (HU-011) | RF-028; baixa do vasilhame em rua |
| quantidade 0 | - | - | Registro mantido com quantidade 0; não é excluído |

## 5. Relacionamento entre entidades

```text
tab_cliente (1) --- (N) tab_cliente_vasilhame (N) --- (1) tab_vasilhame
tab_cliente (1) --- (N) tab_venda (cliente_id)
```

- `tab_venda.cliente_id` vincula a venda ao cliente; obrigatório em vasilhame novo (CT-003).
- `tab_cliente_vasilhame` é a fonte do estado "Em rua" por cliente (RF-028), consumida pelo
  painel de estoque (HU-012) para o saldo "em rua" por produto.

**Nota de consistência:** a busca por nome ou telefone (CT-002) usa consulta em
`tab_cliente`; a identificação do cliente na venda (CT-003) usa o mesmo registro.
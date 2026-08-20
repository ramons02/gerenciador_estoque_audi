# Validação End-to-End: Limite Mínimo de Estoque (Alerta)

**HU**: HU-003 - Limite Mínimo de Estoque (Alerta)
**Branch**: `HU-003-Limite-Estoque-Baixo`

Guia para provar os critérios de aceitação (CT-001 a CT-003) executando o sistema completo.
Detalhes de contratos em `contracts/api.md` e `contracts/ui.md`; estrutura do banco em
`data-model.md`.

## Pré-requisitos

| Item | Valor esperado |
|---|---|
| PostgreSQL | Rodando em `localhost:5432`, banco `gerenciador_estoque` (ou variáveis do `gerenciador_estoque_infra/env/dev.env`) |
| API | `http://localhost:8080` (`mvn spring-boot:run` no `gerenciador_estoque_api`) |
| App | `http://localhost:5173` (`npm run dev` no `gerenciador_estoque_app`, proxy `/api` para a API) |

**Setup:**

```bash
# 1. Banco
createdb gerenciador_estoque

# 2. API (aplica as migrations Flyway na subida)
cd ../gerenciador_estoque_api
mvn spring-boot:run

# 3. App
cd ../gerenciador_estoque_app
npm install
npm run dev
```

As migrations V1 e V8 criam os produtos "Gás P13" e "Água Galão 20L". Para os cenários de
alerta é necessário um estoque de cheios inicial: registrar um carregamento (HU-006) ou
ajustar o saldo diretamente no banco (ver passo de simulação abaixo).

## Preparação dos dados de teste

```sql
-- garantir produto com estoque cheio e definir limite (id do produto conforme o banco)
UPDATE tab_produto SET limite_minimo = 20, estoque_cheios = 30 WHERE id = 1;
```

## Cenários de prova (dado → quando → esperado)

### CT-001 - Cadastro aceita limite mínimo de cheios

1. Dado o cadastro do produto no app.
2. Quando o usuário informa o limite mínimo de 20 cheios.
3. Então o valor é salvo e exibido no cadastro.

**Prova complementar (API):** `PUT /api/produtos/{id}/limite-minimo` com `{"limiteMinimo":20}`
retorna 200 com o produto atualizado. Valor negativo retorna 422.

**Prova de dado (CONSTITUICAO §IV.1):** `SELECT limite_minimo FROM tab_produto WHERE id = {id}`
retorna 20.

### CT-002 - Alerta visual ao atingir ou ficar abaixo do limite

1. Dado um produto com limite mínimo de 20 cheios e saldo de cheios em 30.
2. Quando o saldo de cheios atinge 20 (via venda, HU-007) ou fica abaixo.
3. Então o sistema emite alerta visual de estoque baixo no painel/dashboard (RF-032) e o
   mantém visível ao navegar pelas telas.

**Simulação sem dependência de venda (HU-007):**

```sql
UPDATE tab_produto SET estoque_cheios = 20 WHERE id = 1;  -- igual ao limite: dispara
```

**Prova complementar (API):** a consulta de estoque (painel HU-012/dashboard HU-019) retorna
`estoqueBaixo: true` para o produto com `estoqueCheios <= limiteMinimo` e `limiteMinimo > 0`.

**Caso de borda:** produto com `limite_minimo = 0` e `estoque_cheios = 0` retorna
`estoqueBaixo: false` (limite não definido não gera alerta).

### CT-003 - Dispensa somente por entrada de caminhão

1. Dado um alerta de estoque baixo ativo (cheios em 15, limite 20).
2. Quando uma entrada de caminhão eleva o estoque de cheios acima do limite (via HU-006).
3. Então o alerta é dispensado.
4. Quando uma entrada de caminhão não eleva o estoque acima do limite, então o alerta
   permanece ativo.

**Simulação sem dependência de carregamento (HU-006):**

```sql
UPDATE tab_produto SET estoque_cheios = 25 WHERE id = 1;  -- acima do limite: dispensa
UPDATE tab_produto SET estoque_cheios = 20 WHERE id = 1;  -- nao acima: mantem alerta
```

**Prova complementar (API):** a consulta de estoque retorna `estoqueBaixo: false` quando o
saldo está acima do limite e `estoqueBaixo: true` quando permanece igual ou abaixo.

## Roteiro resumido

| Passo | Ação | Resultado esperado |
|---|---|---|
| 1 | Definir limite mínimo de 20 para "Gás P13" | Salvo e exibido no cadastro |
| 2 | Baixar o saldo de cheios para 20 | Alerta visual ativo (RF-032) |
| 3 | Navegar entre telas | Alerta permanece visível |
| 4 | Baixar o saldo para 15 | Alerta continua ativo |
| 5 | Elevar o saldo para 25 via carregamento | Alerta dispensado |
| 6 | Baixar novamente para 20 | Alerta volta a valer |

## Fora do escopo

- Movimentação real de estoque por venda (HU-007) e carregamento (HU-006): este guia usa
  ajuste direto no banco para isolar a prova do alerta; a prova com movimentos reais é
  coberta pelos quickstarts daquelas HU.
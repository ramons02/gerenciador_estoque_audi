# API Contract: HU-021 - Cadastro de Carga e Vasilhame

**HU**: HU-021 | **Feature**: Cadastro de Carga e Vasilhame | **Date**: 2026-08-20 | **Spec**: [spec.md](../spec.md)
**Requisitos vinculados**: FR-001 a FR-006 (CT-001 a CT-006)
**Data model**: [data-model.md](../data-model.md)

Módulos REST `cargas` e `vasilhames` (pacotes `com.gerenciador.estoque.business.carga` e `com.gerenciador.estoque.business.vasilhame`). Os endpoints de criação já existiam; esta feature adiciona validação no Service e documenta o contrato. Todas as respostas de erro são JSON estruturado `{ "timestamp": "...", "status": ..., "message": "..." }` em pt-BR (CONVENTIONS §6, GlobalExceptionHandler).

---

## POST /api/cargas

Cria uma carga nova.

**Request** (body JSON):

```json
{
  "nome": "Refrigerante"
}
```

**Regras** (Service - CargaService.validar):
- `nome` obrigatório (CT-005): vazio ou só espaços gera 422 "Informe o nome da carga.".
- `nome` único (CT-004): igualdade exata após trim; duplicado gera 422 `Já existe uma carga cadastrada com o nome 'X'.`.
- Na edição (`PUT /api/cargas/{id}`), o próprio registro não conta como duplicado.

**Response 200** - carga criada:

```json
{
  "id": 3,
  "nome": "Refrigerante",
  "criadoEm": "2026-08-20T16:13:28.930",
  "atualizadoEm": "2026-08-20T16:13:28.930",
  "ativo": true
}
```

**Erros**: 422 nome vazio ou duplicado; 500 erro inesperado (mensagem genérica do GlobalExceptionHandler).

---

## POST /api/vasilhames

Cria um vasilhame novo.

**Request** (body JSON):

```json
{
  "nome": "Lata 350ml",
  "precoCasco": 0
}
```

**Regras** (Service - VasilhameService.validar):
- `nome` obrigatório (CT-005): vazio ou só espaços gera 422 "Informe o nome do vasilhame.".
- `nome` único (CT-004): igualdade exata após trim; duplicado gera 422 `Já existe um vasilhame cadastrado com o nome 'X'.`.
- `precoCasco` pode ser zero na criação (Assumptions do spec); ausente assume 0 (default do JPA).
- Na edição (`PUT /api/vasilhames/{id}`), o próprio registro não conta como duplicado.

**Response 200** - vasilhame criado:

```json
{
  "id": 4,
  "nome": "Lata 350ml",
  "precoCasco": 0,
  "criadoEm": "2026-08-20T16:13:23.495",
  "atualizadoEm": "2026-08-20T16:13:23.495",
  "ativo": true
}
```

**Erros**: 422 nome vazio ou duplicado; 500 erro inesperado.

---

## Endpoints reutilizados (sem alteração)

- `GET /api/cargas` e `GET /api/vasilhames`: listagem de ativos (`findByAtivoTrue`) para os seletores da tela.
- `POST /api/produtos`: cadastro do produto combinado com carga e vasilhame novos (CT-006); duplicidade da combinação bloqueada pela constraint `uk_produto_carga_vasilhame` (RDN-001).
- `DELETE /api/cargas/{id}` e `DELETE /api/vasilhames/{id}`: exclusão lógica (`ativo = FALSE`), preservando histórico (RNF-007).

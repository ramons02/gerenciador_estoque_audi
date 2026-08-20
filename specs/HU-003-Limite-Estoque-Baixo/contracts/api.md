# Contrato da API: Limite Mínimo de Estoque (Alerta)

**HU**: HU-003 - Limite Mínimo de Estoque (Alerta)
**Branch**: `HU-003-Limite-Estoque-Baixo`

Base URL: `/api`. Erros estruturados em JSON conforme `GlobalExceptionHandler` (CONVENTIONS §6).

## Formato de erro padrão

```json
{
  "timestamp": "2026-08-20T10:00:00",
  "status": 422,
  "message": "mensagem de erro em pt-BR"
}
```

| HTTP | Situação |
|---|---|
| 200 | Sucesso (recurso ou lista) |
| 400 | Parâmetro obrigatório ausente |
| 404 | Recurso não encontrado ou inativo |
| 422 | Validação de negócio (mensagem pt-BR) |
| 500 | Erro interno não tratado |

---

## 1. Configuração do limite mínimo

### PUT /api/produtos/{id}/limite-minimo

Define o limite mínimo de cheios de um produto (CT-001).

**Request**:
```json
{
  "limiteMinimo": 20
}
```

**Resposta 200**: produto atualizado (corpo do produto, incluindo `limiteMinimo` e
`estoqueCheios`).

**Erros**:
- 404: `Produto não encontrado(a) com id {id}.`
- 422: `O limite mínimo de estoque não pode ser negativo.`

### GET /api/produtos

Lista os produtos com `limiteMinimo` e `estoqueCheios`, base para o painel de estoque e para
o cálculo do alerta (CT-001: exibição do limite configurado).

**Resposta 200**: itens com `limiteMinimo`, `estoqueCheios`, `estoqueVazios` e `ativo`.

## 2. Alerta de estoque baixo (consumo por HU-012 e HU-019)

O alerta é derivado pelo backend na consulta de estoque (condição `limiteMinimo > 0` e
`estoqueCheios <= limiteMinimo`, RF-032). Não existe endpoint próprio de alerta.

**Resposta do painel de estoque (HU-012) e do dashboard (HU-019)** inclui, por produto:
```json
{
  "id": 1,
  "nome": "Gas P13",
  "estoqueCheios": 15,
  "estoqueVazios": 8,
  "emRua": 2,
  "limiteMinimo": 20,
  "estoqueBaixo": true,
  "sugestaoReposicao": 0
}
```

| Campo | Significado |
|---|---|
| estoqueBaixo | `true` quando `limiteMinimo > 0` e `estoqueCheios <= limiteMinimo` (CT-002) |
| limiteMinimo | Limite configurado (RF-003) |
| sugestaoReposicao | Sugestão de reposição com base na média de vendas (RGN-009, implementada na HU-012) |

## 3. Dispensa do alerta

Não existe endpoint de dispensa: o alerta deixa de valer quando um carregamento
(`POST /api/carregamentos`, HU-006) eleva `estoqueCheios` acima do `limiteMinimo` na mesma
transação (RGN-004, CT-003). Carregamento parcial mantém o alerta ativo.

---

## Rastreabilidade

- Endpoints: RF-003 (limite por produto), RF-032 (alerta), RGN-004 (dispensa por
  carregamento), CT-001 a CT-003.
- Commits e documentação do módulo referenciam HU-003 (CONSTITUICAO §X, CONVENTIONS §9).
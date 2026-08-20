# Contract API: Cancelamento/Estorno de Venda

**HU**: HU-020 - Cancelamento/Estorno de Venda
**Módulo**: `com.gerenciador.estoque.venda` | **Base**: `/api/vendas`

Endpoint de ESCRITA (RGN-007, RNF-006/007). O cancelamento é transacional: reversão de estoque e caixa na mesma transação (CONVENTIONS §5.1).

## PUT /api/vendas/{id}/cancelar

Cancela/estorna a venda: reverte o estoque conforme o tipo de venda (FR-001), estorna o caixa pela exclusão das somas de vendas ATIVA (FR-002), mantém o registro no histórico com status CANCELADA (FR-003) e grava quem, quando e o motivo (FR-004, CT-003).

### Request JSON

```json
{
  "motivo": "Cliente desistiu após o lançamento"
}
```

### Response JSON 200

```json
{
  "id": 42,
  "status": "CANCELADA",
  "dataHoraCancelamento": "2026-08-20T16:45:00-03:00",
  "usuarioCancelamento": "Ana (admin)",
  "motivo": "Cliente desistiu após o lançamento",
  "reversao": {
    "estoque": {
      "produto": "Gás P13",
      "cheiosVoltaram": 1,
      "vaziosRetirados": 1
    },
    "valorEstornado": 50.00
  }
}
```

## Erros

| HTTP | codigo | mensagem (pt-BR) |
|---|---|---|
| 400 | VALIDACAO | "Informe o motivo do cancelamento." (FR-004, §XIII.2) |
| 404 | VENDA_NAO_ENCONTRADA | "Venda não encontrada." |
| 409 | VENDA_JA_CANCELADA | "Esta venda já está cancelada." (FR-006, Edge Case do spec) |
| 409 | ESTOQUE_INCONSISTENTE | "Não foi possível reverter o estoque: o saldo do produto não suporta o estorno. Contate o administrador." (RDN-005, Edge Case do spec) |
| 401 | NAO_AUTENTICADO | "Usuário não autenticado. Faça login para continuar." |
| 403 | SEM_PERMISSAO | "Acesso negado. Perfil administrador é necessário." |

## Notas de implementação

- O serviço é `@Transactional`; o saldo é consultado com `@Lock(LockModeType.PESSIMISTIC_WRITE)` por produto dos itens (RNF-008, CONVENTIONS §8).
- Reversão simétrica: troca devolve cheio e retira vazio do pátio; vasilhame novo/avulsa devolve o casco de "em rua" ao estoque de cheios (FR-001, RF-023/024/026).
- O estorno do caixa é implícito: a venda CANCELADA sai das somas de dashboard, relatórios e fechamento de caixa na consulta seguinte (FR-002, CT-004, RGN-007).
- Venda CANCELADA permanece consultável no histórico, com todos os dados originais (RNF-007).
- Erros sempre estruturados em JSON (CONVENTIONS §6).
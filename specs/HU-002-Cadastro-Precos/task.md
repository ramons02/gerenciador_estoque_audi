# Tarefas - HU-002 — Cadastro de Preços (Custo e Venda)

**HU de origem:** HU-002

| # | Critério | Status | Ação |
|---|---|---|---|
| 1 | CT-001 | concluído | implementar e provar |
| 2 | CT-002 | concluído | implementar e provar |
| 3 | CT-003 | concluído com ressalva | implementar e provar |

## Evidências (2026-08-17)

- **CT-001** - POST /api/produtos com `precoCusto` e `precoVenda` persiste (HTTP 200,
  produto id 1: custo 90.00 / venda 110.00). Tela de cadastro com os dois campos no app.
- **CT-002** - POST com `precoVenda` 15.00 < `precoCusto` 20.00 retorna 422:
  "O preço de venda não pode ser menor que o preço de custo." (RGN-005). Teste de
  unidade `salvarPrecoVendaMenorQueCustoLancaErro` e `alterarPrecoVendaParaMenorQueCustoLancaErro`.
- **CT-003** - PUT /api/produtos/1 altera `precoVenda` de 110.00 para 135.00 (HTTP 200) e
  persiste na base. Testes `alterarPrecoVendaViaAtualizacaoPersisteNovoValor` e
  `salvarPrecoCustoZeroLancaErro`/`salvarPrecoVendaZeroLancaErro`.

## Ressalvas

- **CT-003 (parte 2)** - "o sistema mantém o preço da venda no momento em que ela é lançada
  (não recalcula vendas antigas)" depende do módulo de venda (HU-007); fica bloqueado até lá.
- **RGN-005** - exceção "exceto se o administrador autorizar" não implementada (não existe
  controle de usuário/admin ainda); validação segue rígida.

## Regras de execução

- Uma tarefa só fecha com CT provado (teste ou evidência registrada).
- Commit: `tipo(escopo): descrição` referenciando HU (CONVENTIONS §9).
- Atualizar o status conforme avança (pendente → em andamento → concluído).
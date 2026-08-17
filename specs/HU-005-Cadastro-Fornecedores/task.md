# Tarefas - HU-005 — Cadastro de Fornecedores (Distribuidoras)

**HU de origem:** HU-005

| # | Critério | Status | Ação |
|---|---|---|---|
| 1 | CT-001 | concluído | POST /api/fornecedores criou Ultragaz com contato (id 1) — evidência no Neon; 5 testes em FornecedorServiceTest (13/13 + 17 = suíte 30 verdes) |
| 2 | CT-002 | concluído | GET /api/fornecedores alimenta o select de fornecedor na tela de Chegada de Caminhão (CarregamentosPage); fornecedor id 1 usado em carregamento registrado com sucesso |

## Regras de execução

- Uma tarefa só fecha com CT provado (teste ou evidência registrada).
- Commit: `tipo(escopo): descrição` referenciando HU (CONVENTIONS §9).
- Atualizar o status conforme avança (pendente → em andamento → concluído).

# Tarefas - HU-001 — Cadastro de Produtos (Carga + Vasilhame)

**HU de origem:** HU-001

| # | Critério | Status | Ação |
|---|---|---|---|
| 1 | CT-001 | concluído | modelagem carga/vasilhame + entidades + controllers |
| 2 | CT-002 | concluído | `getNome()` combinado (carga + vasilhame) |
| 3 | CT-003 | concluído | validação de duplicidade no `ProdutoService` (teste + uk no banco) |
| 4 | CT-004 | concluído com ressalva | exclusão lógica bloqueada quando houver movimentações; `existsMovimentacao` retorna false até as tabelas de Entrada (HU-006) e Venda (HU-007) existirem |

## Regras de execução

- Uma tarefa só fecha com CT provado (teste ou evidência registrada).
- Commit: `tipo(escopo): descrição` referenciando HU (CONVENTIONS §9).
- Atualizar o status conforme avança (pendente → em andamento → concluído).

## Evidências

- `mvn test` → **8 testes, 0 falhas** (`ProdutoServiceTest`).
- Arquivos: `tab_carga`, `tab_vasilhame`, `tab_produto` (migration V1), `Carga*`, `Vasilhame*`,
  `Produto*` (entidade com composição carga+vasilhame), validações de preço e duplicidade.
- Pendente: prova de execução com banco (credenciais Postgres locais) e tela no app (HU-001 app).
# Contrato de UI: Cadastro de Fornecedores (Distribuidoras)

**HU**: HU-005 - Cadastro de Fornecedores (Distribuidoras)
**Branch**: `HU-005-Cadastro-Fornecedores`

Telas: listagem/formulário de fornecedores (`src/features/fornecedores/`) e seleção de
fornecedor no registro de carregamento (`src/features/carregamentos/`, HU-006). Consumo da
API por cliente HTTP tipado (`src/api/fornecedores.ts`). Vocabulário canônico
(CONSTITUICAO §II): "Carregamento" para entrada de caminhão.

## 1. Listagem de fornecedores

| Elemento | Descrição |
|---|---|
| Título | "Fornecedores" |
| Colunas | Nome, Contato |
| Ações | Botão "Novo fornecedor" e botão "Editar" por linha |

**Comportamento:** carrega `GET /api/fornecedores` (somente ativos).

## 2. Formulário de fornecedor

| Campo | Tipo | Obrigatório | Observação |
|---|---|---|---|
| Nome da distribuidora | Texto | sim | Único entre ativos |
| Contato | Texto | não | Não interfere na seleção do carregamento |

**Ações:** "Salvar" (POST ou PUT) e "Cancelar".

**Validações inline:**
- Nome vazio: "Informe o nome do fornecedor." (bloqueia o salvar).
- Nome duplicado (resposta 422 da API): "Já existe um fornecedor com este nome." (exibida
  sem colapsar em erro genérico, CONSTITUICAO §XI.3).

## 3. Seleção no registro de carregamento (CT-002)

| Elemento | Descrição |
|---|---|
| Onde | Tela de registro de carregamento (HU-006), campo "Fornecedor" |
| Origem dos dados | Lista de `GET /api/fornecedores` (somente cadastrados e ativos) |
| Comportamento | O fornecedor cadastrado aparece na seleção; ao salvar, o carregamento fica vinculado ao fornecedor selecionado |
| Bloqueio | Não é possível registrar carregamento sem um fornecedor cadastrado (mensagem 422 da API) |

## 4. Exclusão de fornecedor

| Elemento | Descrição |
|---|---|
| Ação | Botão "Excluir" por linha, com confirmação |
| Comportamento | `DELETE /api/fornecedores/{id}` (exclusão lógica, 204) |
| Consequência | Some da listagem e da seleção de novos carregamentos; carregamentos históricos permanecem com o nome no relatório (RF-042) |

## 5. Mensagens

| Situação | Mensagem |
|---|---|
| Sucesso ao salvar | "Fornecedor salvo com sucesso." |
| Sucesso ao excluir | "Fornecedor excluído." |
| Erro de validação | Mensagem 422 da API em pt-BR |
| Falha de rede/API | Mensagem da API quando estruturada; caso contrário mensagem genérica em pt-BR |

## 6. Rastreabilidade

- CT-001 (cadastro com nome e contato), CT-002 (disponibilidade na seleção do carregamento),
  RF-005, RF-010.
- Commits referenciam HU-005 (CONVENTIONS §9).
# Contrato de UI: Cadastro de Produtos (Carga + Vasilhame)

**HU**: HU-001 - Cadastro de Produtos (Carga + Vasilhame)
**Branch**: `HU-001-Cadastro-Produtos`

Tela: `src/features/produtos/ProdutosPage.tsx`. Consome a API por cliente HTTP tipado
(`src/api/produtos.ts`), nunca `fetch` solto no componente (CONVENTIONS §7). Vocabulário
canônico nas telas: Carga, Vasilhame, Produto (item vendido), Cheio, Vazio (CONSTITUICAO §II).

## 1. Listagem de produtos

| Elemento | Descrição |
|---|---|
| Título | "Produtos" |
| Coluna 1 | Produto (nome combinado, ex.: "Gás P13") |
| Coluna 2 | Carga |
| Coluna 3 | Vasilhame |
| Coluna 4 | Preço de venda (R$) |
| Coluna 5 | Estoque cheio |
| Ações | Botão "Novo produto" e botão "Editar" por linha |

**Comportamento:** carrega `GET /api/produtos`; cada item exibe o nome combinado de carga +
vasilhame (CT-002).

## 2. Formulário de produto (criar e editar)

| Campo | Tipo | Obrigatório | Origem dos dados |
|---|---|---|---|
| Carga | Select | sim | `GET /api/cargas` |
| Vasilhame | Select | sim | `GET /api/vasilhames` |
| Preço de custo (R$) | Numérico | sim | Manuseado pela HU-002 |
| Preço de venda (R$) | Numérico | sim | Manuseado pela HU-002 |
| Limite mínimo de cheios | Numérico | sim (default 0) | Manuseado pela HU-003 |

**Ações:** "Salvar" (POST ou PUT conforme o modo) e "Cancelar" (volta à listagem).

**Validações inline (bloqueiam o salvar):**
- Carga vazia: "Informe a carga do produto."
- Vasilhame vazio: "Informe o vasilhame do produto."
- Valores não numéricos, zerados ou negativos nos campos de preço.
- Preço de venda menor que o preço de custo (RGN-005).

**Validações vindas da API (exibidas na tela, sem colapsar em erro genérico - CONSTITUICAO §XI.3):**
- Duplicidade da combinação carga + vasilhame (CT-003).

## 3. Exclusão de produto

| Elemento | Descrição |
|---|---|
| Ação | Botão "Excluir" por linha, com confirmação |
| Comportamento | `DELETE /api/produtos/{id}` (exclusão lógica) |
| Bloqueio | Se houver movimentações vinculadas, a API responde 422 e a tela exibe a mensagem "Não é possível excluir o produto {nome} porque ele possui movimentações vinculadas." (CT-004) |

## 4. Mensagens

| Situação | Mensagem |
|---|---|
| Sucesso ao salvar | "Produto salvo com sucesso." |
| Sucesso ao excluir | "Produto excluído." |
| Falha de rede/API | Mensagem da API quando estruturada; caso contrário mensagem genérica em pt-BR |

## 5. Rastreabilidade

- CT-001 (composição carga + vasilhame), CT-002 (nome combinado), CT-003 (duplicidade),
  CT-004 (edição e exclusão condicionada), RF-001.
- Commits referenciam HU-001 (CONVENTIONS §9).
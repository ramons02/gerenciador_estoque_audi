# Contrato de UI: Cadastro de Preços (Custo e Venda)

**HU**: HU-002 - Cadastro de Preços (Custo e Venda)
**Branch**: `HU-002-Cadastro-Precos`

Tela: `src/features/produtos/ProdutosPage.tsx` (formulário e listagem de produto, onde os
preços aparecem como campos do cadastro). Consome a API por cliente HTTP tipado
(`src/api/produtos.ts`). Vocabulário canônico nas telas (CONSTITUICAO §II).

## 1. Formulário de produto (campos de preço)

| Campo | Tipo | Obrigatório | Observação |
|---|---|---|---|
| Preço de custo (R$) | Numérico, máscara monetária | sim | Maior que zero |
| Preço de venda (R$) | Numérico, máscara monetária | sim | Maior que zero e >= preço de custo (RGN-005) |

**Ações:** "Salvar" (POST ou PUT conforme o modo), "Cancelar".

**Validações inline (bloqueiam o salvar):**
- Campo vazio: "Informe o preço de custo/venda."
- Valor zerado ou negativo: "O preço de custo/venda deve ser maior que zero."
- Valor não numérico: bloqueado pela máscara monetária.
- Preço de venda menor que o preço de custo: "O preço de venda não pode ser menor que o
  preço de custo." (RGN-005, CT-002).
- Preço de venda igual ao custo: permitido.

**Validações vindas da API (exibidas sem colapsar em erro genérico - CONSTITUICAO §XI.3):**
- Mesmas mensagens de 422 descritas em `contracts/api.md`.

## 2. Listagem de produtos (colunas de preço)

| Coluna | Conteúdo |
|---|---|
| Produto | Nome combinado (ex.: "Gás P13") |
| Preço de custo (R$) | `precoCusto` vigente |
| Preço de venda (R$) | `precoVenda` vigente |
| Estoque cheio | `estoqueCheios` |

**Comportamento:** ao consultar a listagem, o cadastro exibe os preços atuais do produto
(CT-001). A alteração de preço acontece pelo botão "Editar" do produto, que abre o formulário
preenchido com os valores vigentes.

## 3. Alteração de preço a qualquer momento (CT-003)

| Elemento | Descrição |
|---|---|
| Ação | Botão "Editar" na linha do produto; salvar com os novos preços |
| Comportamento | `PUT /api/produtos/{id}` com o cadastro completo |
| Consequência | O novo preço vale para lançamentos futuros; vendas já lançadas mantêm o valor unitário original (sem aviso de recalculo, não há recálculo) |

## 4. Mensagens

| Situação | Mensagem |
|---|---|
| Sucesso ao salvar | "Produto salvo com sucesso." |
| Erro de validação | Mensagem 422 da API em pt-BR |
| Falha de rede/API | Mensagem da API quando estruturada; caso contrário mensagem genérica em pt-BR |

## 5. Rastreabilidade

- CT-001 (preços no cadastro), CT-002 (validação venda >= custo), CT-003 (alteração livre
  sem recálculo), RF-002, RGN-005.
- Commits referenciam HU-002 (CONVENTIONS §9).
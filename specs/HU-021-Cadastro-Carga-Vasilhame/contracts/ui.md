# UI Contract: HU-021 - Cadastro de Carga e Vasilhame

**HU**: HU-021 | **Feature**: Cadastro de Carga e Vasilhame | **Date**: 2026-08-20 | **Spec**: [spec.md](../spec.md)
**Requisitos vinculados**: FR-001 a FR-006 (CT-001 a CT-006)

Tela afetada: `ProdutosPage` (`src/features/produtos/ProdutosPage.tsx`), aba "Produtos". Cliente HTTP tipado em `src/api/produtos.ts` (CONVENTIONS §7, nunca fetch solto).

---

## Formulário de cadastro de produto (card "Cadastrar produto")

### Seletor de Carga

- Options: as cargas ativas de `GET /api/cargas` + opção terminal **"Nova carga..."**.
- Ao selecionar "Nova carga...": abre o formulário inline abaixo do seletor (CT-001).
- Ao selecionar uma carga existente: fecha o formulário inline e mantém o comportamento atual (preenchimento automático do vasilhame padrão Gás/P13 e Água/Galão 20L).

### Formulário inline "Nova carga..."

- Campo texto "Nome da nova carga" (obrigatório; CT-005).
- Botão **"Criar carga"**: chama `criarCarga(nome)` (`POST /api/cargas`); em sucesso, recarrega `GET /api/cargas`, fecha o inline, limpa o campo e **seleciona a carga criada** no seletor (CT-003).
- Botão **"Cancelar"**: fecha o inline sem salvar.
- Erro (ex.: nome duplicado, CT-004): mensagem do backend exibida no alerta de erro da página (`.alerta.erro`), inline permanece aberto com o nome digitado.

### Seletor de Vasilhame

- Options: os vasilhames ativos de `GET /api/vasilhames` + opção terminal **"Novo vasilhame..."**.
- Ao selecionar "Novo vasilhame...": abre o formulário inline abaixo do seletor (CT-002).

### Formulário inline "Novo vasilhame..."

- Campo texto "Nome do novo vasilhame" (obrigatório; CT-005).
- Campo numérico "Preco do casco (R$)" (opcional; vazio envia 0 - Assumptions do spec).
- Botão **"Criar vasilhame"**: chama `criarVasilhame(nome, precoCasco)` (`POST /api/vasilhames`); em sucesso, recarrega `GET /api/vasilhames`, fecha o inline, limpa os campos e **seleciona o vasilhame criado** no seletor (CT-003).
- Botão **"Cancelar"**: fecha o inline sem salvar.
- Erro (ex.: nome duplicado, CT-004): mensagem do backend exibida no alerta de erro, inline permanece aberto.

## Fluxo completo (CT-006)

1. Selecionar "Nova carga..." e criar "Refrigerante" -> seletor fica com "Refrigerante".
2. Selecionar "Novo vasilhame..." e criar "Lata 350ml" -> seletor fica com "Lata 350ml".
3. Informar preço de custo, preço de venda e limite mínimo e clicar **"Cadastrar"**.
4. O produto "Refrigerante Lata 350ml" aparece na tabela "Produtos cadastrados" (nome combinado carga + vasilhame, CT-002 do HU-001).

## Estilo

- Formulário inline usa a classe `.nova-opcao` (borda tracejada, fundo de superfície, flex com wrap), inputs com o mesmo padrão visual dos demais controles do formulário.
- Botões reutilizam `.botao` e `.botao.primario` existentes.
# Quickstart: HU-021 - Cadastro de Carga e Vasilhame

**HU**: HU-021 | **Feature**: Cadastro de Carga e Vasilhame | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Referências**: [api.md](contracts/api.md) | [ui.md](contracts/ui.md) | [data-model.md](data-model.md)

Guia de validação end-to-end dos CTs da HU-021. Não substitui os contratos: consultar os links acima para campos e mensagens exatas.

---

## Pré-requisitos

- PostgreSQL rodando localmente (config em `gerenciador_estoque_infra/env`); migrations Flyway aplicadas automaticamente na subida da API (V1 a V9 existentes).
- API: `cd gerenciador_estoque_api && mvn spring-boot:run` (porta 8080).
- App: `cd gerenciador_estoque_app && npm install && npm run dev` (porta 5173) ou o JAR com o frontend embutido.
- Massa mínima: cargas "Gas"/"Agua" e vasilhames "P13"/"Galão 20L" (seed V8).

---

## Cenários de prova (roteirizados)

### CT-001 - Cadastro de carga nova pela tela

Na aba Produtos, selecionar "Nova carga..." no seletor de Carga, informar "Refrigerante" e clicar "Criar carga". Verificar que o seletor passa a exibir "Refrigerante" selecionado e que "Refrigerante" aparece em `GET /api/cargas`.

### CT-002 - Cadastro de vasilhame novo pela tela

Na aba Produtos, selecionar "Novo vasilhame..." no seletor de Vasilhame, informar "Lata 350ml" (preço do casco pode ficar 0) e clicar "Criar vasilhame". Verificar que o seletor passa a exibir "Lata 350ml" selecionado e que aparece em `GET /api/vasilhames`.

### CT-003 - Item criado fica selecionado e o cadastro continua

Após criar carga e vasilhame novos, informar preço de custo, preço de venda e limite mínimo e clicar "Cadastrar". Verificar que o produto aparece na tabela com o nome combinado (ex.: "Refrigerante Lata 350ml") e que `GET /api/produtos` retorna a nova combinação.

### CT-004 - Duplicidade bloqueada

Com "Gás" cadastrada, tentar criar "Gás" via "Nova carga...": verificar mensagem `Já existe uma carga cadastrada com o nome 'Gás'.` no alerta de erro e o inline permanecendo aberto. Repetir com "P13" no vasilhame: `Já existe um vasilhame cadastrado com o nome 'P13'.`. Via API: `POST /api/cargas {"nome":"Gas"}` responde 422.

### CT-005 - Nome vazio bloqueado

Tentar criar carga ou vasilhame com nome em branco: verificar mensagem de nome obrigatório ("Informe o nome da carga." / "Informe o nome do vasilhame.") e nada persistido.

### CT-006 - Produto combinado com carga e vasilhame novos

Repetir o fluxo do CT-003 com uma combinação nova (ex.: "Refrigerante" + "Lata 350ml"): o produto é salvo e listado. Tentar cadastrar a mesma combinação de novo: bloqueado (constraint `uk_produto_carga_vasilhame`; mensagem do servidor no alerta).

---

## Observações

- A exclusão de carga/vasilhame é lógica (ativo = FALSE); itens com movimentações nunca são apagados (RNF-007).
- O acréscimo do cartão continua valendo somente para produtos de carga Gás (RF-021-A); produtos com cargas novas usam preço normal em qualquer forma de pagamento.
- Os dados de teste usados nesta validação devem ser removidos ao final (DELETE lógico via `DELETE /api/cargas/{id}`, `DELETE /api/vasilhames/{id}` e `DELETE /api/produtos/{id}`).
# Quickstart: HU-007 - Lançamento Rápido de Venda (Balcão/Entrega)

**HU**: HU-007 | **Feature**: Lançamento Rápido de Venda | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Referências**: [api.md](contracts/api.md) | [ui.md](contracts/ui.md) | [data-model.md](data-model.md)

Guia de validação end-to-end dos CTs da HU-007. Não substitui os contratos: consultar os links acima para campos e mensagens exatas.

---

## Pré-requisitos

- PostgreSQL rodando localmente (config em `gerenciador_estoque_infra/env`); migrations Flyway aplicadas automaticamente na subida da API (V1 a V9 existentes).
- API: `cd gerenciador_estoque_api && mvn spring-boot:run` (porta 8080).
- App: `cd gerenciador_estoque_app && npm install && npm run dev` (porta 5173).
- Massa mínima: 1 produto ativo com preço de venda (HU-001/HU-002), saldo de cheios conhecido (carregamento via HU-006 ou `UPDATE` de teste) e Configurações com as formas desejadas (V9 insere Dinheiro/PIX/Cartão habilitados e acréscimo 0,00).

---

## Cenários de prova (roteirizados)

### CT-001 - Campos obrigatórios

Dado o lançamento de venda aberto, quando o vendedor tenta confirmar sem produto, quantidade, tipo ou forma de pagamento, então o sistema exige os campos obrigatórios com validação inline ("Informe o produto.", "Informe a quantidade.", "Informe o tipo da venda.", "Informe a forma de pagamento.") e não conclui. Contrato: [ui.md](contracts/ui.md).

### CT-002 - Total calculado automaticamente

Dado um produto com preço de venda cadastrado (ex.: R$ 115,00), quando o vendedor informa quantidade 2, então o total exibido é R$ 230,00 (2 × preço) antes da confirmação, e o `total` gravado confere com o retorno do `POST /api/vendas`. Contrato: [api.md](contracts/api.md); cálculo no Service (research.md, Decisão 3).

### CT-003 - Formas de pagamento habilitadas

Dado Configurações com Dinheiro, PIX e Cartão habilitados, quando o vendedor abre a lista de formas, então apenas as habilitadas aparecem. Desabilitar PIX em `PUT /api/configuracoes` e recarregar: PIX some da lista e o Service rejeita venda com PIX (422 "A forma de pagamento PIX não está habilitada nas Configurações."). Fiado nunca aparece (RGN-002). Contrato: [api.md](contracts/api.md).

### CT-003-A - Acréscimo do cartão (somente carga Gás)

Dado acréscimo de R$ 5,00 por unidade em `GET /api/configuracoes`, quando a mesma venda de produto de carga Gás (2 unidades, preço 115,00) é lançada com Cartão, então o total é R$ 240,00 (2 × 115,00 + 2 × 5,00) e `acrescimoCartao` é gravado; com Dinheiro ou PIX, o total é R$ 230,00 sem acréscimo. Quando a mesma venda é feita com produto de carga Água (ex.: "Água Galão 20L", 2 unidades, preço 20,00) paga com Cartão, então o total é R$ 40,00, preço normal sem acréscimo (RF-021-A, RGN-002). Contrato: [api.md](contracts/api.md); regra em [data-model.md](data-model.md) (`tab_configuracao.acrescimo_cartao`).

### CT-004 - Bloqueio por estoque insuficiente

Dado um produto com 10 cheios em estoque (conferir em `GET /api/estoque`), quando o vendedor lança 11 unidades, então o sistema bloqueia com "Estoque insuficiente para 11 unidade(s) de Gás P13. Disponível: 10." (409). Quando lança exatamente 10, o lançamento é permitido (limite exato). Contrato: [api.md](contracts/api.md); regra em [data-model.md](data-model.md) (RDN-005, RNF-008).

### CT-005 - Lançamento em poucos segundos

Dado o lançamento aberto, quando o vendedor executa o fluxo completo (produto, quantidade, tipo, forma, confirmar), então a resposta chega em até 2 segundos mesmo com o histórico do dia aberto (RNF-003). Medir com o histórico populado; validações são locais e o `POST /api/vendas` é único.

### CT-006 - Data/hora e usuário registrados

Dado um lançamento confirmado, quando o histórico é consultado, então cada venda mostra data/hora (fuso Brasília) e o usuário responsável (`idUsuario` autenticado). Verificação no banco: `SELECT data_hora, id_usuario FROM tab_venda WHERE id = <id>;` e `SELECT tipo, quantidade, id_referencia FROM tab_movimentacao_estoque WHERE id_referencia = <id>;` deve conter SAIDA_CHEIO na mesma transação (RNF-005, RNF-006).

---

## Cenário de resiliência (edge cases)

- Produto com estoque zerado: venda bloqueada com mensagem clara.
- Acréscimo do cartão calculado por unidade e multiplicado pela quantidade, antes da soma do total, somente para produto de carga Gás (Água nunca sofre acréscimo).
- Concorrência (RNF-008): duas vendas simultâneas de 8 unidades com saldo 10 → uma conclui, a outra retorna 409, saldo final 2, nunca negativo. Verificação: `SELECT qtd_cheios FROM tab_estoque WHERE id_produto = <p>;` >= 0.
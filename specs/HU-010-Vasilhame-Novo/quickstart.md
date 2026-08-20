# Quickstart: HU-010 - Venda de Vasilhame Novo (Casco + Carga)

**HU**: HU-010 | **Feature**: Venda de Vasilhame Novo | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Referências**: [api.md](contracts/api.md) | [ui.md](contracts/ui.md) | [data-model.md](data-model.md)

Guia de validação end-to-end dos CTs da HU-010. Não substitui os contratos: consultar os links acima para campos e mensagens exatas.

---

## Pré-requisitos

- Mesmos da HU-007 (ver [quickstart HU-007](../HU-007-Lancamento-Venda/quickstart.md)): banco PostgreSQL, API na 8080 (`mvn spring-boot:run`), app na 5173 (`npm run dev`).
- Massa mínima: 1 produto ativo com preço de venda da carga, `preco_casco` definido em `tab_vasilhame` (conferir em `GET /api/produtos`: campo `precoCasco`), 1 cliente cadastrado (HU-004) e saldo de cheios conhecido.

---

## Cenários de prova (roteirizados)

### CT-001 - Venda marcada como vasilhame novo

Dado o lançamento de venda aberto, quando o vendedor marca o item como "Vasilhame novo", então o sistema trata a venda sem devolução de vazio (o request segue com `tipoItem: CASCO_NOVO`). Contrato: [api.md](contracts/api.md).

### CT-002 - Preço casco + carga

Dado um vasilhame com preço de casco 115,00 e um produto com preço de carga 115,00, quando o total é calculado, então o item exibe R$ 230,00 (casco + carga) e o retorno do `POST /api/vendas` traz `precoUnitario: 230.00` e `precoCasco: 115.00`. Verificação no banco: `SELECT preco_unitario, preco_casco FROM tab_venda_item WHERE id_venda = <id>;`. Contrato: [api.md](contracts/api.md); regra em [data-model.md](data-model.md) (RGN-010).

### CT-003 - Baixa de cheio e registro em rua

Dado um produto com 10 cheios e um cliente, quando o vendedor confirma uma venda de vasilhame novo de 2 unidades, então o estoque passa a 8 cheios e o controle do cliente ganha 2 vasilhames "em rua". Verificação no banco: `SELECT qtd_em_rua FROM tab_estoque WHERE id_produto = <p>;` e `SELECT quantidade FROM tab_cliente_vasilhame WHERE cliente_id = <c> AND vasilhame_id = <v>;`, e `SELECT tipo, quantidade FROM tab_movimentacao_estoque WHERE id_referencia = <id da venda>;` contém SAIDA_CHEIO (2) e EM_RUA (2) no mesmo commit (RNF-005).

### CT-004 - Cliente obrigatório

Dado uma venda de vasilhame novo, quando o vendedor tenta confirmar sem informar o cliente, então o sistema bloqueia com "Informe o cliente para venda de vasilhame novo." (422) e não conclui. Quando um cliente é selecionado (ou cadastrado rápido no modal, sem sair do lançamento), a venda conclui (spec.md Edge Cases; RF-004).

---

## Cenário de resiliência (edge cases)

- Cliente ainda não cadastrado: cadastro rápido dentro do lançamento (modal) antes de concluir; após salvar, a venda segue com o cliente selecionado.
- Múltiplas unidades: N cheios baixados e N vasilhames em rua registrados (CT-003).
- Devolução posterior: o vasilhame em rua é baixado no registro do cliente correto (HU-011); conferir `SELECT quantidade FROM tab_cliente_vasilhame WHERE cliente_id = <c>;` após a devolução.
- Estoque insuficiente: venda de 11 unidades com saldo 10 é bloqueada com "Estoque insuficiente para 11 unidade(s) de Gás P13. Disponível: 10." (409) antes de qualquer atualização (RF-031).
- Concorrência (RNF-008): vendas simultâneas não geram cheios nem em rua negativos; verificar `SELECT qtd_cheios, qtd_em_rua FROM tab_estoque WHERE id_produto = <p>;` >= 0 após execução paralela.
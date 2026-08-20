# Quickstart: HU-008 - Venda com Entrega (Taxa de Entrega)

**HU**: HU-008 | **Feature**: Venda com Entrega | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Referências**: [api.md](contracts/api.md) | [ui.md](contracts/ui.md) | [data-model.md](data-model.md)

Guia de validação end-to-end dos CTs da HU-008. Não substitui os contratos: consultar os links acima para campos e mensagens exatas.

---

## Pré-requisitos

- Mesmos da HU-007 (ver [quickstart HU-007](../HU-007-Lancamento-Venda/quickstart.md)): banco PostgreSQL, API na 8080 (`mvn spring-boot:run`), app na 5173 (`npm run dev`).
- Configuração vigente: chave `taxa_entrega` presente em `tab_configuracao` (V3 insere 10.00); conferir em `GET /api/configuracoes`.
- Massa mínima: 1 produto ativo com preço e saldo de cheios.

---

## Cenários de prova (roteirizados)

### CT-001 - Taxa somada automaticamente no tipo Entrega

Dado a taxa de entrega configurada em 10.00 (conferir em `GET /api/configuracoes`), quando o vendedor lança uma venda com tipo Entrega (1 unidade, preço 115,00), então o total exibido é R$ 125,00 com a linha "Taxa de entrega: R$ 10,00", e o retorno do `POST /api/vendas` traz `taxaEntrega: 10.00` e `total: 125.00`. Quando a mesma venda é lançada com tipo Balcão, o total é R$ 115,00 sem taxa. Contrato: [api.md](contracts/api.md).

### CT-002 - Configuração da taxa pelo administrador

Dado um administrador autenticado, quando ele altera a taxa para 15.00 em Configurações (`PUT /api/configuracoes` com `{ "taxaEntrega": 15.00 }`), então o novo valor passa a ser usado nas vendas do tipo Entrega seguintes (`GET /api/configuracoes` retorna 15.00 e o próximo lançamento soma 15.00). Vendas já lançadas mantêm o valor gravado (10.00) - conferir no histórico. Contrato: [api.md](contracts/api.md); regra em [data-model.md](data-model.md) (`tab_venda.taxa_entrega`).

### CT-003 - Discriminação no relatório de vendas

Dado vendas dos tipos Balcão e Entrega registradas no período, quando o relatório de vendas é gerado, então cada venda aparece com seu Tipo (Balcão/Entrega) e o total com a taxa aplicada quando Entrega (RF-041, CONVENTIONS §10). Verificação no banco: `SELECT tipo, taxa_entrega, total FROM tab_venda WHERE id = <id>;` mostra `ENTREGA` com taxa e total coerentes, e `BALCAO` com taxa nula.

---

## Cenário de resiliência (edge cases)

- Taxa configurada como zero (entrega gratuita): a venda Entrega conclui com `taxaEntrega: 0.00` e total sem acréscimo.
- Entrega sem taxa configurada: remover a chave `taxa_entrega` de `tab_configuracao` (teste manual no banco, depois restaurar); a confirmação é bloqueada com "A taxa de entrega não está configurada. Defina o valor em Configurações." (422). Restaurar a chave antes de seguir.
- Taxa alterada após vendas do dia já lançadas: apenas as novas vendas usam o novo valor (CT-002; edge case do spec).
- Concorrência e atomicidade: a taxa é gravada na mesma transação do estoque da venda (RNF-005) - a falha de qualquer passo reverte tudo; conferir que o saldo de cheios e o total não ficam parcialmente aplicados.
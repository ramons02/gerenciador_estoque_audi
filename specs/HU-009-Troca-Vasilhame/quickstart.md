# Quickstart: HU-009 - Troca de Vasilhame (Venda Normal)

**HU**: HU-009 | **Feature**: Troca de Vasilhame | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Referências**: [api.md](contracts/api.md) | [ui.md](contracts/ui.md) | [data-model.md](data-model.md)

Guia de validação end-to-end dos CTs da HU-009. Não substitui os contratos: consultar os links acima para campos e mensagens exatas.

---

## Pré-requisitos

- Mesmos da HU-007 (ver [quickstart HU-007](../HU-007-Lancamento-Venda/quickstart.md)): banco PostgreSQL, API na 8080 (`mvn spring-boot:run`), app na 5173 (`npm run dev`).
- Massa mínima: 1 produto ativo com preço de venda e saldos conhecidos de cheios e vazios (conferir em `GET /api/estoque`).

---

## Cenários de prova (roteirizados)

### CT-001 - Baixa de cheio e adição de vazio por unidade vendida

Dado um produto com 10 cheios e 5 vazios no pátio, quando o vendedor lança uma venda com troca de 2 unidades (tipo Balcão, Dinheiro) e confirma, então o estoque passa a 8 cheios e 7 vazios no pátio (-2 cheios, +2 vazios). Verificação no banco: `SELECT tipo, quantidade, id_referencia FROM tab_movimentacao_estoque WHERE id_referencia = <id da venda>;` contém SAIDA_CHEIO com 2 e ENTRADA_VAZIO com 2, gravados no mesmo commit. Contrato: [api.md](contracts/api.md).

### CT-002 - Atomicidade da operação

Dado uma venda com troca em confirmação, quando ocorre falha no meio da operação (ex.: queda de conexão, falha simulada em teste de integração), então nenhuma atualização é aplicada - o estoque permanece com os valores anteriores (8 cheios e 7 vazios no exemplo do CT-001) e não existe venda registrada. A operação completa é aplicada ou nada é aplicado (RNF-005, RDN-004). Coberto por `VendaTrocaIntegracaoTest` (`@SpringBootTest`).

### CT-003 - Troca sem custo

Dado um produto com preço 115,00, quando uma venda com troca de 2 unidades é calculada, então o total é R$ 230,00 - idêntico à venda sem troca (o vazio recebido não altera o total). Contrato: [api.md](contracts/api.md) (Response `total`); regra em [data-model.md](data-model.md).

### CT-004 - Pátio refletido imediatamente

Dado uma venda com troca confirmada, quando o saldo de vazios do pátio é consultado em seguida (`GET /api/estoque` ou painel de estoque), então o vazio recebido já está somado ao saldo, sem nenhuma ação manual adicional. Verificação no banco: `SELECT qtd_vazios FROM tab_estoque WHERE id_produto = <p>;` reflete +N no mesmo instante da venda (Constituição §IV.1).

---

## Cenário de resiliência (edge cases)

- Troca de múltiplas unidades: N cheios baixados e N vazios adicionados na mesma operação (CT-001).
- Pátio já com saldo: o vazio recebido apenas incrementa o saldo.
- Estoque de cheios insuficiente: venda com troca de 11 unidades com saldo 10 é bloqueada com "Estoque insuficiente para 11 unidade(s) de Gás P13. Disponível: 10." (409) antes de qualquer atualização.
- Concorrência (RNF-008): vendas com troca simultâneas não podem gerar saldo negativo em cheios nem em vazios; verificar `SELECT qtd_cheios, qtd_vazios FROM tab_estoque WHERE id_produto = <p>;` >= 0 após execução paralela.
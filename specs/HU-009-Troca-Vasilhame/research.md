# Research: HU-009 - Troca de Vasilhame (Venda Normal)

**HU**: HU-009 | **Feature**: Troca de Vasilhame | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Requisitos vinculados**: RF-023, RF-025, RDN-003, RDN-004, RDN-005, RNF-005, RNF-007, RNF-008

Fase 0 - decisões de design registradas antes de qualquer código. Cada decisão referencia a convenção, constituição ou requisito que a justifica.

---

## Decisão 1: Troca como variação do POST /api/vendas, não endpoint separado

**Decision**: A venda com troca é marcada no request do `POST /api/vendas` da HU-007 (campo `troca: true` no item, tipo_item CHEIO); o mesmo `VendaService` trata o fluxo. Não existe rota `/api/vendas/troca`.

**Rationale**: A troca é comportamento complementar da mesma entidade Venda (RF-023: venda normal em que o cliente entrega 1 vazio e leva 1 cheio). Manter um único endpoint evita duplicação de contrato, de regras (formas de pagamento, bloqueio de estoque) e de tela (RNF-001), e as variações HU-008/009/010 compartilham a mesma transação de venda.

**Alternatives considered**: Endpoint dedicado `/api/vendas/troca` - rejeitado por duplicar regras e contrato sem ganho; fluxo separado no frontend com lógica própria - rejeitado por violar a base comum e aumentar passos de lançamento (RNF-001).

---

## Decisão 2: Movimentação dupla atômica (baixar cheio + adicionar vazio)

**Decision**: Na confirmação da venda com troca, o `VendaService` executa, em UMA transação `@Transactional`: valida saldo de cheios, baixa N cheios, adiciona N vazios ao pátio, persiste venda + itens e grava as movimentações SAIDA_CHEIO e ENTRADA_VAZIO. Falha em qualquer passo reverte tudo.

**Rationale**: RDN-004 define que a troca altera dois estoques simultaneamente (-1 cheio, +1 vazio) e CONVENTIONS §5.4 exige a operação na MESMA transação; RNF-005 proíbe estado parcial (baixar cheio sem adicionar vazio). O teste de atomicidade (CT-002, FR-002) é coberto por integração com falha simulada no meio da operação.

**Alternatives considered**: Duas transações (baixar cheio, depois somar vazio) - rejeitado por permitir estado parcial (defeito grave, §III.3); trigger no banco para o vazio - rejeitado por esconder a regra de negócio e dificultar teste de lógica pura (§XI.2).

---

## Decisão 3: Lock pessimista por produto cobrindo cheios e vazios

**Decision**: Dentro da transação da troca, o saldo de cheios (validação de venda) e o saldo de vazios (incremento do pátio) são lidos/atualizados com `@Lock(PESSIMISTIC_WRITE)` no `ProdutoRepository`; o lock cobre os dois estados do produto, serializando vendas concorrentes.

**Rationale**: RNF-008 exige que vendas simultâneas não gerem estoque negativo; o pátio é contabilmente finito (RDN-003) e recebe vazios de várias vendas concorrentes. Serializar por produto na transação garante a invariante sem corrida (CONVENTIONS §5.3 e §8).

**Alternatives considered**: Lock apenas no cheio (validação) e UPDATE direto no vazio - rejeitado por deixar o pátio sem serialização; lock otimista - rejeitado por retentativas e erros em vez de serialização previsível.

---

## Decisão 4: Troca sem custo: total da venda não alterado

**Decision**: O vazio recebido do cliente não entra no cálculo do total; o `VendaService` calcula `total = soma(quantidade × preco_venda) + acréscimo/taxa conforme forma e tipo` (acréscimo do cartão somente para carga Gás, HU-007), ignorando o flag de troca para valor.

**Rationale**: RF-023 e CT-003 definem que a troca não tem custo para o cliente e não altera o total da venda. Gravar o total sem a troca mantém o caixa coerente com o preço de venda e a conferência do balanço (RGN-008).

**Alternatives considered**: Cobrar o vazio como item de valor zero - rejeitado por poluir o relatório de vendas com linhas sem valor; desconto implícito na troca - rejeitado por violar a regra explícita (troca sem custo).

---

## Decisão 5: Pátio atualizado imediatamente com rastro referenciado à venda

**Decision**: O incremento do pátio (ENTRADA_VAZIO, quantidade N) é gravado em `tab_movimentacao_estoque` com `id_referencia` = id da venda, no mesmo commit; nenhuma ação manual adicional é necessária para o saldo refletir o recebido (CT-004).

**Rationale**: RDN-003 e o spec exigem que o pátio reflita imediatamente o vazio recebido, e RNF-007 exige rastro de auditoria sem DELETE. O `id_referencia` liga a entrada de vazio à venda de origem, permitindo reconstituir o histórico de 12 meses (RNF-010) e a reversão no cancelamento (RGN-007, HU-020).

**Alternatives considered**: Atualizar saldo sem movimentação - rejeitado por perder o rastro (RNF-007) e o balanço (RF-043); movimentação genérica sem referência - rejeitado por não permitir auditar a origem do vazio.

---

## Decisão 6: Bloqueio da troca quando o cheio é insuficiente

**Decision**: A troca segue a mesma validação da HU-007: `quantidade <= qtd_cheios` sob lock, com mensagem estruturada pt-BR ("Estoque insuficiente para N unidade(s) de Gás P13. Disponível: M.") antes de qualquer atualização.

**Rationale**: O spec.md (Edge Cases) exige que a venda com troca seja bloqueada antes de qualquer atualização quando o cheio é insuficiente; RF-031 e RDN-005 valem para toda venda, com troca ou não. A validação antecede a gravação (DECISÕES TÉCNICAS do projeto).

**Alternatives considered**: Permitir a troca parcial ou com saldo negativo - rejeitado por violar RF-031/RDN-005 (invariante não negociável, §VI.3); validar apenas na tela - rejeitado porque o banco é a fonte da verdade (§IV.1).

---

## Consistência com as decisões globais

As decisões seguem as DECISÕES TÉCNICAS do projeto: transação única estoque + caixa com lock pessimista por produto, validação de saldo antes de gravar, cálculo de total no Service, registro de movimentação de estoque, sem DELETE de venda e erros estruturados em pt-BR. A troca é comportamento complementar do `POST /api/vendas` da HU-007, compartilhando a mesma base de dados e as mesmas regras de transação com a HU-010 (vasilhame novo), que movimenta o estoque no sentido oposto (cheio para em rua).
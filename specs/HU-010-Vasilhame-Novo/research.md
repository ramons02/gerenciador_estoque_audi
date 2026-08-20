# Research: HU-010 - Venda de Vasilhame Novo (Casco + Carga)

**HU**: HU-010 | **Feature**: Venda de Vasilhame Novo | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Requisitos vinculados**: RF-004, RF-024, RF-026, RF-028, RDN-005, RDN-008, RNF-005, RNF-007, RNF-008, RGN-010

Fase 0 - decisões de design registradas antes de qualquer código. Cada decisão referencia a convenção, constituição ou requisito que a justifica.

---

## Decisão 1: Vasilhame novo como variação do POST /api/vendas, não endpoint separado

**Decision**: A venda de vasilhame novo é marcada no request do `POST /api/vendas` da HU-007 com `tipoItem = CASCO_NOVO` no item; o mesmo `VendaService` trata o fluxo, com cliente obrigatório e preço composto.

**Rationale**: RF-024 define a venda de vasilhame novo como um tipo de venda (casco novo + carga, sem devolução de vazio), comportamento complementar da mesma entidade Venda (tipo_item CHEIO com troca na HU-009; CASCO_NOVO aqui). Um único endpoint preserva as regras comuns (formas de pagamento, bloqueio de estoque, transação) e a tela única de lançamento (RNF-001).

**Alternatives considered**: Endpoint dedicado `/api/vendas/vasilhame-novo` - rejeitado por duplicar regras e contrato sem ganho; fluxo de venda separado no frontend - rejeitado por violar a base comum e aumentar passos na portaria.

---

## Decisão 2: Preço composto (casco + carga) calculado no Service

**Decision**: O `VendaService` lê `tab_vasilhame.preco_casco` e `tab_produto.preco_venda` (preço da carga do produto composto) e calcula `preco_unitario = preco_casco + preco_venda` para itens CASCO_NOVO, gravando `preco_casco` no item e o total correspondente.

**Rationale**: RGN-010 define que o preço do vasilhame novo é diferente da venda com troca e que o preço do casco é configurado à parte; RF-024 e CT-002 exigem casco + carga. Calcular no Service com valores do banco garante o caixa correto (fonte da verdade, §IV.1) e testabilidade da lógica pura (§XI.2).

**Alternatives considered**: Enviar preço do casco no request - rejeitado por permitir divergência de caixa; preço fixo de casco no código - rejeitado por violar RGN-010 e CONVENTIONS §8 (configurável, nunca hardcoded).

---

## Decisão 3: Cliente obrigatório na venda sem devolução de vazio

**Decision**: O Service rejeita a venda com item CASCO_NOVO (ou qualquer venda sem devolução de vazio) sem `idCliente`, com mensagem estruturada pt-BR; o app oferece busca e cadastro rápido do cliente antes de concluir.

**Rationale**: CT-004 e RF-028 exigem identificar o cliente quando não há devolução de vazio, pois "em rua" só aumenta quando o cliente leva um vasilhame sem devolver equivalente (RDN-008). Sem cliente não há como rastrear o comodato nem dar baixa na devolução. O edge case do spec exige cadastro possível antes de concluir (RF-004).

**Alternatives considered**: Cliente opcional com venda "avulsa" - rejeitado por inviabilizar o controle de comodato por cliente (RF-028); cadastro obrigatório fora do fluxo de venda - rejeitado por quebrar o lançamento rápido (RNF-001) - a solução é busca/cadastro rápido integrado.

---

## Decisão 4: Movimentação atômica baixar cheio + registrar em rua

**Decision**: Na confirmação, o Service executa em UMA transação: valida saldo de cheios, baixa N cheios, registra N vasilhames "em rua" para o cliente (incrementa `tab_cliente_vasilhame`), grava venda + itens e a movimentação EM_RUA com `id_referencia` da venda. Falha em qualquer passo reverte tudo.

**Rationale**: RF-026 exige baixar o cheio e registrar o vasilhame em rua; RNF-005 e CONVENTIONS §5.1 exigem estoque e registro na MESMA transação (estado parcial é defeito grave, §III.3). O controle em rua por cliente (RF-028) é a base da baixa na devolução posterior (HU-011).

**Alternatives considered**: Baixar cheio e atualizar em rua em transações separadas - rejeitado por permitir estado parcial; registrar em rua apenas em relatório (derivado) - rejeitado por duplicar a fonte da verdade e quebrar o rastro de auditoria.

---

## Decisão 5: Controle de comodato em tab_cliente_vasilhame (quantidade por cliente)

**Decision**: O "em rua" é controlado por cliente e vasilhame em `tab_cliente_vasilhame` (quantidade agregada, UNIQUE cliente + vasilhame), incrementada na venda de vasilhame novo e decrementada na devolução (RF-028). O saldo `qtd_em_rua` de `tab_estoque` reflete a soma.

**Rationale**: RDN-008 e RF-028 definem o comodato por cliente com baixa quando o vazio é devolvido; a tabela já existe na V3 com a constraint UNIQUE(cliente_id, vasilhame_id) e é o alvo natural do controle. Manter quantidade agregada por cliente/vasilhame é suficiente para o balanço (RF-043) e para a baixa na devolução.

**Alternatives considered**: Controle unitário por vasilhame (um registro por casco individual) - rejeitado por complexidade sem ganho no escopo (o negócio controla por quantidade); derivar o em rua apenas das vendas - rejeitado por não registrar a baixa da devolução (RF-028).

---

## Decisão 6: Lock pessimista por produto cobrindo cheios e o saldo em rua

**Decision**: A validação de cheios e a atualização de em rua acontecem sob `@Lock(PESSIMISTIC_WRITE)` no `ProdutoRepository`, dentro da transação da venda, serializando vendas concorrentes do mesmo produto.

**Rationale**: RF-031 e RDN-005 exigem estoque nunca negativo e RNF-008 serializa vendas simultâneas; o saldo em rua participa da mesma invariante (todo vasilhame em exatamente um estado, RDN-002). O lock por produto garante leitura e escrita atômicas (CONVENTIONS §5.3 e §8).

**Alternatives considered**: Lock apenas na leitura do cheio - rejeitado por corrida no saldo em rua; controle de concorrência por cliente - rejeitado por não proteger a invariante do produto.

---

## Consistência com as decisões globais

As decisões seguem as DECISÕES TÉCNICAS do projeto: transação única estoque + caixa com lock pessimista por produto, validação de saldo antes de gravar, cálculo de total no Service antes de persistir, registro de movimentação de estoque, sem DELETE de venda e erros estruturados em pt-BR. O vasilhame novo é a variação que movimenta cheio para em rua (oposta à troca da HU-009, que movimenta cheio para vazio), compartilhando o mesmo `POST /api/vendas` e a mesma base de dados.
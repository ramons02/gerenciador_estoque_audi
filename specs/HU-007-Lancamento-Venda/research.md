# Research: HU-007 - Lançamento Rápido de Venda (Balcão/Entrega)

**HU**: HU-007 | **Feature**: Lançamento Rápido de Venda | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Requisitos vinculados**: RF-020, RF-021, RF-021-A, RF-031, RF-052, RNF-001, RNF-003, RNF-005, RNF-006, RNF-008, RGN-002

Fase 0 - decisões de design registradas antes de qualquer código. Cada decisão referencia a convenção, constituição ou requisito que a justifica.

---

## Decisão 1: Venda persistida em transação única com estoque, movimentação e total

**Decision**: O `VendaService.lancar` executa em uma única transação `@Transactional`: valida saldo, calcula total e acréscimo, persiste `tab_venda` + `tab_venda_item`, gera a movimentação em `tab_movimentacao_estoque` e atualiza os saldos. Qualquer falha reverte tudo.

**Rationale**: Toda movimentação altera estoque e caixa na MESMA transação; estado parcial é defeito grave (Constituição §III.3, CONVENTIONS §5.1) e o endpoint de venda DEVE ser transacional (CONVENTIONS §6), reverter estoque e caixa se qualquer passo falhar (RNF-005).

**Alternatives considered**: Gravar venda e atualizar estoque em chamadas separadas - rejeitada por permitir estado parcial; transação no controller - rejeitada por concentrar regra fora do Service (padrão de camadas do projeto).

---

## Decisão 2: Validação de saldo sob lock pessimista por produto

**Decision**: Dentro da transação, o `ProdutoRepository` consulta o saldo de cheios com `@Lock(LockModeType.PESSIMISTIC_WRITE)` antes de gravar a venda; o bloqueio só libera no commit.

**Rationale**: RF-031 bloqueia venda sem estoque suficiente, RDN-005 proíbe estoque negativo e RNF-008 exige que vendas simultâneas não gerem estoque negativo. Sem lock, duas vendas simultâneas podem passar a validação e estourar o saldo (CONVENTIONS §5.3 e §8).

**Alternatives considered**: UPDATE com condição `WHERE estoque_cheios >= quantidade` e checagem de linhas afetadas - rejeitado por perder a mensagem de erro clara e a semântica de validação; lock otimista com versão - rejeitado por causar retentativas e falhas de concorrência em vez de serialização.

---

## Decisão 3: Total calculado no Service, nunca no cliente

**Decision**: O Service calcula `total_item = quantidade × preco_venda` por item e `total_venda = soma dos itens (+ taxa/acréscimo conforme variações)`, usando o preço vigente do produto no banco, e persiste `preco_unitario` e `total` na venda.

**Rationale**: O total é regra de negócio (CT-002, RF-020) e o banco é a fonte da verdade (Constituição §IV.1). Confiar no preço enviado pelo cliente permitiria divergência de caixa e exigiria revalidação de qualquer forma.

**Alternatives considered**: Aceitar total calculado no frontend - rejeitado por risco de caixa divergente (RGN-008); calcular no banco via view - rejeitado por esconder a regra e impedir teste unitário de lógica pura (Constituição §XI.2).

---

## Decisão 4: Acréscimo do cartão aplicado no Service antes de persistir

**Decision**: Quando `forma_pagamento = CARTAO` e a carga do produto do item é Gás, o Service lê `acrescimo_cartao` de `tab_configuracao` (R$ por unidade), multiplica pela quantidade e soma ao total da venda, gravando o valor final e o acréscimo aplicado. Produtos de carga Água usam o preço normal em qualquer forma de pagamento (Dinheiro, PIX ou Cartão), e Dinheiro e PIX usam o preço normal em todos os produtos.

**Rationale**: RF-021-A define acréscimo fixo por unidade com valor configurável (RF-052) e RGN-002 determina que Cartão SEMPRE aplica o acréscimo configurado aos produtos de carga Gás; produtos de carga Água seguem o preço normal em qualquer forma de pagamento. Aplicar no Service garante que o valor gravado seja o cobrado, independente da tela. A distinção por carga vem do cadastro do produto (HU-001, RDN-001/RDN-007).

**Alternatives considered**: Aplicar acréscimo na tela e enviar no request - rejeitado por permitir divergência de caixa; aplicar no total pós-calculo sem gravar o acréscimo - rejeitado por perder o rastro do que foi cobrado (conferência de caixa, RGN-008).

---

## Decisão 5: Formas de pagamento servidas pela configuração vigente

**Decision**: O app carrega as formas habilitadas via `GET /api/configuracoes` (chaves `pagamento_DINHEIRO`, `pagamento_PIX`, `pagamento_CARTAO`) e apresenta apenas as habilitadas; o Service revalida a forma recebida contra a configuração antes de gravar. A forma Fiado não existe (RGN-002).

**Rationale**: CT-003 exige que apenas as formas habilitadas apareçam e RF-052 torna as formas configuráveis. A lista deve refletir a configuração vigente (spec.md Edge Cases), inclusive entre a abertura da tela e a confirmação.

**Alternatives considered**: Formas fixas no código - rejeitado por violar RF-052 (configurável, nunca hardcoded - CONVENTIONS §8); validar apenas na tela - rejeitado porque a confirmação pode ocorrer após mudança de configuração.

---

## Decisão 6: Registro de data/hora, usuário e movimentação de saída de cheios

**Decision**: A venda grava `data_hora` (servidor, fuso Brasília) e `id_usuario` (autenticado, RNF-006), e cada item gera movimentação SAIDA_CHEIO em `tab_movimentacao_estoque` com `id_referencia` = id da venda, na mesma transação.

**Rationale**: CT-006 exige data/hora e usuário responsável; RNF-007 exige rastro (quem, quando) e nenhuma venda pode ser apagada; a movimentação com referência dá base ao balanço (RF-043) e ao cancelamento com reversão futura (RGN-007).

**Alternatives considered**: Sem movimentação individual (só saldo) - rejeitado por perder o histórico de 12 meses (RNF-010); usuário como texto livre - rejeitado por violar RNF-006 (papéis e responsabilidade rastreável).

---

## Consistência com as decisões globais

Todas as decisões seguem as DECISÕES TÉCNICAS do projeto: transação única estoque + caixa com lock pessimista por produto, validação de saldo antes de gravar, cálculo de total e acréscimo no Service antes de persistir, registro de movimentação de estoque, sem DELETE de venda e com erros estruturados em pt-BR. As HU-008 (taxa de entrega), HU-009 (troca de vasilhame) e HU-010 (vasilhame novo) complementam esta base no mesmo endpoint `POST /api/vendas`, cada uma cobrindo apenas seu escopo de CTs.
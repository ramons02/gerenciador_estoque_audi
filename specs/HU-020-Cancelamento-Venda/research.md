# Research - Fase 0: Cancelamento/Estorno de Venda

**HU**: HU-020 - Cancelamento/Estorno de Venda
**Fase**: 0 - Pesquisa e decisões técnicas
**Data**: 2026-08-20

## Decision 1: Cancelamento em transação única com lock pessimista por produto

**Decision**: O `CancelamentoVendaService` é `@Transactional`: valida a venda, consulta os saldos com `@Lock(LockModeType.PESSIMISTIC_WRITE)` por produto dos itens, reverte o estoque e grava o status CANCELADA na mesma transação (RGN-007, CT-001). Se qualquer passo falhar, nada é gravado (RNF-005).

**Rationale**: A RGN-007 exige reversão automática de estoque e caixa; a CONVENTIONS §5.1 manda que toda movimentação aconteça na mesma transação do registro; o lock por produto (CONVENTIONS §8) serializa a concorrência e impede estoque negativo (RNF-008, RDN-005). Estado parcial é defeito grave (Constituição §III.3).

**Alternatives considered**: Reverter estoque e marcar a venda em transações separadas. Rejeitada: violaria RNF-005 e a Constituição §III.3 (venda cancelada com estoque não revertido ou vice-versa). Reverter sem lock confiando em UPDATE com condição: rejeitada, não serializa leitura e gravação de saldos sob concorrência (RNF-008).

## Decision 2: Reversão simétrica do estoque conforme o tipo de venda

**Decision**: A reversão repõe exatamente o que a venda movimentou (FR-001): venda com troca devolve 1 cheio ao estoque e retira 1 vazio do pátio (RF-023/RF-025, RDN-004); venda de vasilhame novo ou avulsa (sem devolução de vazio) devolve o casco de "em rua" ao estoque de cheios (RF-024/RF-026, RDN-008, Edge Case do spec). A regra é isolada em `ReversaoEstoqueService` (lógica pura, sem mock).

**Rationale**: A FR-001 e o Edge Case do spec exigem a reversão simétrica; o tipo da movimentação da venda original define a reversão. Isolar em serviço de lógica pura permite teste direto (Constituição §XI.2).

**Alternatives considered**: Reverter apenas cheios em todos os casos. Rejeitada: deixaria o pátio e o "em rua" divergentes (RDN-002/004/008). Reversão manual pelo administrador: rejeitada, a RGN-007 exige reversão automática.

## Decision 3: Auditoria com quem, quando e o motivo no próprio registro

**Decision**: O cancelamento preenche no registro da venda: `status = CANCELADA`, `motivo_cancelamento` (obrigatório), `data_hora_cancelamento` e `id_usuario_cancelamento` (do usuário autenticado) (FR-003/FR-004, CT-003). O registro permanece no histórico (RNF-007, CT-002); nenhum dado é apagado.

**Rationale**: A Constituição §XIII.2 e a RNF-006/007 exigem rastro de quem, quando e motivo, sem exceção; manter no próprio registro da venda garante que histórico e detalhe do cancelamento ficam juntos (CT-002) e evita tabela de log paralela que poderia divergir.

**Alternatives considered**: Tabela de log de auditoria separada para cancelamentos. Rejeitada: duplicaria o estado do cancelamento e arriscaria divergência com o registro da venda; os campos na venda já atendem RNF-007 e CT-003. Motivo opcional: rejeitada, a auditoria §XIII.2 exige o motivo.

## Decision 4: Reversão do caixa implícita pela exclusão de vendas CANCELADA nas somas

**Decision**: Não existe livro caixa por venda: o caixa é sempre derivado das vendas ATIVA do dia (dashboard RF-050/RF-051, relatório RF-041, fechamento RGN-006). Ao marcar a venda como CANCELADA na mesma transação, ela sai automaticamente de todas as somas, estornando o caixa conforme a forma de pagamento original (FR-002, CT-004).

**Rationale**: A RGN-007 exige o estorno automático; como o caixa é derivado (RGN-008), o estorno é consequência direta do status CANCELADA gravado na transação única (Decision 1), sem registro financeiro paralelo que pudesse divergir das vendas (RF-043, FR-005).

**Alternatives considered**: Gravar um registro de estorno financeiro (ex.: movimento de caixa negativo) além do status. Rejeitada: criaria duas fontes de verdade para o caixa do dia e permitiria divergência entre o estorno e a venda (RGN-008).

## Decision 5: Recusa de cancelamento duplicado e de venda inexistente

**Decision**: O cancelamento é recusado com erro claro quando a venda não existe (404) ou já está CANCELADA (409) (FR-006, Edge Case do spec). A validação ocorre no início da transação, antes de qualquer alteração de saldo.

**Rationale**: A FR-006 e o Edge Case do spec exigem recusa de cancelamento de venda já cancelada; validar no início da transação evita reversão dupla de estoque (que geraria estoque indevido) e mantém a mensagem clara em pt-BR (CONVENTIONS §6).

**Alternatives considered**: Ignorar silenciosamente a tentativa de cancelamento duplicado. Rejeitada: esconderia do usuário que a operação não foi aplicada; erro explícito é o comportamento exigido (FR-006).

## Decision 6: Estorno registrado como movimentação de estoque própria, fora das somas do balanço

**Decision**: A reversão grava em `tab_movimentacao_estoque` registros com tipo `ESTORNO_CHEIO`, `ESTORNO_VAZIO` ou `ESTORNO_EM_RUA`, referenciando a venda cancelada (FR-001, RF-043). O balanço (HU-018) e o relatório de vendas (HU-016) não somam estornos como entradas ou vendas (FR-005).

**Rationale**: O rastro da reversão precisa existir como movimentação (RNF-007, RNF-010), mas não pode distorcer o balanço: as colunas (+) Entradas e (-) Vendas contam apenas ENTRADA_CHEIO e SAIDA_CHEIO de carregamentos e vendas ATIVA (RF-043). Tipos próprios mantêm o balanço correto e o histórico completo.

**Alternatives considered**: Gravar a reversão como ENTRADA_CHEIO comum. Rejeitada: inflaria o balanço como se fosse entrada de caminhão (RF-043, FR-005). Não gravar movimentação de estorno: rejeitada, perderia o rastro de auditoria (RNF-007).

## Decision 7: Testes de reversão, concorrência e auditoria com prioridade máxima

**Decision**: O `ReversaoEstoqueServiceTest` cobre a reversão simétrica para troca e vasilhame novo (lógica pura, sem mock); o `CancelamentoVendaServiceTest` cobre duplicado, motivo obrigatório, venda inexistente e gravação da auditoria; o `CancelamentoVendaIntegrationTest` roda contra o banco real e inclui cancelamento sob concorrência com venda simultânea, provando que a reversão não gera estoque negativo (RNF-008) nem estado parcial (RNF-005).

**Rationale**: A CONVENTIONS §11 dá prioridade máxima às regras de estoque e caixa (reversão, transação, bloqueio por saldo) e exige teste de concorrência (RNF-008); a Constituição §XI.2 manda testar lógica pura sem mocks. O teste de integração prova a transação atômica contra dados reais (Constituição §IV).

**Alternatives considered**: Validar o cancelamento apenas manualmente pela tela. Rejeitada: a reversão atômica e a concorrência são críticas e exigem prova contínua automatizada.
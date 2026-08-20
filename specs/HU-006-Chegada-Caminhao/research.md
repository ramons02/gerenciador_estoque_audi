# Research: HU-006 - Registro de Chegada de Caminhão (Entrada)

**HU**: HU-006 | **Feature**: Chegada de Caminhão (Carregamento) | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Requisitos vinculados**: RF-010, RF-011, RF-012, RDN-003, RDN-009, RNF-005, RNF-007, RNF-008

Fase 0 - decisões de design registradas antes de qualquer código. Cada decisão referencia a convenção, constituição ou requisito que a justifica.

---

## Decisão 1: Carregamento persistido em transação única com a movimentação de estoque

**Decision**: A confirmação do carregamento grava `tab_carregamento`, `tab_carregamento_item`, as movimentações em `tab_movimentacao_estoque` e a atualização dos saldos de cheios e vazios em uma única transação `@Transactional` no Service.

**Rationale**: A chegada de caminhão envolve dois fluxos opostos na mesma operação (entram cheios, saem vazios devolvidos - RDN-009). Qualquer atualização de estoque fora da transação do registro é defeito grave (CONVENTIONS §5.1, Constituição §III.3) e o estado parcial corromperia o estoque permanentemente (RNF-005).

**Alternatives considered**: Transações separadas (registro primeiro, estoque depois) - rejeitada por permitir estado parcial; trigger no banco para atualizar estoque - rejeitada por esconder a regra de negócio do código e dificultar o rastro.

---

## Decisão 2: Validação da devolução de vazios sob lock pessimista por produto

**Decision**: Ao validar a devolução de vazios contra o saldo do pátio, o Repository consulta o saldo com `@Lock(LockModeType.PESSIMISTIC_WRITE)` dentro da mesma transação da confirmação.

**Rationale**: O saldo de vazios é contabilmente finito (RDN-003) e vendas simultâneas (trocas que adicionam vazios) ou outras confirmações concorrentes não podem gerar saldo negativo (RNF-008). O lock serializa a leitura e a escrita na mesma transação (CONVENTIONS §5.3 e §8).

**Alternatives considered**: Validar sem lock e confiar no UPDATE condicional - rejeitada por janela de corrida entre leitura e escrita; validação apenas na tela - rejeitada porque o banco é a fonte da verdade (Constituição §IV.1).

---

## Decisão 3: Valor unitário calculado no Service, com arredondamento e proteção de divisão por zero

**Decision**: O `CarregamentoService` calcula `valor_unitario = custo_total ÷ qtd_cheios_entraram` com arredondamento de 2 casas decimais; quantidade de cheios igual a zero é rejeitada na validação antes de qualquer cálculo.

**Rationale**: O cálculo é regra de negócio (CT-001, FR-002), não pode depender do cliente nem da tela. O edge case do spec exige tratar divisão por zero e arredondar custos não divisíveis exatos (spec.md, Edge Cases).

**Alternatives considered**: Calcular no frontend - rejeitado por permitir divergência de valor gravado; aceitar divisão por zero - rejeitado por gerar valor infinito/erro de estado de negócio (Constituição §XI.3).

---

## Decisão 4: Custo médio recalculado no mesmo commit da entrada

**Decision**: Ao confirmar o carregamento, o Service recalcula e persiste o custo médio do produto na mesma transação, usando o custo unitário da nova carga combinado ao saldo de cheios resultante.

**Rationale**: RF-012 exige recálculo após cada carregamento e CONVENTIONS §5.5 determina que o custo médio seja recalculado no mesmo commit da entrada, para apuração de custo e margem sem estados intermediários.

**Alternatives considered**: Recalcular sob demanda ao gerar relatórios - rejeitado por divergir da convenção e dificultar auditoria; cálculo no relatório apenas - rejeitado porque o custo médio é atributo de produto (apuração contábil em qualquer momento).

---

## Decisão 5: Registro de movimentação de estoque com referência ao carregamento

**Decision**: Cada confirmação gera registros em `tab_movimentacao_estoque` (tipo ENTRADA_CHEIO com quantidade N e tipo SAIDA_VAZIO com quantidade M, ambos com `id_referencia` = id do carregamento), na mesma transação.

**Rationale**: Nenhuma entrada pode ser apagada, apenas estornada com rastro (RNF-007, Constituição §III.4). A movimentação com `id_referencia` permite reconstituir o histórico, validar o balanço (RF-043) e dar base à auditoria de cancelamento futuro.

**Alternatives considered**: Confiar apenas nos saldos atuais de `tab_estoque` - rejeitado por perder o histórico exigido em RNF-010 (retenção de 12 meses); rastro em tabela de log genérica sem referência - rejeitado por não permitir ligar a movimentação ao documento de origem.

---

## Decisão 6: Bloqueio de devolução de vazios com erro estruturado em pt-BR

**Decision**: Quando a devolução excede o saldo do pátio, o Service lança exceção de negócio com mensagem padronizada (ex.: "Devolução de 15 vazios excede o saldo do pátio. Disponível: 10."), respondida como JSON estruturado com status de erro de validação.

**Rationale**: CT-002 exige bloqueio com indicação clara e CONVENTIONS §6 define mensagens de erro em pt-BR e respostas estruturadas, nunca texto puro/HTML. Erro de validação não pode colapsar em erro de sistema (Constituição §XI.3).

**Alternatives considered**: Retornar 500 com mensagem genérica - rejeitado por violar §XI.3; mensagem em inglês/técnica - rejeitado por violar CONVENTIONS §6 e o vocabulário do domínio (§II).

---

## Consistência com as decisões globais

Todas as decisões acima seguem as DECISÕES TÉCNICAS do projeto: transação única estoque + caixa com lock pessimista por produto, validação de saldo antes de gravar, cálculo de total/valor no Service, registro de movimentação de estoque, sem DELETE de movimento e com erros estruturados em pt-BR. O fluxo complementar (vendas que consomem o cheio recebido e devolvem vazios ao pátio) é tratado nas HU-007, HU-009 e HU-010, com a mesma base de dados e as mesmas regras de transação.
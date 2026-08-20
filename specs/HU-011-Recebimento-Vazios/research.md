# Research - Fase 0: Recebimento de Vasilhames Vazios Avulsos

**HU**: HU-011 - Recebimento de Vasilhames Vazios Avulsos
**Fase**: 0 - Pesquisa e decisões técnicas
**Data**: 2026-08-20

## Decision 1: Saldo de vazios persistido em tab_estoque, atualizado na mesma transação do lançamento

**Decision**: O saldo de vazios (e de em rua) é mantido materializado em `tab_estoque`, atualizado atomicamente na transação que grava o lançamento de devolução em `tab_devolucao_vazio` e a movimentação em `tab_movimentacao_estoque`.

**Rationale**: A Constituição §III.3 e a CONVENTIONS §5.1 exigem que toda movimentação altere estoque na MESMA transação do registro do movimento; a CONVENTIONS §8 exige lock por produto ao consultar e atualizar na mesma transação (RNF-008). Com saldo materializado, a leitura do painel (HU-012) e a validação de saldo (HU-013) são imediatas e consistentes.

**Alternatives considered**: Derivar o saldo por soma das linhas de `tab_movimentacao_estoque` a cada consulta. Rejeitada: custo de varredura cresce com 12 meses de histórico (RNF-010), leitura do painel ficaria lenta e o cálculo sob concorrência dificultaria a invariante de saldo nunca negativo (RDN-005).

## Decision 2: Lock pessimista por produto ao ler o saldo dentro da transação

**Decision**: Ao confirmar um recebimento, o Service consulta `tab_estoque` do produto com `@Lock(LockModeType.PESSIMISTIC_WRITE)` antes de incrementar o saldo.

**Rationale**: Recebimentos e devoluções de venda (troca) podem ocorrer simultaneamente no mesmo produto com 1-5 usuários (RNF-010); a CONVENTIONS §8 determina lock por produto para garantir as invariantes sob concorrência (RNF-008). O lock serializa leitura e atualização dentro da mesma transação.

**Alternatives considered**: Update condicional com `WHERE qtd >= 0`. Rejeitada: não protege a sequência ler-calcular-gravar no caso de operações combinadas (vazios + em rua) e pode falhar silenciosamente sem mensagem clara (CONVENTIONS §6).

## Decision 3: Baixa do comodato limitada ao saldo em rua do cliente, excedente só entra no pátio

**Decision**: Quando o lançamento é vinculado a um cliente, a baixa de `qtd_em_rua` do cliente (tab_cliente_em_rua) é limitada ao saldo existente: se a quantidade devolvida N excede o saldo em rua M do cliente, baixa apenas M (sem gerar negativo, RDN-005) e o total N entra integralmente no pátio de vazios.

**Rationale**: É o comportamento definido nos Edge Cases do spec: "a baixa do comodato não deve gerar saldo em rua negativo; o excedente apenas entra no pátio". Também cobre o CT-003 (baixa quando há comodato) e o caso de cliente sem comodato (baixa zero, só pátio).

**Alternatives considered**: Bloquear a devolução quando a quantidade excede o comodato do cliente. Rejeitada: contraria o spec (o excedente deve ser aceito no pátio) e impediria devoluções legítimas de vazios sem vínculo de comodato registrado.

## Decision 4: Cliente opcional no lançamento, validado apenas quando informado

**Decision**: O campo `idCliente` é opcional no request (CT-001); quando informado, o sistema valida a existência do cliente (RF-004) e aplica a baixa de comodato (RF-028). Quando ausente, apenas o pátio de vazios é incrementado.

**Rationale**: A HU exige cliente opcional (CT-001) e a CONVENTIONS §6 manda validação de negócio com erro claro em pt-BR. O vínculo posterior não é necessário porque a devolução avulsa é um lançamento de pátio, não de contrato.

**Alternatives considered**: Exigir cliente sempre para permitir baixa de comodato. Rejeitada: contraria o CT-001 e o Edge Case "lançamento sem cliente informado não deve perder a possibilidade de vinculação posterior". Exigir venda associada: rejeitada, contraria RF-027 (devolução fora de venda).

## Decision 5: Data/hora e usuário gerados pelo servidor no lançamento

**Decision**: `data_hora` é gerada pelo servidor (timestamp da transação) e `id_usuario` vem do usuário autenticado na sessão, gravados em `tab_devolucao_vazio` (CT-004).

**Rationale**: O CT-004 exige data/hora e usuário em todo lançamento; a CONVENTIONS §6 e a RNF-006/007 exigem rastro de auditoria com quem e quando. Gerar no servidor evita relógio de cliente divergente e fraude de horário.

**Alternatives considered**: Data/hora enviada pelo cliente no request. Rejeitada: permite divergência de relógio e adulteração do rastro de auditoria (RNF-007). Usuário opcional: rejeitada, contraria CT-004.

## Decision 6: Movimentação de estoque com tipo ENTRADA_VAZIO e referência ao lançamento

**Decision**: O recebimento gera linha em `tab_movimentacao_estoque` com `tipo = ENTRADA_VAZIO`, quantidade N, `id_referencia` apontando para o id em `tab_devolucao_vazio`; quando há baixa de comodato, gera também `tipo = SAIDA_EM_RUA` com a quantidade baixada. O vasilhame devolvido entra como vazio, nunca como cheio (Edge Case do spec).

**Rationale**: A tabela canônica de movimentações (definida nas decisões de arquitetura do projeto) é a base do balanço (RF-043) e do painel (RF-030); o `id_referencia` dá rastreabilidade total do movimento (RNF-007). O tipo distinto garante que o estado do produto não é confundido (Edge Case).

**Alternatives considered**: Reaproveitar tipo de venda com flag "avulsa". Rejeitada: mistura conceitos e quebra o balanço (RF-043) e o vocabulário do domínio (Constituição §II).

## Decision 7: Endpoint POST /api/devolucoes no módulo devolucao, resposta 201 com saldos

**Decision**: A API expõe `POST /api/devolucoes` (e GET para consulta de auditoria) no módulo `com.gerenciador.estoque.devolucao`, com rotas `/api/<modulo>` em minúsculo sem acento (CONVENTIONS §4/§6). A resposta 201 devolve o lançamento e os saldos atualizados (pátio, em rua global e do cliente).

**Rationale**: As decisões técnicas do projeto definem módulo por feature com Controller/Service/Repository/dto e respostas estruturadas em pt-BR (CONVENTIONS §6). Retornar os saldos atualizados evita nova chamada do app e permite confirmar o CT-002 na tela.

**Alternatives considered**: Endpoint dentro do módulo `estoque` (ex.: POST /api/estoque/devolucoes). Rejeitada: as rotas definidas no plano do projeto fixam `/api/devolucoes` para recebimento de vazios, mantendo /api/estoque somente para leitura de saldos e alertas.

## Decision 8: Teste de prioridade máxima para a regra de pátio nunca negativo

**Decision**: A regra de baixa de comodato limitada ao saldo existente (nunca negativo) é testada com lógica pura (sem mock) em ServiceTest e com teste de integração @SpringBootTest cobrindo o fluxo completo (lançamento + estoque + movimentação na mesma transação).

**Rationale**: A Constituição §XI.2 e a CONVENTIONS §11 dão prioridade máxima de cobertura às regras de estoque; a regra de reversão/limite de comodato é regra de estoque. Teste de integração prova a atomicidade da transação (RNF-005).

**Alternatives considered**: Testar apenas o Controller com mocks. Rejeitada: não prova a regra de negócio nem a atomicidade (CONVENTIONS §11: não usar mocks para lógica pura).
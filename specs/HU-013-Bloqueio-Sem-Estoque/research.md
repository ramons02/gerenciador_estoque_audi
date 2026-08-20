# Research - Fase 0: Bloqueio de Venda sem Estoque

**HU**: HU-013 - Bloqueio de Venda sem Estoque
**Fase**: 0 - Pesquisa e decisões técnicas
**Data**: 2026-08-20

## Decision 1: Validação de saldo no Service com lock pessimista por produto

**Decision**: O `VendaService` é `@Transactional` e, ao gravar cada item, consulta `tab_estoque` do produto com `@Lock(LockModeType.PESSIMISTIC_WRITE)`, compara a quantidade solicitada com `qtd_cheios` e só então baixa o saldo. Qualquer leitura e escrita de saldo acontecem na MESMA transação (CONVENTIONS §8).

**Rationale**: A CONVENTIONS §8 e a RNF-008 exigem lock por produto para impedir estoque negativo sob vendas simultâneas; a invariante "estoque nunca negativo" (RDN-005) é prioridade máxima de cobertura (CONVENTIONS §11). O lock serializa as transações do mesmo produto, tornando o CT-002 (vendas simultâneas) garantido.

**Alternatives considered**: Validar no Controller antes da transação (leitura sem lock) e baixar depois. Rejeitada: entre a leitura e a gravação outra venda poderia consumir o saldo, furando o bloqueio (CT-002). Validar no frontend apenas: rejeitada, nunca é confiável e contraria a fonte da verdade de dados (Constituição §IV).

## Decision 2: Erro estruturado 409 ESTOQUE_INSUFICIENTE com mensagem pt-BR

**Decision**: Quando a quantidade excede `qtd_cheios`, a API retorna HTTP 409 com corpo `{ "codigo": "ESTOQUE_INSUFICIENTE", "mensagem": "Estoque insuficiente para 3 unidade(s) de Gás P13. Disponível: 1." }`, sem baixar estoque e sem registrar venda.

**Rationale**: A CONVENTIONS §6 fixa resposta de erro estruturada com mensagem em pt-BR (exemplo idêntico de "Estoque insuficiente para N unidade(s) de <produto>. Disponível: X."); a Constituição §XI.3 manda que erro de validação não colapse em estado de negócio (bloqueio de venda não pode virar "sistema indisponível"). 409 expressa conflito com o estado atual do estoque.

**Alternatives considered**: Retornar 400 genérico ou 500. Rejeitada: 400 confunde validação de formulário com conflito de negócio; 500 simularia indisponibilidade (Constituição §XI.3). Mensagem sem detalhe ("Estoque insuficiente"): rejeitada, a CONVENTIONS §6 exige o detalhe com saldo disponível.

## Decision 3: Validação centralizada única para todos os fluxos de venda

**Decision**: O bloqueio é implementado em um único ponto do Service de venda (validação do item antes da baixa), usado igualmente pelos fluxos balcão, entrega, troca de vasilhame e venda de vasilhame novo (CT-003). O campo `tipoOperacao` do request (NORMAL, TROCA, VASILHAME_NOVO, AVULSA) não altera a regra de bloqueio por cheios.

**Rationale**: O CT-003 exige que o bloqueio valha para qualquer fluxo; centralizar evita brecha por fluxo esquecido (spec: "inconsistência entre fluxos criaria brechas") e atende à regra "nenhuma regra de negócio inventada no código" (CONVENTIONS §12, RF-031).

**Alternatives considered**: Validar em cada fluxo separadamente (um método por tipo de venda). Rejeitada: duplicaria a regra e facilitaria divergência entre fluxos (CT-003).

## Decision 4: Quantidade igual ao saldo é permitida; bloqueio apenas quando excede

**Decision**: A condição de bloqueio é `quantidade > qtd_cheios`; quantidade igual ao saldo é aceita (Edge Case do spec).

**Rationale**: O Edge Case do spec é explícito: "Quantidade solicitada igual ao saldo de cheios disponível deve ser permitida (bloqueio apenas quando a quantidade é maior que o saldo)". Reflete a operação real da portaria (vender o último cheio é permitido).

**Alternatives considered**: Bloquear também quando igual (estoque zerado após a venda). Rejeitada: contraria o Edge Case e o RNF-001 (lançamento rápido não pode ser frustrado por regra mais rígida que a definida).

## Decision 5: Estoque, movimentação e caixa revertidos juntos no cancelamento

**Decision**: O cancelamento de venda (RGN-007, RNF-007) reverte na mesma transação: status da venda para "cancelada", movimentações reversas (ENTRADA_CHEIO, SAIDA_VAZIO, SAIDA_EM_RUA conforme o tipo) e saldo em `tab_estoque`, devolvendo a capacidade de venda (Edge Case: "venda cancelada deve devolver o estoque ao saldo").

**Rationale**: A RGN-007 e a RNF-007 exigem reversão automática com rastro e status; a Constituição §VI.1 proíbe apagar movimento. Reverter o saldo permite nova venda dentro do que foi liberado (Edge Case do spec) e mantém a consistência do balanço (RF-043).

**Alternatives considered**: Criar venda nova negativa em vez de cancelar. Rejeitada: polui o histórico e dificulta o fechamento de caixa (RGN-006); o status "cancelada" com reversão é o padrão definido.

## Decision 6: Teste de concorrência com threads para provar RNF-008

**Decision**: O `VendaConcorrenciaIntegrationTest` (teste de integração com @SpringBootTest) dispara vendas simultâneas do mesmo produto cuja soma excede o saldo e asserta que nenhuma venda gera estoque negativo e apenas as vendas que cabem no saldo são aprovadas (CT-002).

**Rationale**: A CONVENTIONS §11 exige teste de concorrência ("vendas simultâneas não podem gerar estoque negativo - RNF-008") e prioridade máxima para regra de estoque; a prova de concorrência só é válida em teste de integração contra o banco real, não com mocks (CONVENTIONS §11).

**Alternatives considered**: Simular concorrência com mocks no ServiceTest. Rejeitada: mocks não reproduzem o comportamento real do lock no PostgreSQL; a prova seria falsa (Constituição §IV).

## Decision 7: Bloqueio considera apenas o saldo de cheios

**Decision**: A condição de bloqueio usa somente `qtd_cheios`; falta de vazios no pátio ou de vasilhames em rua nunca bloqueia a venda (Edge Case do spec).

**Rationale**: O spec é explícito: "O bloqueio não deve ocorrer por falta de vazios ou de vasilhames em rua - apenas pela falta de cheios". A RF-031 define o bloqueio por estoque de cheios.

**Alternatives considered**: Bloquear também quando não há vazios para troca. Rejeitada: contraria o Edge Case e a RF-031; a troca sem vazio devolvido é tratada como venda de vasilhame novo (RF-024/RF-026).
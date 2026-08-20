# Research - Fase 0: Dashboard (Resumo do Dia)

**HU**: HU-019 - Dashboard (Resumo do Dia)
**Fase**: 0 - Pesquisa e decisões técnicas
**Data**: 2026-08-20

## Decision 1: Endpoint único de resumo do dia com agregação no servidor

**Decision**: O dashboard é alimentado por um único endpoint `GET /api/dashboard/resumo-dia` que retorna, em uma resposta, o total faturado, os totais por forma de pagamento, as unidades vendidas por produto e o total geral (RF-050 a RF-053, CT-001 a CT-003). A agregação roda no servidor sobre `tab_venda`/`tab_venda_item`.

**Rationale**: A tela inicial precisa do resumo completo de um relance (RF-050 a RF-053); agregar no servidor mantém a fonte da verdade no banco (Constituição §IV) e evita múltiplas requisições (RNF-001: poucos passos, resposta rápida em até 2 segundos - RNF-003).

**Alternatives considered**: Um endpoint por card (faturamento, formas, unidades, alertas). Rejeitada: quatro requisições por abertura da tela inicial, maior latência e estados parciais de carregamento (RNF-003). Agregar no app a partir do relatório de vendas: rejeitada, duplicaria a lógica de agregação e divergiria do relatório (SC-002).

## Decision 2: Cartão como forma única somando crédito e débito

**Decision**: O total por forma de pagamento é agregado pelos valores gravados em `tab_venda.forma_pagamento` (DINHEIRO, PIX, CARTAO). Crédito e débito não se distinguem na venda: ambos são gravados como CARTAO (RF-021, RF-021-A), portanto a soma do dia é única (RF-051, CT-002).

**Rationale**: A RF-021 define Cartão (Crédito/Débito) como forma única com acréscimo fixo (aplicado somente a produtos de carga Gás, RF-021-A); a RF-051 exige crédito e débito somados em um único valor. Gravar a forma única já atende o requisito sem campo extra nem conversão na leitura (RGN-002).

**Alternatives considered**: Gravar CARTAO_CREDITO e CARTAO_DEBITO e somar na consulta. Rejeitada: criaria campo fora do modelo de RF-021 e complexidade desnecessária, já que RF-051 soma os dois de qualquer forma.

## Decision 3: Unidades vendidas por produto e total geral a partir dos itens

**Decision**: As unidades vendidas do dia são a soma de `quantidade` em `tab_venda_item` das vendas ATIVA do dia, agrupada por produto, com total geral (RF-052, CT-003). O rótulo usa o vocabulário canônico: vasilhames vendidos (Constituição §II); o RF-052 original cita "botijões/galões", termos não canônicos para carga/vasilhame.

**Rationale**: A RF-052 exige o total por produto e total geral; a quantidade vive no item da venda (RF-020, RF-025). Usar o termo canônico vasilhame mantém a linguagem única do projeto (Constituição §II) e a coluna Produto identifica a combinação carga + vasilhame (RF-001).

**Alternatives considered**: Repetir o texto "botijões/galões" do RF-052 na interface. Rejeitada: contraria o vocabulário obrigatório da Constituição §II e a CONVENTIONS §2. Somar somente o total financeiro e omitir unidades por produto: rejeitada, fere RF-052 (CT-003).

## Decision 4: Alertas de estoque baixo derivados do saldo materializado vs limite mínimo

**Decision**: Os alertas de estoque baixo são derivados na consulta: produtos com `tab_estoque.qtd_cheios <= limite_minimo` (RF-053, CT-004). O alerta usa o saldo materializado de cheios, o mesmo que alimenta o painel de estoque e o bloqueio de venda (RF-030/RF-031/RF-032).

**Rationale**: A RF-053 exige alertas dos produtos no limite mínimo ou abaixo; a RF-032 define o critério exato. Usar o saldo materializado evita recálculo por replay e mantém consistência com as demais telas (RGN-004: alerta persiste até nova entrada de caminhão).

**Alternatives considered**: Calcular o alerta reaplicando movimentações do dia. Rejeitada: custo desnecessário e risco de divergência com o painel de estoque (RF-030). Considerar também vazios/em rua no alerta: rejeitada, o critério de RF-032 é o saldo de cheios.

## Decision 5: Exclusão de vendas canceladas de todos os totais

**Decision**: Todas as agregações do dashboard filtram `status = ATIVA` na origem (FR-005, RGN-007): faturamento, formas de pagamento e unidades vendidas ignoram vendas CANCELADA. O cancelamento (HU-020) remove a venda das somas automaticamente, sem ação do dashboard.

**Rationale**: A FR-005 da spec exige que vendas canceladas não somem nos totais; o filtro na query garante consistência com relatórios e fechamento de caixa (SC-002, RGN-008) e com o cancelamento da HU-020.

**Alternatives considered**: Filtrar canceladas no app após receber os totais. Rejeitada: o app não teria como descontar o que já veio agregado, e duplicaria regra de negócio (CONVENTIONS §12).

## Decision 6: Dia sem vendas exibe valores zerados sem erro

**Decision**: Quando não há vendas ATIVA no dia, a resposta traz todos os totais zerados, unidades vazias (ou zeradas) e apenas os alertas ativos (RF-053), sem erro (Edge Case do spec). Formas de pagamento sem movimentação no dia são exibidas com valor zero (Edge Case do spec).

**Rationale**: O Edge Case do spec exige valores zerados, sem erro; manter o contrato estável permite que a tela inicial reaja normalmente em dias parados, com os alertas de reposição ainda visíveis (RF-053).

**Alternatives considered**: Retornar 404 ou erro quando o dia não tem vendas. Rejeitada: contraria o Edge Case e esconderia a tela inicial de dias sem movimento.

## Decision 7: Testes de agregação com conferência contra o relatório do dia

**Decision**: O `ResumoDiaServiceTest` cobre agregação por forma de pagamento, unidades por produto, exclusão de canceladas e alertas por limite mínimo (lógica pura, sem mock); o `ResumoDiaIntegrationTest` roda contra o banco real e confere os totais do dashboard com o relatório de vendas do mesmo dia (SC-002, RF-050 a RF-053).

**Rationale**: A Constituição §XI.2 manda testar lógica pura sem mocks; a CONVENTIONS §11 prioriza regras de estoque e caixa. A conferência com o relatório de vendas do dia (SC-002) prova a consistência entre as telas.

**Alternatives considered**: Validar o dashboard apenas manualmente. Rejeitada: a conferência de caixa (RGN-008) é crítica e exige prova contínua; o teste de integração a automatiza.
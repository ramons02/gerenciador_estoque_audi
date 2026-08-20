# Research - Fase 0: Painel de Estoque em Tempo Real (Pátio)

**HU**: HU-012 - Painel de Estoque em Tempo Real (Pátio)
**Fase**: 0 - Pesquisa e decisões técnicas
**Data**: 2026-08-20

## Decision 1: Saldos lidos do estado materializado em tab_estoque, sem soma de movimentações

**Decision**: O painel lê os saldos de Cheios, Vazios e Em rua diretamente de `tab_estoque` (estado materializado mantido pelas features de escrita), com junção a `tab_produto` para o nome.

**Rationale**: A CONVENTIONS §8 fixa o saldo materializado com lock por produto nas transações de escrita; o painel (RF-030) precisa de leitura rápida em até 1 segundo (SC-002) mesmo com 12 meses de movimentações (RNF-010). O saldo materializado é a fonte da verdade de exibição, alinhado ao que HU-013 e HU-014 validam.

**Alternatives considered**: Derivar os saldos somando `tab_movimentacao_estoque` por produto a cada consulta. Rejeitada: custo cresce com o histórico (RNF-010), contraria o desempenho do SC-002 e duplica a lógica de cálculo já centralizada nas escritas.

## Decision 2: Atualização imediata por reconsulta automática após mutações e polling curto

**Decision**: No app, o hook `useSaldosEstoque` dispara reconsulta de `GET /api/estoque` automaticamente após qualquer mutação confirmada (venda, entrada, devolução) e, adicionalmente, em intervalo periódico curto (30 segundos), sem recarga manual da tela (CT-002).

**Rationale**: Com 1-5 usuários simultâneos (RNF-010), reconsulta após mutação própria garante o reflexo imediato (< 1s, SC-002) e o polling cobre mutações de outros terminais. A CONVENTIONS §7 exige consumo via cliente HTTP tipado, o que o hook centraliza.

**Alternatives considered**: SSE ou WebSocket para push de atualizações. Rejeitada: complexidade de infraestrutura e conexão persistente sem ganho para 1-5 usuários; polling simples atende ao SC-002. Recarregar a página manualmente: rejeitada, contraria explicitamente o CT-002.

## Decision 3: Destaque de estoque baixo como flag calculada na API (saldo <= limite mínimo)

**Decision**: A API calcula e devolve `alertaEstoqueBaixo` por produto comparando `qtd_cheios` com `limite_minimo` (<= dispara alerta, conforme RF-032 e RGN-004), e o app renderiza o destaque visual a partir dessa flag.

**Rationale**: O alerta é uma regra de negócio (RF-032) que precisa de fonte única: o mesmo cálculo serve o painel (CT-003), o dashboard (RF-053) e a HU-014. Manter o cálculo na API evita divergência de regra entre telas e frontends.

**Alternatives considered**: Calcular o destaque no app com lógica local. Rejeitada: duplicaria a regra (RF-032) em cada tela e permitiria divergência; a CONVENTIONS §12 manda a regra nascer dos requisitos e ser centralizada. Persistir alertas em tabela própria: rejeitada na HU-014 (estado derivado duplicado).

## Decision 4: Produto sem limite mínimo configurado nunca gera alerta

**Decision**: Quando `limite_minimo` é nulo (não configurado, RF-003), `alertaEstoqueBaixo` é sempre falso, independentemente do saldo.

**Rationale**: Edge Case do spec: "produto recém-cadastrado, sem limite mínimo configurado, não deve gerar falso alerta de estoque baixo". A configuração vem de outra feature (HU-003/RF-003); o painel apenas consome (Assumption do spec).

**Alternatives considered**: Considerar limite zero como gatilho (saldo 0 <= 0). Rejeitada: geraria alerta falso para produtos sem configuração, contrariando o Edge Case.

## Decision 5: Produto sem movimentação aparece no painel, inclusive com saldos zerados

**Decision**: A consulta parte de `tab_produto` (junção com `tab_estoque`), garantindo que todo produto ativo aparece com seus saldos, inclusive zeros (Edge Case do spec).

**Rationale**: O Edge Case exige que produto sem movimentações no dia apareça com saldos atuais (ou zerados). Partir do produto ativo também deixa a visão completa para o administrador decidir reposição (RF-030).

**Alternatives considered**: Partir de `tab_estoque` (só produtos com linha de saldo). Rejeitada: produto recém-cadastrado sem linha de estoque sumiria do painel, contrariando o Edge Case.

## Decision 6: Endpoints de leitura no módulo estoque, sem escrita nesta feature

**Decision**: A feature expõe somente `GET /api/estoque` e `GET /api/estoque/alertas` no módulo `com.gerenciador.estoque.estoque`; nenhuma escrita é feita aqui - os saldos são atualizados pelas features de venda, carregamento e devolução nas suas próprias transações (RNF-005).

**Rationale**: As decisões técnicas do projeto separam módulos por responsabilidade; o painel é leitura pura (RF-030). A invariante de transação única (Constituição §III.3) é preservada porque nenhuma atualização acontece fora das features de escrita.

**Alternatives considered**: Endpoint que recalcula e reescreve saldos na leitura. Rejeitada: violaria a separação leitura/escrita e poderia induzir atualizações fora de transação com registro de movimento (Constituição §VI.2).

## Decision 7: Testes de leitura com foco na regra de alerta e na junção de dados

**Decision**: A regra `alertaEstoqueBaixo` (saldo <= limite, nulo nunca alerta) é testada como lógica pura no ServiceTest; o IntegrationTest cobre a junção produto x estoque (produtos zerados presentes) e a consistência com o contrato.

**Rationale**: A Constituição §XI.2 manda testar lógica pura sem mocks; a regra de alerta é regra de estoque, com prioridade máxima de cobertura (CONVENTIONS §11). O teste de integração prova o comportamento real da query (Constituição §IV).

**Alternatives considered**: Testar apenas o Controller com mocks do Service. Rejeitada: não prova a regra nem a query de junção; seria cobertura falsa do CT-001/CT-003.
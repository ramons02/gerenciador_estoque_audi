# Research - Fase 0: Alerta de Estoque Baixo

**HU**: HU-014 - Alerta de Estoque Baixo
**Fase**: 0 - Pesquisa e decisões técnicas
**Data**: 2026-08-20

## Decision 1: Alerta como estado DERIVADO, nunca persistido em tabela própria

**Decision**: O alerta é calculado em leitura: `qtd_cheios <= limite_minimo` (com `limite_minimo` não nulo) no endpoint `GET /api/estoque/alertas`. Nenhuma tabela de alertas é criada.

**Rationale**: O alerta é função direta do saldo e do limite (RF-032, RGN-004). Como estado derivado, ele reflete imediatamente qualquer venda que derrube o saldo (Edge Case do spec) e some automaticamente quando uma entrada eleva o saldo acima do limite (CT-003) - sem job de sincronização e sem risco de divergência entre o alerta armazenado e o saldo real (fonte da verdade de dados, Constituição §IV).

**Alternatives considered**: Tabela `tab_alerta` com flag ativo/inativo atualizada nas transações de venda e entrada. Rejeitada: duplicaria o estado e criaria risco de divergência (ex.: entrada aplicada e flag não atualizada); a persistência exigiria regra de negócio extra não prevista nos requisitos (CONVENTIONS §12).

## Decision 2: Endpoint único de alertas consumido pelo dashboard e pelo painel

**Decision**: `GET /api/estoque/alertas` é a fonte única; o dashboard (RF-053, CT-002) e o painel de estoque (HU-012, CT-002) consomem o mesmo endpoint, sem lógica de alerta duplicada no app.

**Rationale**: A regra RF-032/RGN-004 deve ter uma única implementação (CONVENTIONS §12); exibir em dois lugares não justifica duas implementações. O endpoint compartilhado garante que o alerta seja idêntico nas duas telas (CT-002).

**Alternatives considered**: Cada tela calcula o alerta com lógica local no app. Rejeitada: duplicaria a regra e permitiria divergência (ex.: painel com alerta e dashboard sem); contraria a centralização da regra de negócio (CONVENTIONS §12).

## Decision 3: Sugestão de reposição pela média de vendas diárias dos últimos 30 dias

**Decision**: A sugestão de reposição (CT-004, RGN-009) é calculada como `mediaVendasDiarias = totalVendidoNosUltimos30Dias / 30`, com uma casa decimal; `sugestaoReposicao = max(1, arredondaParaCima(mediaVendasDiarias))`. Para produto sem histórico de vendas no período, `sugestaoReposicao = max(1, limiteMinimo - qtdCheios)`.

**Rationale**: A RGN-009 manda considerar a média de vendas diárias como sugestão de reposição; o período de 30 dias é uma janela estável que captura o ritmo de venda sem distorção sazonal pontual e cobre o histórico disponível (RNF-010, 12 meses). O arredondamento para cima garante sugestão de pelo menos 1 unidade (Edge Case: produto recém-cadastrado não inviabiliza a sugestão).

**Alternatives considered**: Sugerir exatamente o limite mínimo menos o saldo atual. Rejeitada: não usa a média de vendas diárias exigida pela RGN-009. Usar média do último mês calendário: rejeitada, mês parcial distorce o cálculo; janela deslizante de 30 dias é mais estável.

## Decision 4: Alerta dispara com saldo IGUAL ao limite ("atingir ou ficar abaixo")

**Decision**: A condição de alerta é `qtd_cheios <= limite_minimo`, incluindo a igualdade (Edge Case do spec e RF-032: "atingir ou ficar abaixo").

**Rationale**: RF-032 e o Edge Case do spec são explícitos: "Estoque exatamente igual ao limite mínimo deve disparar o alerta". O painel (HU-012) e o dashboard compartilham a mesma condição, mantendo consistência.

**Alternatives considered**: Disparar apenas abaixo do limite (estrito `<`). Rejeitada: contraria RF-032 ("atingir") e o Edge Case; o administrador perderia o aviso no exato momento em que o produto entra em zona de risco.

## Decision 5: Produto sem limite mínimo configurado nunca gera alerta

**Decision**: Quando `limite_minimo` é nulo (RF-003 não configurado), o produto é excluído do resultado de alertas.

**Rationale**: Edge Case do spec (tanto da HU-012 quanto da HU-014): "Produto sem limite mínimo configurado não deve gerar alerta (a configuração do limite é pré-requisito, RF-003)". A configuração vem de outra feature (HU-003); esta apenas consome (Assumption do spec).

**Alternatives considered**: Tratar limite ausente como zero (alerta quando saldo 0). Rejeitada: geraria alerta para produto recém-cadastrado sem configuração, contrariando o Edge Case e poluindo o dashboard com falso positivo (SC-003 da HU-014).

## Decision 6: Persistência do alerta garantida pela derivação, sem job de reativação

**Decision**: Nenhum job ou flag reativa o alerta: como ele é derivado do saldo (Decision 1), permanece visível enquanto `qtd_cheios <= limite_minimo` e some apenas quando uma entrada de cheios (carregamento, RF-011) eleva o saldo acima do limite (CT-003, RGN-004).

**Rationale**: A RGN-004 exige persistência "até a nova entrada de caminhão"; a derivação atende a isso sem mecanismo extra, reduzindo superfície de bug (ex.: flag presa em "ativo" após a entrada). A fonte da verdade é o dado (Constituição §IV).

**Alternatives considered**: Registrar data de ativação/desativação do alerta em tabela. Rejeitada: desnecessário para o comportamento exigido e contraria a Decision 1 (estado duplicado).

## Decision 7: Testes de prioridade máxima para a regra de alerta e da média de vendas

**Decision**: A regra de alerta (<= com limite nulo excluído) e o cálculo de média/sugestão são testados como lógica pura, sem mock (MediaVendasServiceTest, EstoqueAlertaServiceTest); o IntegrationTest cobre a consulta real sobre `tab_estoque` e `tab_venda_item` (produto sem histórico, saldo no limite, entrada elevando o saldo).

**Rationale**: Regra de estoque com prioridade máxima de cobertura (CONVENTIONS §11); lógica pura testada diretamente (Constituição §XI.2). O teste de integração prova o comportamento da query contra o banco real (Constituição §IV), incluindo o CT-003 (alerta some após entrada).

**Alternatives considered**: Testar apenas o Controller com mocks do Service. Rejeitada: não prova a regra de cálculo nem a persistência do alerta pela derivação; cobertura falsa do CT-001/CT-004.
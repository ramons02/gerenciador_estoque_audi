# Research - Fase 0: Fechamento de Caixa e Balanço de Estoque

**HU**: HU-018 - Fechamento de Caixa e Balanço de Estoque
**Fase**: 0 - Pesquisa e decisões técnicas
**Data**: 2026-08-20

## Decision 1: Fechamento de caixa com registro único por data em tab_fechamento_caixa

**Decision**: O fechamento do dia grava um registro em `tab_fechamento_caixa` com `data` única (constraint unique), totais por forma de pagamento (total_dinheiro, total_pix, total_cartao, total_geral), `status` (ABERTO/FECHADO), `data_hora_fechamento` e `id_usuario_fechamento`. O serviço é `@Transactional`: calcula os totais e grava o fechamento na mesma transação (RGN-006, CT-002).

**Rationale**: A RF-043 e a RGN-006 exigem um fechamento por dia com conferência do caixa; a unicidade por `data` impede duplicidade e o status permite saber se o dia já foi encerrado. A transação única garante que o fechamento nunca fica parcial (CONVENTIONS §5.1, Constituição §III.3).

**Alternatives considered**: Fechamento apenas calculado sob demanda, sem registro persistido. Rejeitada: sem registro não há histórico do encerramento do dia (RNF-010) nem como impedir fechamento duplicado. Um fechamento por venda individual: rejeitada, o fechamento é do dia inteiro (RF-043).

## Decision 2: Conciliação do dia como pré-condição do fechamento (RGN-006)

**Decision**: O `POST /api/caixa/fechar` recalcula, dentro da transação, a soma das vendas `status = ATIVA` do dia por forma de pagamento. Se existir venda do dia em estado não conciliado (nem ATIVA nem CANCELADA) ou o formulário de venda em edição estiver aberto no app, o fechamento é bloqueado com a lista das vendas pendentes (CT-002). O app também desabilita o botão "Fechar caixa" enquanto houver venda em edição.

**Rationale**: A RGN-006 define que o fechamento só conclui com todas as vendas do dia conciliadas; o spec exige aviso das vendas pendentes. Como a venda só é persistida quando confirmada (HU-007, transação atômica), "venda em aberto/em edição" é estado do formulário no app: o bloqueio de UI cobre o fluxo e o servidor rejeita qualquer divergência de integridade (CONVENTIONS §6).

**Alternatives considered**: Persistir rascunho de venda em banco com status EM_EDICAO. Rejeitada: criaria regra fora dos requisitos (CONVENTIONS §12) e contrariaria o lançamento rápido atômico (RNF-001, RNF-005). Fechar o dia sem verificar pendências: rejeitada, violaria RGN-006 diretamente.

## Decision 3: Balanço de estoque derivado das movimentações e do saldo materializado

**Decision**: No Balanço (RF-043, CT-001/CT-004), por produto: `Estoque Inicial = saldoAtual - (entradas - vendas no período)`, `Estoque Final = saldoAtual` (qtd_cheios de `tab_estoque`), `(+) Entradas` e `(-) Vendas` somados de `tab_movimentacao_estoque` no período, e `Vazios em Pátio = qtd_vazios` atual. Movimentações de estorno (cancelamento, HU-020) não entram nas colunas Entradas/Vendas.

**Rationale**: O saldo materializado é a fonte da verdade atual (CONVENTIONS §8); as movimentações do período dão a variação. Derivar o inicial pela diferença evita replay do histórico completo e mantém consistência com o painel e o alerta (RF-030/RF-032). O CT-004 exige o estoque inicial baseado em movimentações anteriores ao período.

**Alternatives considered**: Recalcular o estoque inicial reaplicando todas as movimentações anteriores ao período desde o início dos dados. Rejeitada: custo cresce com 12 meses de histórico (RNF-010) sem ganho de precisão, já que o saldo materializado é confiável. Contar estornos como entrada/saída no balanço: rejeitada, distorceria o balanço de cancelamentos (RGN-007, FR-005).

## Decision 4: Balanço e fechamento de caixa como funcionalidades separadas

**Decision**: O balanço de estoque é um relatório de leitura (GET /api/relatorios/balanco, HU-015) e o fechamento de caixa é uma ação de escrita (GET/POST /api/caixa). A tela de fechamento mostra o resumo do dia e o estado do fechamento; o balanço do dia é consultado na seção de relatórios (SC-003: estoque final confere com o pátio no mesmo instante).

**Rationale**: A RF-043 define o balanço como planilha por período (leitura) e a RGN-006 define o fechamento como ação; separar evita acoplar escrita e leitura e permite que o balanço use o contrato de exportação da HU-015 (RF-044). O Assumption do spec trata conciliação como pré-requisito do fechamento, não do balanço.

**Alternatives considered**: Fechar caixa e gerar balanço em um único endpoint. Rejeitada: misturaria leitura e escrita e impediria o balanço de períodos sem fechamento.

## Decision 5: Colunas exatas de CONVENTIONS §10 e exportação pela HU-015

**Decision**: O balanço usa as colunas exatas de CONVENTIONS §10: Produto, Estoque Inicial, (+) Entradas, (-) Vendas, Estoque Final, Vazios em Pátio (RF-043, CT-001), exportado pelo endpoint `GET /api/relatorios/balanco` da HU-015 com BOM UTF-8 (RF-044, CT-003, RNF-009).

**Rationale**: CONVENTIONS §10 fixa as colunas como contrato de formato; reutilizar o endpoint consolidado da HU-015 evita duplicar a geração de CSV e garante que tela e arquivo nunca divirjam (CT-003).

**Alternatives considered**: Endpoint próprio de exportação do balanço no módulo caixa. Rejeitada: duplicaria o CsvExporter e arriscaria divergência de colunas (CONVENTIONS §10).

## Decision 6: Dia sem movimentações gera balanço com produtos cadastrados e valores zerados

**Decision**: Quando o período não tem movimentações, o balanço lista os produtos cadastrados com Estoque Inicial e Estoque Final iguais ao saldo atual e Entradas/Vendas zeradas, sem erro (Edge Case do spec).

**Rationale**: O Edge Case do spec exige balanço exibido com produtos cadastrados e valores zerados, sem erro; isso permite conferência mesmo em dias parados e mantém o contrato estável.

**Alternatives considered**: Retornar 404 ou lista vazia quando não há movimentações. Rejeitada: contraria o Edge Case e esconderia o estoque parado do administrador.

## Decision 7: Regras de caixa com prioridade máxima de cobertura de teste

**Decision**: O `FechamentoCaixaServiceTest` cobre conciliação completa (fecha), venda pendente (bloqueia com aviso), fechamento duplicado (rejeita) e cálculo de totais por forma de pagamento (RGN-006, CT-002); o `ResumoCaixaServiceTest` cobre a agregação do dia; o `FechamentoCaixaIntegrationTest` roda contra o banco real (RGN-006, CT-001 a CT-004).

**Rationale**: A CONVENTIONS §11 dá prioridade máxima às regras de estoque e caixa; a Constituição §XI.2 manda testar lógica pura sem mocks. O teste de integração prova a conciliação contra dados reais (Constituição §IV).

**Alternatives considered**: Validar o fechamento apenas manualmente pela tela. Rejeitada: a regra de conciliação (RGN-006) é crítica e exige prova contínua.
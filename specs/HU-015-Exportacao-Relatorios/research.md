# Research - Fase 0: Exportação de Relatórios (Período Personalizado)

**HU**: HU-015 - Exportação de Relatórios (Período Personalizado)
**Fase**: 0 - Pesquisa e decisões técnicas
**Data**: 2026-08-20

## Decision 1: Período como parâmetro de consulta com enum e validação de intervalo

**Decision**: O período é informado por `periodo` com valores `HOJE|7DIAS|MES|PERSONALIZADO` (RF-040, CT-001). Para `PERSONALIZADO`, `inicio` e `fim` (formato YYYY-MM-DD) são obrigatórios; o sistema valida que `fim >= inicio`, rejeitando intervalo invertido com mensagem clara (Edge Case do spec).

**Rationale**: A RF-040 define exatamente as quatro opções de período; a CONVENTIONS §6 manda validação com mensagem pt-BR. Validar no servidor impede relatório vazio por engano e mantém a fonte da verdade dos dados (Constituição §IV).

**Alternatives considered**: Interpretar o período no app e enviar apenas inicio/fim. Rejeitada: duplicaria a regra de definição de período (Hoje/7 dias/Mês) e permitiria divergência entre app e API; o enum na API centraliza a regra (CONVENTIONS §12). Aceitar fim anterior a inicio normalizando silenciosamente: rejeitada, esconderia erro do usuário e geraria relatório fora do esperado (Edge Case).

## Decision 2: CSV UTF-8 com BOM para compatibilidade com Excel, LibreOffice e Google Sheets

**Decision**: O CSV é gerado com BOM UTF-8 (`EF BB BF`) no início do arquivo, separador ponto e vírgula e cabeçalhos em pt-BR (CT-004, RNF-009), com escape de aspas e quebras de linha nos campos.

**Rationale**: A RNF-009 exige compatibilidade com Excel, LibreOffice e Google Sheets e preservação de caracteres acentuados (Edge Case do spec). Sem BOM, o Excel Windows lê o CSV como Latin-1 e corrompe acentos ("vazão", "fechamento"); o BOM é o mecanismo padrão para forçar UTF-8 (CONVENTIONS §10).

**Alternatives considered**: Enviar CSV sem BOM em UTF-8 puro. Rejeitada: quebra no Excel Windows (acentos corrompidos), contrariando RNF-009 e o Edge Case do spec. Gerar xlsx binário: rejeitada, o Assumption do spec diz que Excel/CSV é suficiente e CSV é formato aberto exigido (RNF-009).

## Decision 3: Colunas EXATAS por relatório, centralizadas em cabeçalhos canônicos

**Decision**: Cada relatório usa as colunas exatas de CONVENTIONS §10: Vendas (Data/Hora, Produto, Qtd, Valor Unitário, Total (R$), Forma de Pagamento, Tipo (Balcão/Entrega)); Carregamentos (Data, Fornecedor, Produto, Qtd Cheios Entraram, Qtd Vazios Saíram, Custo Total); Balanço (Produto, Estoque Inicial, (+) Entradas, (-) Vendas, Estoque Final, Vazios em Pátio). As definições ficam em `CabeçalhosRelatorio` (RF-041/042/043).

**Rationale**: CONVENTIONS §10 fixa as colunas como contrato de formato; a RGN-008 exige discriminar forma de pagamento e tipo para conferência de caixa. Centralizar evita divergência entre a consulta e o arquivo exportado e entre os três relatórios.

**Alternatives considered**: Definir colunas ad hoc em cada Service. Rejeitada: risco de divergência entre relatórios e de desvio do contrato de CONVENTIONS §10 (CT-004).

## Decision 4: Mesmo endpoint serve JSON (painel) e CSV (exportação)

**Decision**: Cada relatório tem um endpoint único que responde JSON por padrão e CSV quando o cliente envia `Accept: text/csv` ou `?formato=csv`, com `Content-Type: text/csv; charset=UTF-8` e `Content-Disposition: attachment` (RF-044, CT-002).

**Rationale**: A CONVENTIONS §7 exige cliente HTTP tipado no app e o painel precisa exibir os dados antes de exportar (RF-041 a RF-043 são telas consolidadas); um endpoint só evita duplicação de consulta e mantém a consistência entre o que a tela mostra e o que o arquivo traz (CT-002). O download via header dispensa transformação de JSON para CSV no app.

**Alternatives considered**: Endpoints separados para JSON e CSV. Rejeitada: duplicaria as queries de relatório e arriscaria divergência de dados entre tela e arquivo. Gerar CSV no app a partir do JSON: rejeitada, duplicaria a lógica de escape/BOM (Decision 2) e divergiria da CONVENTIONS §10.

## Decision 5: Balanço de estoque derivado das movimentações e do saldo atual

**Decision**: No Balanço (RF-043), por produto: `Estoque Inicial = saldoAtual - (entradas - saídas no período)`, `Estoque Final = saldoAtual`, `(+) Entradas` e `(-) Vendas` somados de `tab_movimentacao_estoque` no período, e `Vazios em Pátio = qtd_vazios` atual. O saldo atual vem de `tab_estoque` (estado materializado).

**Rationale**: O saldo materializado é a fonte da verdade atual (CONVENTIONS §8); as movimentações do período dão a variação. Derivar o inicial pela diferença evita replay do histórico completo e mantém consistência com o painel e o alerta (RF-030/RF-032).

**Alternatives considered**: Recalcular o estoque inicial reaplicando todas as movimentações anteriores ao período desde o início dos dados. Rejeitada: custo cresce com 12 meses de histórico (RNF-010) sem ganho de precisão, já que o saldo materializado é confiável.

## Decision 6: Período sem movimentações gera arquivo válido com cabeçalhos e zero linhas

**Decision**: Quando a consulta do período não retorna linhas, a exportação ainda gera o CSV com o cabeçalho canônico e zero linhas de dados, sem erro (Edge Case do spec, SC-004).

**Rationale**: O Edge Case do spec exige arquivo válido com cabeçalhos e zero linhas, sem falha de exportação; o cabeçalho em pt-BR permanece (CT-004).

**Alternatives considered**: Retornar 404 ou erro quando não há dados. Rejeitada: contraria o Edge Case e quebraria o fluxo de prestação de contas em períodos sem movimento.

## Decision 7: Validação de formato e colunas por teste de integração

**Decision**: O `CsvExporterTest` (lógica pura, sem mock) cobre BOM, escape e separadores; o `RelatorioExportacaoIntegrationTest` gera os três CSVs contra o banco real e asserta as colunas exatas de CONVENTIONS §10, presença do BOM e linhas com acentos preservados (CT-003, CT-004).

**Rationale**: A Constituição §XI.2 manda testar lógica pura sem mocks; a CONVENTIONS §11 prioriza regras de estoque e caixa, e o formato de relatório é contrato de negócio (RNF-009). O teste de integração prova a compatibilidade real do arquivo (Constituição §IV).

**Alternatives considered**: Validar o CSV apenas manualmente abrindo em planilha. Rejeitada: evidência não reprodutível; o teste de integração dá prova contínua dos CT-003/CT-004.
# Research - Fase 0: Relatório de Vendas (Diário/Mensal)

**HU**: HU-016 - Relatório de Vendas (Diário/Mensal)
**Fase**: 0 - Pesquisa e decisões técnicas
**Data**: 2026-08-20

## Decision 1: Período como parâmetro de consulta com enum e validação de intervalo

**Decision**: O período é informado por `periodo` com valores `HOJE|7DIAS|MES|PERSONALIZADO` (RF-040, CT-002). Para `PERSONALIZADO`, `inicio` e `fim` (formato YYYY-MM-DD) são obrigatórios; o sistema valida que `fim >= inicio`, rejeitando intervalo invertido com mensagem clara (Edge Case do spec).

**Rationale**: A RF-040 define exatamente as quatro opções de período; a CONVENTIONS §6 manda validação com mensagem pt-BR. Validar no servidor impede relatório vazio por engano e mantém a fonte da verdade dos dados (Constituição §IV).

**Alternatives considered**: Interpretar o período no app e enviar apenas inicio/fim. Rejeitada: duplicaria a regra de definição de período (Hoje/7 dias/Mês) e permitiria divergência entre app e API; o enum na API centraliza a regra (CONVENTIONS §12). Aceitar fim anterior a inicio normalizando silenciosamente: rejeitada, esconderia erro do usuário e geraria relatório fora do esperado (Edge Case).

## Decision 2: Consulta de leitura direta em tab_venda e tab_venda_item, filtrando status ATIVA

**Decision**: O relatório é uma projeção de leitura sobre `tab_venda` + `tab_venda_item` (join com `tab_produto` para o nome), filtrado por `data_hora` no intervalo e `status = ATIVA` (RF-041, FR-005). Nenhuma tabela auxiliar de relatório é materializada.

**Rationale**: O registro de venda é a fonte da verdade (Constituição §IV.1); filtrar `status = ATIVA` na query exclui vendas canceladas da consolidação (FR-005, RGN-007) sem custo de sincronização. O histórico de 12 meses é mantido em `tab_venda` (RNF-010).

**Alternatives considered**: Materializar tabela de relatório diária. Rejeitada: duplicaria dados, exigiria job de sincronização e arriscaria divergência com o registro da venda (RGN-008). Consultar também vendas canceladas e filtrar no app: rejeitada, vazaria dados que não devem aparecer e inflaria o payload (FR-005).

## Decision 3: Colunas exatas de CONVENTIONS §10 com valores já consolidados na venda

**Decision**: As colunas são exatamente as de CONVENTIONS §10: Data/Hora, Produto, Qtd, Valor Unitário, Total (R$), Forma de Pagamento, Tipo (Balcão/Entrega) (RF-041, CT-001). O Total (R$) da linha é `quantidade * valor_unitario`; o `valor_unitario` e o `total` da venda já embutem o acréscimo de cartão (RF-021-A, somente carga Gás) e a taxa de entrega (RF-022) aplicados no lançamento.

**Rationale**: CONVENTIONS §10 fixa as colunas como contrato de formato; a RGN-008 exige discriminar forma de pagamento e tipo em 100% das linhas para conferência do caixa físico. Usar os valores gravados na venda evita recalcular regra de negócio na leitura (CONVENTIONS §12).

**Alternatives considered**: Recalcular acréscimo e taxa na consulta do relatório. Rejeitada: divergiria do total gravado na venda e duplicaria a regra de negócio (RF-022; RF-021-A somente carga Gás). Incluir colunas extras (taxa, acréscimo, desconto): rejeitada, foge do contrato de CONVENTIONS §10 e da RF-041.

## Decision 4: Exportação CSV reutiliza o endpoint consolidado da HU-015

**Decision**: O botão de exportação do relatório de vendas baixa o CSV do endpoint `GET /api/relatorios/vendas` (Accept: text/csv ou ?formato=csv), definido e documentado na HU-015, com BOM UTF-8, separador ponto e vírgula e as mesmas colunas de CONVENTIONS §10 (RF-044, CT-003).

**Rationale**: A HU-015 já estabelece o contrato canônico de exportação CSV (CsvExporter com BOM e escape, RNF-009); reutilizar evita duplicar a geração de arquivo e garante que tela e arquivo nunca divirjam (CT-003). O Assumption do spec aceita Excel/CSV como formato.

**Alternatives considered**: Endpoint próprio de CSV no módulo venda. Rejeitada: duplicaria o CsvExporter e arriscaria divergência de colunas entre os relatórios (CONVENTIONS §10). Gerar CSV no app a partir do JSON: rejeitada, duplicaria a lógica de escape/BOM e contrariaria a CONVENTIONS §7.

## Decision 5: Período sem vendas retorna lista vazia sem erro

**Decision**: Quando a consulta do período não retorna linhas, o relatório responde 200 com `linhas` vazias e `totalPeriodo = 0`, sem erro (Edge Case do spec, SC-001).

**Rationale**: O Edge Case do spec exige relatório exibido vazio, sem erro; manter o contrato estável permite que a tela reaja normalmente e que o CSV da HU-015 também saia com cabeçalho e zero linhas.

**Alternatives considered**: Retornar 404 ou erro quando não há dados. Rejeitada: contraria o Edge Case e quebraria o fluxo de conferência em dias sem venda.

## Decision 6: Total do período e totais por forma de pagamento agregados no servidor

**Decision**: A resposta do relatório inclui `totalPeriodo` (soma das linhas) e `totaisPorForma` (dinheiro, pix, cartao) calculados no servidor a partir das vendas ATIVA do período (RGN-008, FR-003).

**Rationale**: A RGN-008 exige conferência do caixa físico por forma de pagamento; agregar no servidor mantém a fonte da verdade no banco (Constituição §IV) e evita somar no app, onde valores poderiam divergir do relatório exportado (CT-003).

**Alternatives considered**: Somar os totais no app a partir das linhas. Rejeitada: duplicaria a lógica de agregação e permitiria divergência entre tela e arquivo exportado (RGN-008).

## Decision 7: Testes de filtro, exclusão de canceladas e colunas exatas

**Decision**: O `RelatorioVendaServiceTest` cobre filtro por período (Hoje, 7 dias, Mês, personalizado) e exclusão de vendas CANCELADA (lógica pura, sem mock); o `VendaControllerTest` cobre validação de período e intervalo invertido; o `RelatorioVendaIntegrationTest` roda contra o banco real e asserta as colunas exatas de CONVENTIONS §10 (CT-001 a CT-003).

**Rationale**: A Constituição §XI.2 manda testar lógica pura sem mocks; a CONVENTIONS §11 prioriza regras de estoque e caixa, e o formato do relatório é contrato de negócio (RGN-008, RNF-009). O teste de integração prova o resultado real sobre o banco (Constituição §IV).

**Alternatives considered**: Validar o relatório apenas manualmente pela tela. Rejeitada: evidência não reprodutível; o teste de integração dá prova contínua dos CTs.
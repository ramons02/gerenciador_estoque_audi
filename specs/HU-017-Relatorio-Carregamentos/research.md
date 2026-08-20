# Research - Fase 0: Relatório de Carregamentos (Entradas)

**HU**: HU-017 - Relatório de Carregamentos (Entradas)
**Fase**: 0 - Pesquisa e decisões técnicas
**Data**: 2026-08-20

## Decision 1: Período como parâmetro de consulta com enum e validação de intervalo

**Decision**: O período é informado por `periodo` com valores `HOJE|7DIAS|MES|PERSONALIZADO` (RF-040, CT-002). Para `PERSONALIZADO`, `inicio` e `fim` (formato YYYY-MM-DD) são obrigatórios; o sistema valida que `fim >= inicio`, rejeitando intervalo invertido com mensagem clara (Edge Case do spec).

**Rationale**: A RF-040 define exatamente as quatro opções de período; a CONVENTIONS §6 manda validação com mensagem pt-BR. Validar no servidor impede relatório vazio por engano e mantém a fonte da verdade dos dados (Constituição §IV).

**Alternatives considered**: Interpretar o período no app e enviar apenas inicio/fim. Rejeitada: duplicaria a regra de definição de período e permitiria divergência entre app e API; o enum na API centraliza a regra (CONVENTIONS §12).

## Decision 2: Consulta de leitura em tab_carregamento com join de fornecedor e produto

**Decision**: O relatório é uma projeção de leitura sobre `tab_carregamento` (data_hora, id_fornecedor, custo_total) + `tab_carregamento_item` (id_produto, qtd_cheios_entraram, qtd_vazios_sairam) + `tab_fornecedor` (nome) (RF-042, CT-001). Cada carregamento gera uma linha por produto, pois a RF-042 traz Produto como coluna.

**Rationale**: O registro do carregamento é a fonte da verdade (Constituição §IV.1); o RDN-009 define o carregamento como dois fluxos opostos (cheios entram, vazios saem), e as duas quantidades vivem no item. O nome do fornecedor vem de `tab_fornecedor` (RF-005).

**Alternatives considered**: Materializar tabela de relatório. Rejeitada: duplicaria dados e arriscaria divergência com o carregamento gravado. Uma linha por carregamento com produtos concatenados: rejeitada, foge do formato linha/coluna da RF-042 e do CSV de CONVENTIONS §10.

## Decision 3: Apenas carregamentos registrados e confirmados

**Decision**: O relatório consolida somente carregamentos confirmados (FR-004); o sistema não persiste carregamento em edição - o registro só existe em `tab_carregamento` após a confirmação da chegada do caminhão (HU-006), então a consulta não precisa de filtro de status.

**Rationale**: A FR-004 exige apenas carregamentos confirmados; como a feature de entrada (HU-006) grava o carregamento em transação única apenas na confirmação, todo registro em `tab_carregamento` é confirmado por construção (CONVENTIONS §5). Documentar a invariante evita inventar coluna de status desnecessária.

**Alternatives considered**: Adicionar coluna `status` em `tab_carregamento` para rascunho. Rejeitada: o processo de chegada de caminhão (HU-006) é atômico e não tem rascunho persistido; coluna morta violaria o princípio de não criar regra fora dos requisitos (CONVENTIONS §12).

## Decision 4: Colunas exatas de CONVENTIONS §10

**Decision**: As colunas são exatamente as de CONVENTIONS §10: Data, Fornecedor, Produto, Qtd Cheios Entraram, Qtd Vazios Saíram, Custo Total (RF-042, CT-001). Custo Total vem de `tab_carregamento.custo_total`; as quantidades vêm do item.

**Rationale**: CONVENTIONS §10 fixa as colunas como contrato de formato; o custo total por linha é o custo do carregamento (RF-010). Reutilizar os valores gravados evita recalcular custo médio na leitura (RF-012 é regra de escrita).

**Alternatives considered**: Recalcular custo total por produto na consulta. Rejeitada: duplicaria a apuração de custo e divergiria do valor gravado na chegada (RF-010/RF-012).

## Decision 5: Exportação CSV reutiliza o endpoint consolidado da HU-015

**Decision**: O botão de exportação do relatório de carregamentos baixa o CSV do endpoint `GET /api/relatorios/carregamentos` (Accept: text/csv ou ?formato=csv), definido e documentado na HU-015, com BOM UTF-8, separador ponto e vírgula e as mesmas colunas de CONVENTIONS §10 (RF-044, CT-003).

**Rationale**: A HU-015 já estabelece o contrato canônico de exportação CSV (RNF-009); reutilizar evita duplicar a geração de arquivo e garante que tela e arquivo nunca divirjam (CT-003).

**Alternatives considered**: Endpoint próprio de CSV no módulo carregamento. Rejeitada: duplicaria o CsvExporter e arriscaria divergência de colunas entre os relatórios (CONVENTIONS §10).

## Decision 6: Período sem carregamentos retorna lista vazia sem erro

**Decision**: Quando a consulta do período não retorna linhas, o relatório responde 200 com `linhas` vazias, sem erro (Edge Case do spec, SC-001).

**Rationale**: O Edge Case do spec exige relatório exibido vazio, sem erro; manter o contrato estável permite que a tela reaja normalmente e que o CSV da HU-015 saia com cabeçalho e zero linhas.

**Alternatives considered**: Retornar 404 ou erro quando não há dados. Rejeitada: contraria o Edge Case e quebraria o fluxo de acompanhamento em períodos sem carregamento.

## Decision 7: Testes de filtro, quantidades e colunas exatas

**Decision**: O `RelatorioCarregamentoServiceTest` cobre filtro por período e o par cheios/vazios por item (lógica pura, sem mock); o `CarregamentoControllerTest` cobre validação de período e intervalo invertido; o `RelatorioCarregamentoIntegrationTest` roda contra o banco real e asserta as colunas exatas de CONVENTIONS §10 (CT-001 a CT-003).

**Rationale**: A Constituição §XI.2 manda testar lógica pura sem mocks; a CONVENTIONS §11 prioriza regras de estoque e caixa, e o formato do relatório é contrato de negócio (RNF-009). O teste de integração prova o resultado real sobre o banco (Constituição §IV).

**Alternatives considered**: Validar o relatório apenas manualmente pela tela. Rejeitada: evidência não reprodutível; o teste de integração dá prova contínua dos CTs.
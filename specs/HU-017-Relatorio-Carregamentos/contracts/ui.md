# Contract UI: Relatório de Carregamentos (Entradas)

**HU**: HU-017 - Relatório de Carregamentos (Entradas)
**Tela**: Painel de Relatórios - seção Relatório de Carregamentos | **Rota**: /relatorios

## Descrição

Seção do painel de relatórios (RF-040 a RF-044) onde o administrador gera o relatório de carregamentos por período e exporta em CSV (CT-001, CT-002, CT-003). O seletor de período é compartilhado com as demais seções do painel (mesma convenção da HU-015).

## Layout

- Seletor de período no topo da seção: Hoje, Últimos 7 dias, Mês Atual, Personalizado (RF-040, CT-002).
- Quando "Personalizado": campos Data inicial e Data final (YYYY-MM-DD).
- Tabela com as colunas EXATAS de CONVENTIONS §10: Data, Fornecedor, Produto, Qtd Cheios Entraram, Qtd Vazios Saíram, Custo Total (RF-042, CT-001).
- Botão "Exportar CSV" (RF-044, CT-003).

## Campos e ações

| Elemento | Comportamento |
|---|---|
| Seletor de período | 4 opções; ao trocar, reconsulta o relatório via GET /api/carregamentos/relatorio |
| Data inicial/final | Visíveis apenas em Personalizado; validação "final >= inicial" |
| Tabela do relatório | Colunas exatas de CONVENTIONS §10 (mesmas do CSV) |
| Botão "Exportar CSV" | Baixa o arquivo via GET /api/relatorios/carregamentos com Accept: text/csv (HU-015) |
| Indicador de carregamento | Skeleton enquanto consulta; bloqueio de exportação duplicada |

## Validações inline

- Personalizado sem datas: "Informe as datas inicial e final para o período personalizado."
- Data final anterior à inicial: "A data final deve ser posterior ou igual à data inicial." (Edge Case do spec)
- Ação de exportar desabilitada enquanto a consulta está em andamento.

## Regras de comportamento

- Apenas carregamentos confirmados aparecem no relatório (FR-004); não existe carregamento em edição persistido (HU-006).
- Período sem carregamentos: tabela vazia, sem erro (Edge Case do spec, SC-001).
- O CSV baixa com BOM UTF-8 e abre em Excel, LibreOffice e Google Sheets com acentos preservados (RNF-009, CT-003).
- O download usa o cliente HTTP tipado (carregamentosApi.ts/relatoriosApi.ts) e o utilitário downloadArquivo.ts; nunca fetch solto (CONVENTIONS §7).

## Mensagens

- Sucesso de download: notificação discreta "Arquivo exportado."
- Erro de rede: "Não foi possível carregar o relatório. Verifique a conexão e tente novamente."
- Erros de validação/permissão: mensagens do backend exibidas em alerta (pt-BR).
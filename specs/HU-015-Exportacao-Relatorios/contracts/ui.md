# Contract UI: Exportação de Relatórios (Período Personalizado)

**HU**: HU-015 - Exportação de Relatórios (Período Personalizado)
**Tela**: Painel de Relatórios | **Rota**: /relatorios

## Descrição

Painel com os três relatórios consolidados (RF-041 a RF-043) e exportação em CSV por período (RF-040, RF-044). O administrador seleciona o período uma vez e exporta qualquer um dos relatórios (CT-001, CT-002).

## Layout

- Seletor de período no topo: Hoje, Últimos 7 dias, Mês Atual, Personalizado (RF-040, CT-001).
- Quando "Personalizado": campos Data inicial e Data final (YYYY-MM-DD).
- Três seções: Relatório de Vendas, Relatório de Carregamentos, Fechamento de Caixa e Balanço.
- Cada seção: tabela com os dados do período + botão "Exportar CSV" (RF-044, CT-002).

## Campos e ações

| Elemento | Comportamento |
|---|---|
| Seletor de período | 4 opções; ao trocar, reconsulta os três relatórios |
| Data inicial/final | Visíveis apenas em Personalizado; validação "final >= inicial" |
| Tabela do relatório | Colunas exatas de CONVENTIONS §10 (mesmas do CSV) |
| Botão "Exportar CSV" | Baixa o arquivo via GET com Accept: text/csv ou ?formato=csv |
| Indicador de carregamento | Skeleton enquanto consulta; bloqueio de exportação duplicada |

## Validações inline

- Personalizado sem datas: "Informe as datas inicial e final para o período personalizado."
- Data final anterior à inicial: "A data final deve ser posterior ou igual à data inicial." (Edge Case do spec)

## Regras de comportamento

- Cada relatório tem seu próprio botão de exportação funcional (CT-002); exportar um não altera o período selecionado dos outros (Edge Case do spec).
- Período sem movimentações: a tabela fica vazia e o CSV é baixado com cabeçalho e zero linhas, sem erro (Edge Case do spec, SC-004).
- O CSV baixa com BOM UTF-8 e abre em Excel, LibreOffice e Google Sheets com acentos preservados (RNF-009, CT-003).
- O download usa o cliente HTTP tipado (relatoriosApi.ts) e o utilitário downloadArquivo.ts; nunca fetch solto (CONVENTIONS §7).

## Mensagens

- Sucesso de download: notificação discreta "Arquivo exportado."
- Erro de rede: "Não foi possível exportar o relatório. Verifique a conexão e tente novamente."
- Erros de validação/permissão: mensagens do backend exibidas em alerta (pt-BR).
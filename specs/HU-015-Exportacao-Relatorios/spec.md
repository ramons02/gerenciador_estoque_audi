# Feature Specification: Exportação de Relatórios (Período Personalizado)

**Feature Branch**: `HU-015-Exportacao-Relatorios`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como administrador, quero exportar relatórios em Excel/CSV por período (Hoje, Últimos 7 dias, Mês Atual ou personalizado), para análise e prestação de contas."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Seleção de período do relatório (Priority: P1)
O painel oferece ao administrador a seleção do período dos relatórios: Hoje, Últimos 7 dias, Mês Atual ou um período personalizado definido por ele, de modo que a exportação cubra exatamente o intervalo desejado.

**Why this priority**: A seleção de período (RF-040) é a base de toda a exportação — sem ela não há como gerar relatórios para análise e prestação de contas nos intervalos que o negócio exige.

**Independent Test**: Abrir o painel de relatórios, selecionar cada uma das opções de período (incluindo um período personalizado) e confirmar que todas estão disponíveis e selecionáveis.

**Acceptance Scenarios**:

1. **Given** o painel de relatórios aberto, **When** o administrador acessa a seleção de período, **Then** ele pode escolher entre Hoje, Últimos 7 dias, Mês Atual e período personalizado (CT-001, RF-040).
2. **Given** a opção de período personalizado selecionada, **When** o administrador informa as datas inicial e final, **Then** o sistema aceita o intervalo e o usa para o relatório (CT-001).

---

### User Story 2 - Exportação de cada relatório (Priority: P1)
Cada relatório do painel possui um botão de exportação que gera o arquivo em Excel/CSV com os dados consolidados do período selecionado, permitindo ao administrador baixar o relatório de vendas, de carregamentos ou de fechamento de caixa e balanço.

**Why this priority**: O botão de exportação (RF-044) é a entrega concreta da HU — sem ele a seleção de período não gera o arquivo que o administrador precisa para análise e prestação de contas.

**Independent Test**: Selecionar um período, clicar no botão de exportação de um relatório e confirmar que o arquivo é gerado e baixado com os dados do período escolhido.

**Acceptance Scenarios**:

1. **Given** um período selecionado, **When** o administrador clica no botão de exportação de um relatório, **Then** o sistema gera o arquivo (Excel/CSV) com os dados consolidados daquele período (CT-002, RF-044).
2. **Given** relatórios de vendas, carregamentos e fechamento de caixa disponíveis, **When** o administrador exporta cada um, **Then** cada relatório possui seu próprio botão de exportação funcional (CT-002).

---

### User Story 3 - Compatibilidade e legibilidade dos arquivos (Priority: P2)
Os arquivos exportados abrem corretamente em Excel, LibreOffice e Google Sheets, com cabeçalhos claros em português, para que a prestação de contas seja feita em qualquer ferramenta de planilha sem ajustes manuais.

**Why this priority**: A compatibilidade (RNF-009) e os cabeçalhos legíveis garantem que o arquivo seja efetivamente utilizável na análise — se não abrir ou não for legível, a exportação perde o valor.

**Independent Test**: Exportar um relatório e abrir o arquivo em Excel, LibreOffice e Google Sheets verificando a renderização e a presença dos cabeçalhos em português.

**Acceptance Scenarios**:

1. **Given** um relatório exportado, **When** o administrador abre o arquivo em Excel, LibreOffice ou Google Sheets, **Then** o arquivo abre corretamente (CT-003, RNF-009).
2. **Given** um relatório exportado, **When** o administrador visualiza o arquivo, **Then** ele contém cabeçalhos claros em português para cada coluna (CT-004).

### Edge Cases

- Período personalizado com data final anterior à data inicial deve ser rejeitado ou normalizado, sem gerar relatório vazio por engano.
- Período sem movimentações deve gerar arquivo válido, com cabeçalhos e zero linhas de dados (sem erro de exportação).
- Exportação de um relatório não deve interferir no período selecionado de outro relatório no mesmo painel.
- Caracteres acentuados do português nos dados e cabeçalhos devem ser preservados (UTF-8) ao abrir em qualquer planilha (RNF-009).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema deve oferecer a seleção de período Hoje, Últimos 7 dias, Mês Atual e período personalizado para os relatórios (RF-040, CT-001).
- **FR-002**: O sistema deve disponibilizar um botão de exportação para cada relatório do painel, gerando o arquivo com os dados do período selecionado (RF-044, CT-002).
- **FR-003**: Os arquivos exportados devem ser compatíveis com Excel, LibreOffice e Google Sheets (UTF-8) (RNF-009, CT-003).
- **FR-004**: Os arquivos exportados devem conter cabeçalhos claros em português (RNF-009, CT-004).

### Key Entities *(include if feature involves data)*

- **Relatório**: consolidação das movimentações (vendas, carregamentos, fechamento de caixa e balanço de estoque) em planilha.
- **Período**: intervalo de tempo aplicado ao relatório — Hoje, Últimos 7 dias, Mês Atual ou personalizado.
- **Arquivo de Exportação**: planilha Excel/CSV gerada com dados do período e cabeçalhos em português.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% dos relatórios do painel são exportáveis com as quatro opções de período (Hoje, 7 dias, Mês Atual, personalizado).
- **SC-002**: 100% dos arquivos exportados abrem sem erros em Excel, LibreOffice e Google Sheets.
- **SC-003**: 100% dos arquivos exportados possuem cabeçalhos em português legíveis, inclusive com caracteres acentuados preservados.
- **SC-004**: Exportações de períodos sem movimentações geram arquivos válidos, sem falha.

## Assumptions

- Os relatórios (vendas, carregamentos, fechamento de caixa/balanço) já existem como telas consolidadas no painel (RF-041 a RF-043) — esta feature adiciona o período e a exportação (RF-040/RF-044).
- A exportação em Excel/CSV é suficiente para o usuário; formatos proprietários adicionais não são exigidos.
- O histórico de movimentações fica armazenado por pelo menos 12 meses (RNF-010), cobrindo qualquer período personalizado solicitado.
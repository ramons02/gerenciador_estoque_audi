# Research: HU-008 - Venda com Entrega (Taxa de Entrega)

**HU**: HU-008 | **Feature**: Venda com Entrega | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Requisitos vinculados**: RF-020, RF-022, RF-041, RF-052, RNF-001, RNF-005, RNF-007, RGN-001

Fase 0 - decisões de design registradas antes de qualquer código. Cada decisão referencia a convenção, constituição ou requisito que a justifica.

---

## Decisão 1: Taxa de entrega lida da configuração e aplicada no Service

**Decision**: Ao persistir uma venda com `tipo = ENTREGA`, o `VendaService` lê `taxa_entrega` de `tab_configuracao` na mesma transação, soma ao total e grava o valor aplicado em `tab_venda.taxa_entrega`. Vendas BALCAO não aplicam a taxa.

**Rationale**: RF-022 e RGN-001 definem que a venda por Entrega acrescenta automaticamente a taxa configurável ao total, sem cálculo manual do vendedor. Aplicar no Service garante que o valor gravado seja o cobrado (caixa confere - RGN-008) e que a taxa nunca fique hardcoded (CONVENTIONS §8).

**Alternatives considered**: Taxa enviada pela tela no request - rejeitada por permitir divergência de caixa e burlar a configuração; taxa fixa no código - rejeitada por violar RF-052/CONVENTIONS §8 (configurável pelo admin).

---

## Decisão 2: Configuração da taxa persistida para novas vendas (não retroativa)

**Decision**: A tela de Configurações altera a taxa via `PUT /api/configuracoes` (chave `taxa_entrega`); o novo valor passa a valer apenas para vendas confirmadas depois da alteração. Vendas já lançadas mantêm o valor gravado no cabeçalho.

**Rationale**: CT-002 exige que o admin configure o valor e que novas vendas o usem; o spec.md (Edge Cases) define que taxa alterada após vendas do dia não muda as vendas já lançadas. Persistir a taxa aplicada em `tab_venda.taxa_entrega` preserva a conferência histórica (RGN-008, RF-041).

**Alternatives considered**: Recalcular o total de vendas antigas ao mudar a taxa - rejeitado por violar o edge case do spec e corromper o histórico; derivar a taxa do total a cada leitura - rejeitado por tornar o valor cobrado ambíguo.

---

## Decisão 3: Tipo de venda persistido e discriminado no relatório

**Decision**: `tab_venda.tipo` guarda BALCAO ou ENTREGA (RF-020); o Relatório de Vendas (RF-041, CONVENTIONS §10) discrimina o tipo e exibe o total com a taxa aplicada quando Entrega, com cabeçalhos pt-BR e exportação CSV UTF-8.

**Rationale**: CT-003 e RGN-008 exigem discriminar tipo (Balcão/Entrega) e total com taxa para conferência do caixa físico; RF-041 fixa as colunas do relatório e RNF-009 o formato UTF-8 com cabeçalhos claros.

**Alternatives considered**: Relatório sem a coluna de tipo - rejeitado por violar CT-003 e RGN-008; tipo derivado de outra fonte (ex.: endereço) - rejeitado por duplicar informação e quebrar a rastreabilidade da venda.

---

## Decisão 4: Tratamento de taxa zero e de Entrega sem taxa configurada

**Decision**: Taxa zero é válida (entrega gratuita) e segue a regra sem acréscimo. Venda do tipo Entrega só pode ser concluída com a chave `taxa_entrega` presente em `tab_configuracao`; ausência bloqueia com mensagem clara em pt-BR.

**Rationale**: Os Edge Cases do spec.md exigem os dois comportamentos: taxa zero não quebra a venda e Entrega sem taxa configurada não conclui (exigir configuração antes). A ausência da chave é um estado de configuração, não um erro de sistema (Constituição §XI.3).

**Alternatives considered**: Assumir taxa zero quando a chave não existe - rejeitado porque venda de Entrega sem taxa configurada deve ser bloqueada (edge case explícito); mensagem técnica de erro - rejeitado por violar CONVENTIONS §6 e §XI.3.

---

## Decisão 5: Total composto persistido na mesma transação da venda

**Decision**: O total final (itens + taxa, e acréscimo do cartão quando aplicável - somente carga Gás com Cartão, HU-007) é composto no `VendaService` e persistido em `tab_venda.total` na mesma transação que baixa o estoque e grava a movimentação.

**Rationale**: O total da Entrega é parte do caixa e do estoque da mesma operação; estado parcial (estoque baixado sem total correto) é defeito grave (Constituição §III.3, CONVENTIONS §5.1, RNF-005). A composição no Service mantém a regra de caixa testável como lógica pura (Constituição §XI.2).

**Alternatives considered**: Somar a taxa depois do commit da venda - rejeitado por criar estado parcial de caixa; composição no relatório apenas - rejeitado por divergir do valor gravado.

---

## Consistência com as decisões globais

As decisões seguem as DECISÕES TÉCNICAS do projeto: tudo dentro da transação única estoque + caixa da venda, cálculo de total no Service antes de persistir, configurações em `tab_configuracao` nunca hardcoded, sem DELETE de venda, erros estruturados em pt-BR e relatório com cabeçalhos pt-BR em CSV UTF-8 (CONVENTIONS §10). A taxa de entrega é um comportamento complementar do lançamento da HU-007 no mesmo `POST /api/vendas`; esta HU cobre apenas os CT-001 a CT-003 da entrega.
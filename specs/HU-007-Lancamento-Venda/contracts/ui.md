# UI Contract: HU-007 - Lançamento Rápido de Venda (Balcão/Entrega)

**HU**: HU-007 | **Feature**: Lançamento Rápido de Venda | **Date**: 2026-08-20 | **Spec**: [spec.md](../spec.md)
**Requisitos vinculados**: RF-020, RF-021, RF-021-A, RF-031, RF-052, RNF-001, RNF-002, RNF-003, RGN-002
**API**: [api.md](api.md)

Tela em `src/features/vendas/VendasPage.tsx` (módulo `vendas`), consumindo `src/api/vendas.ts`, `src/api/configuracoes.ts`, `src/api/produtos.ts` e `src/api/estoque.ts` (cliente HTTP tipado, nunca fetch solto - CONVENTIONS §7). Esta tela é a base das HU-008 (entrega), HU-009 (troca) e HU-010 (vasilhame novo).

---

## Tela: Vendas (lançamento rápido)

Região principal de lançamento (formulário único, sem navegação para lançar - RNF-001) e histórico do dia ao lado/abaixo (não bloqueia o lançamento - RNF-003).

### Formulário de lançamento

| Campo | Tipo | Obrigatório | Validação inline | Comportamento |
|---|---|---|---|---|
| Produto | Select grande | sim | Não pode ficar vazio | Lista de `GET /api/produtos`; mostra preço de venda ao selecionar |
| Quantidade | Number grande | sim | > 0 e <= saldo de cheios | Saldo exibido ao lado: "Disponível: 10" (RF-031) |
| Tipo | Segmented (Balcão / Entrega) | sim | - | Valor default Balcão; Entrega exige taxa configurada (HU-008) |
| Forma de pagamento | Botões/seleção | sim | - | Apenas formas habilitadas em `GET /api/configuracoes` (CT-003) |
| Total | Money grande (somente leitura) | - | - | Atualizado em tempo real: qtd × preço, + acréscimo do cartão por unidade quando Cartão e produto de carga Gás (CT-002, CT-003-A); Água com preço normal |

**Ações**:
- Botão "Confirmar venda": habilita só com produto, quantidade, tipo e forma válidos (CT-001); executa `POST /api/vendas` e fecha em poucos segundos (CT-005).
- Após sucesso: toast "Venda registrada com sucesso.", histórico do dia atualizado, formulário limpo e saldo do produto atualizado (CT-006: data/hora e usuário visíveis no histórico).

**Mensagens**:
- Bloqueio de estoque: mensagem da API em destaque "Estoque insuficiente para 3 unidade(s) de Gás P13. Disponível: 1." e botão desabilitado (CT-004; RF-031).
- Forma desabilitada após abrir a tela: a lista é recarregada e a forma some; se a venda estava em edição, o campo é limpo com aviso "A forma de pagamento CARTAO não está habilitada nas Configurações." (CT-003; spec.md Edge Cases).
- Validações inline: "Informe o produto.", "Informe a quantidade.", "Informe o tipo da venda.", "Informe a forma de pagamento."
- Erro de rede: "Não foi possível lançar a venda. Verifique a conexão." - sem perder o formulário.

### Histórico do dia

Colunas: Data/Hora, Produto, Qtd, Valor unitário, Total, Forma de pagamento, Tipo, Usuário (base do Relatório de Vendas - RF-041, CONVENTIONS §10). Exibido sem bloquear o lançamento (RNF-003). Sem ação de exclusão em tela (RNF-007).

---

## Fluxo por CT

| CT | Fluxo na UI |
|---|---|
| CT-001 | Confirmar com formulário vazio: validações inline exigem produto, quantidade, tipo e forma |
| CT-002 | Produto com preço 115,00 e quantidade 2: total exibe R$ 230,00 em tempo real |
| CT-003 | Configurações com PIX desabilitado: PIX não aparece entre as formas; demais formas aparecem |
| CT-003-A | Acréscimo 5,00 por unidade com Cartão: total exibe qtd × (preço + 5,00) somente para produto de carga Gás; Dinheiro/PIX exibem preço normal em todos os produtos; Água exibe preço normal mesmo paga com Cartão |
| CT-004 | Saldo 10, tentar quantidade 11: botão bloqueado e mensagem de estoque insuficiente; com 10 exatos, liberado |
| CT-005 | Fluxo completo (selecionar produto, quantidade, tipo, forma, confirmar) executável em poucos segundos |
| CT-006 | Histórico mostra data/hora e usuário responsável de cada venda lançada |

---

## Nota de integração com as variações

- HU-008: o campo Tipo com "Entrega" soma a taxa ao total (mesmo formulário).
- HU-009: opção "Troca de vasilhame" marca a venda como troca (sem custo; pátio atualizado).
- HU-010: opção "Vasilhame novo" exige cliente e soma preço do casco.
- Esses campos são ativados nas respectivas entregas, sem duplicar a tela.
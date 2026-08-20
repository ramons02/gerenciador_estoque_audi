# UI Contract: HU-006 - Registro de Chegada de Caminhão (Entrada)

**HU**: HU-006 | **Feature**: Chegada de Caminhão (Carregamento) | **Date**: 2026-08-20 | **Spec**: [spec.md](../spec.md)
**Requisitos vinculados**: RF-010, RF-011, RF-012, RDN-003, RNF-001, RNF-005
**API**: [api.md](api.md)

Tela única em `src/features/carregamentos/CarregamentosPage.tsx` (módulo `carregamentos`), consumindo `src/api/carregamentos.ts`, `src/api/produtos.ts` e `src/api/fornecedores.ts` (cliente HTTP tipado, nunca fetch solto - CONVENTIONS §7).

---

## Tela: Carregamentos

Duas regiões: formulário de nova chegada (esquerda/topo) e lista de carregamentos confirmados (abaixo), com botão de exportação reservado ao módulo de relatórios (RF-042, HU-017).

### Formulário de chegada

| Campo | Tipo | Obrigatório | Validação inline | Comportamento |
|---|---|---|---|---|
| Fornecedor | Select | sim | Não pode ficar vazio | Lista de `GET /api/fornecedores` ativos |
| Data | Date/time | não | - | Default = agora (Brasília) |
| Produto | Select | sim | Não pode ficar vazio | Lista de `GET /api/produtos` ativos |
| Qtd cheios entraram | Number | sim | > 0 | Campo de número grande (RNF-002) |
| Qtd vazios saíram | Number | sim | >= 0 e <= saldo de vazios do pátio | Saldo atual exibido ao lado como dica |
| Custo total | Money | sim | >= 0 | Formato R$ |
| Valor unitário | Money (somente leitura) | - | - | Calculado em tempo real: custo total ÷ cheios, 2 casas (CT-001) |

**Ações**:
- Botão "Registrar chegada": habilita só com todos os obrigatórios válidos; confirma via `POST /api/carregamentos`.
- Após sucesso: formulário limpo, lista atualizada, saldos de `GET /api/estoque` refletem cheios +N e vazios -N (CT-003).

**Mensagens**:
- Sucesso: "Carregamento registrado com sucesso." (toast)
- Erro 409: mostra a mensagem da API em destaque: "Devolução de 15 vazios excede o saldo do pátio. Disponível: 10." e mantém o formulário preenchido (CT-002).
- Erro de rede: "Não foi possível registrar o carregamento. Verifique a conexão." - sem perda do formulário.
- Validações inline em pt-BR: "Informe o fornecedor.", "A quantidade de cheios deve ser maior que zero.", "A devolução de vazios não pode exceder o saldo do pátio (10)."

### Lista de carregamentos

Colunas: Data/Hora, Fornecedor, Produto, Cheios entraram, Vazios saíram, Custo total, Valor unitário (mesmos cabeçalhos do Relatório de Carregamentos - RF-042, CONVENTIONS §10). Sem ação de exclusão em tela (RNF-007).

---

## Fluxo por CT

| CT | Fluxo na UI |
|---|---|
| CT-001 | Preencher fornecedor, produto, 100 cheios, 20 vazios, R$ 3.000,00: campo "Valor unitário" exibe R$ 30,00 antes de confirmar |
| CT-002 | Saldo do pátio 10, informar 15 vazios: botão bloqueado e mensagem inline "A devolução de vazios não pode exceder o saldo do pátio (10)."; com 10 exatos, liberado |
| CT-003 | Confirmar: lista atualiza e saldos do painel de estoque mostram cheios +N e vazios -N imediatamente |
| CT-004 | Sem indicador na tela (custo médio é interno ao produto; conferível em relatório de margem) |
| CT-005 | O registro aparece no Relatório de Carregamentos do período (HU-017) |
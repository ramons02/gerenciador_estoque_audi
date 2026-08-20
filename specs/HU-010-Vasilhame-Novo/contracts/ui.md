# UI Contract: HU-010 - Venda de Vasilhame Novo (Casco + Carga)

**HU**: HU-010 | **Feature**: Venda de Vasilhame Novo | **Date**: 2026-08-20 | **Spec**: [spec.md](../spec.md)
**Requisitos vinculados**: RF-004, RF-024, RF-026, RF-028, RNF-001, RNF-002, RGN-010
**API**: [api.md](api.md)
**Base comum**: [ui.md HU-007](../HU-007-Lancamento-Venda/contracts/ui.md)

A HU-010 usa o mesmo formulário de lançamento da HU-007, com a opção de vasilhame novo e a exigência de cliente.

---

## Tela: Vendas (opção "Vasilhame novo")

No `LancamentoVenda.tsx` (HU-007), por item do lançamento:

| Elemento | Comportamento |
|---|---|
| Opção "Vasilhame novo" (checkbox/switch por item) | Marca o item como CASCO_NOVO: sem devolução de vazio (CT-001) |
| Preço exibido | Passa a ser casco + carga, atualizado em tempo real (ex.: "Gás P13: R$ 230,00 (carga R$ 115,00 + casco R$ 115,00)") (CT-002) |
| Seletor de cliente | Torna-se obrigatório quando há item de vasilhame novo; busca por nome/telefone em `GET /api/clientes` (CT-004) |
| Botão "Cadastrar cliente" | Abre cadastro rápido (modal) reusando HU-004, sem sair do lançamento (RNF-001; spec.md Edge Cases) |
| Indicador "Em rua" | Ao confirmar: aviso "N vasilhame(s) registrado(s) como em rua para o cliente" |

**Validações e mensagens**:
- Confirmar com item CASCO_NOVO sem cliente: bloqueio com "Informe o cliente para venda de vasilhame novo." e destaque no seletor (CT-004).
- Bloqueio de estoque da HU-007 continua valendo: "Estoque insuficiente para N unidade(s) de Gás P13. Disponível: M."
- Item com "Vasilhame novo" e item com "Troca de vasilhame" são mutuamente exclusivos por item (comportamentos opostos - HU-009).

## Tela: Clientes (cadastro rápido)

Modal de cadastro rápido dentro do lançamento (nome, telefone, endereco) via `POST /api/clientes` (HU-004); após salvar, o cliente fica selecionado no lançamento (spec.md Edge Cases).

## Controle de comodato (leitura)

Após a venda, o controle por cliente (consulta de "em rua" por cliente, base da HU-011) mostra o vasilhame vendido como "em rua" vinculado ao cliente (CT-003; RF-028). Quando o cliente devolve o vazio (HU-011), o saldo do cliente é baixado.

## Fluxo por CT

| CT | Fluxo na UI |
|---|---|
| CT-001 | Marcar "Vasilhame novo" em um item: o formulário entende a venda sem devolução de vazio |
| CT-002 | Com preços cadastrados (carga 115,00; casco 115,00): o item exibe R$ 230,00 = casco + carga |
| CT-003 | Confirmar venda de vasilhame novo de 2 unidades: saldo de cheios -2, "em rua" +2 para o cliente (conferir no painel de estoque e no controle do cliente) |
| CT-004 | Tentar confirmar sem cliente: bloqueio "Informe o cliente para venda de vasilhame novo."; cadastrar cliente rápido e concluir sem sair do lançamento |

## Nota de integração

- HU-009 (troca) e HU-010 (vasilhame novo) são opções opostas no mesmo formulário; o item sem marcação é venda avulsa da HU-007.
- A baixa do "em rua" na devolução do vazio é a HU-011; a tela de vendas não oferece ajuste manual do comodato.
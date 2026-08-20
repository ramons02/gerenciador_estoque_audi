# Contrato de UI: Limite Mínimo de Estoque (Alerta)

**HU**: HU-003 - Limite Mínimo de Estoque (Alerta)
**Branch**: `HU-003-Limite-Estoque-Baixo`

Telas: formulário de produto (`src/features/produtos/ProdutosPage.tsx`) para configurar o
limite; painel de estoque (`src/features/estoque/`, HU-012) e dashboard
(`src/features/dashboard/`, HU-019) para exibir o alerta persistente (RF-032, RF-053).
Consumo da API por cliente HTTP tipado (`src/api/produtos.ts`). Vocabulário canônico
(CONSTITUICAO §II): os saldos são sempre de cheios, vazios e "em rua".

## 1. Formulário de produto (campo de limite)

| Campo | Tipo | Obrigatório | Observação |
|---|---|---|---|
| Limite mínimo de cheios | Numérico | não (default 0) | 0 = sem alerta |

**Ações:** "Salvar" (envia o formulário completo do produto) e, no painel de estoque, ação
rápida "Definir limite" que chama `PUT /api/produtos/{id}/limite-minimo`.

**Validações inline:**
- Valor negativo: "O limite mínimo de estoque não pode ser negativo." (bloqueia o salvar).

## 2. Alerta visual de estoque baixo (CT-002)

| Elemento | Descrição |
|---|---|
| Onde | Painel de estoque (HU-012) e dashboard (HU-019), visível de forma permanente enquanto ativo |
| Conteúdo | Por produto: nome, saldo de cheios, limite configurado e aviso de reposição |
| Condição | `estoqueBaixo = true` vindo da API (limite > 0 e cheios <= limite) |
| Persistência | O alerta permanece visível ao navegar entre telas e entre sessões, até a dispensa (CT-002) |
| Múltiplos alertas | Um alerta por produto abaixo do limite, exibidos simultaneamente |

**Exemplo de apresentação:**

```text
Gás P13: estoque cheio 15 de 20 (limite) - aviso: hora de comprar mais
Água Galão 20L: estoque cheio 10 de 12 (limite) - aviso: hora de comprar mais
```

## 3. Dispensa do alerta (CT-003)

| Ação | Efeito |
|---|---|
| Entrada de caminhão (HU-006) eleva cheios acima do limite | Alerta some automaticamente |
| Entrada de caminhão parcial (não eleva acima do limite) | Alerta permanece |
| Alterar o limite mínimo | Não dispensa nem ativa alerta retroativamente |
| Qualquer outra ação | Nenhuma dispensa; não existe botão de dispensa manual |

## 4. Mensagens

| Situação | Mensagem |
|---|---|
| Sucesso ao definir limite | "Limite mínimo atualizado." |
| Erro de validação | "O limite mínimo de estoque não pode ser negativo." |
| Falha de rede/API | Mensagem da API quando estruturada; caso contrário mensagem genérica em pt-BR |

## 5. Rastreabilidade

- CT-001 (cadastro do limite), CT-002 (alerta persistente), CT-003 (dispensa por
  carregamento), RF-003, RF-032, RGN-004.
- Commits referenciam HU-003 (CONVENTIONS §9).
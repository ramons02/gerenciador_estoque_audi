# Contract UI: Painel de Estoque em Tempo Real (Pátio)

**HU**: HU-012 - Painel de Estoque em Tempo Real (Pátio)
**Tela**: Painel de Estoque | **Rota**: /estoque (menu "Pátio" ou dashboard)

## Descrição

Tela de consulta para o administrador ver a situação do pátio a qualquer momento (RF-030): por produto, as quantidades de Cheios, Vazios (pátio) e Em rua, com destaque visual para produtos em estoque baixo (RF-032, CT-003).

## Layout

- Lista/tabela de produtos ativos, uma linha ou card por produto.
- Colunas: Produto (nome), Cheios, Vazios, Em rua.
- Indicador de alerta: badge "Estoque baixo" e cor de destaque quando `alertaEstoqueBaixo` (CT-003).
- Legenda dos estados conforme vocabulário do domínio (Constituição §II): "Cheio", "Vazio", "Em rua".

## Campos e ações

| Elemento | Comportamento |
|---|---|
| Card do produto | Exibe nome (ex.: Gás P13) e os 3 valores numéricos |
| Badge "Estoque baixo" | Exibido quando `alertaEstoqueBaixo: true` (CT-003) |
| Indicador de carregamento | Esqueleto/skeleton enquanto a consulta roda |
| Sem produtos ativos | Mensagem vazia "Nenhum produto cadastrado." |

## Atualização em tempo real (CT-002)

- O painel reconsulta `GET /api/estoque` automaticamente após qualquer mutação confirmada na sessão (venda, carregamento, devolução), sem recarga manual.
- Reconsulta periódica a cada 30 segundos para capturar mutações de outros terminais.
- Indicador sutil de "atualizado agora" (timestamp da última consulta).

## Regras de exibição

- Produto sem movimentação aparece com saldos 0 (Edge Case do spec).
- Produto sem limite mínimo configurado nunca exibe badge (Edge Case do spec).
- Produto com saldo exatamente no limite mínimo exibe badge (RGN-004).
- Não há formulário nesta tela: é leitura pura (RF-030).
- Consumo da API via cliente HTTP tipado (estoqueApi.ts), nunca fetch solto (CONVENTIONS §7).

## Mensagens

- Erro de rede: "Não foi possível carregar o estoque. Verifique a conexão e tente novamente."
- Sem autorização: mensagem do backend exibida em alerta ("Acesso negado. Perfil administrador é necessário.").
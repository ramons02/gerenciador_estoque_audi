# Pesquisa Técnica: Limite Mínimo de Estoque (Alerta)

**HU**: HU-003 - Limite Mínimo de Estoque (Alerta)
**Fase**: 0 (decisões de arquitetura)
**Branch**: `HU-003-Limite-Estoque-Baixo`

Formato por decisão: **Decision** / **Rationale** / **Alternatives considered**.

---

## Decisão 1: limite_minimo como coluna INTEGER na tab_produto

**Decision**: O limite mínimo de cheios é a coluna `limite_minimo INTEGER NOT NULL DEFAULT 0`
na `tab_produto` (migration V1), editável por produto.

**Rationale**: O CT-001 exige que o cadastro do produto aceite o limite mínimo, e a RGN-004
define o limite como referência do alerta. Manter o limite junto ao produto evita tabela
extra e consulta adicional para comparar com o saldo de cheios (que também é coluna da
`tab_produto`). A DECISÃO TÉCNICA do projeto prevê `tab_estoque` como opção, mas o modelo
vigente concentra saldo e limite no produto.

**Alternatives considered**: `tab_estoque` separada com saldos por produto (estrutura
distinta do schema vigente, exigiria migração e JOIN em toda consulta de alerta); `tab_configuracao`
global (limite seria único para todos os produtos, violando "por produto" do RF-003).

## Decisão 2: Alerta derivado, sem estado persistente

**Decision**: O alerta é calculado no momento da leitura pela condição pura
`estoque_cheios <= limite_minimo` (com limite maior que zero), sem tabela, flag ou evento
persistido de alerta.

**Rationale**: RF-032 define o alerta como condição do saldo em relação ao limite; o estado
derivado elimina risco de divergência entre o dado e o alerta (CONVENTIONS §12) e simplifica
a dispensa (a condição simplesmente deixa de valer). A persistência é desnecessária para a
escala (1 a 5 usuários, RNF-010) e para o requisito de "alerta persistente" (CT-002), que se
refere à exibição contínua na tela, não ao armazenamento.

**Alternatives considered**: tabela de alertas com flag "ativo" (estado duplicado que pode
dessincronizar do saldo real; exige rotina de reconciliação); evento/notificação em memória
(perde o alerta entre sessões, contrariando o edge case "persiste entre sessões/reinícios").

## Decisão 3: Condição "atingir ou ficar abaixo" como menor ou igual

**Decision**: O alerta dispara com `estoque_cheios <= limite_minimo` (menor ou igual),
cobrindo o caso de saldo exatamente igual ao limite.

**Rationale**: A RGN-004 e o CT-002 usam "ao atingir ou ficar abaixo" do limite; o edge case
da spec registra explicitamente que "estoque exatamente igual ao limite dispara o alerta".
A lógica pura `estoqueBaixo()` no modelo é testada diretamente, sem mock (CONSTITUICAO §XI.2,
CONVENTIONS §11).

**Alternatives considered**: condição estritamente menor (`<`) (perderia o disparo no limite
exato, contrariando a spec e o edge case).

## Decisão 4: Produtos sem limite não geram alerta

**Decision**: O alerta só existe quando `limite_minimo > 0`; o default 0 significa "sem
alerta" mesmo com saldo zerado.

**Rationale**: O edge case da spec diz que "produtos com limite não definido não geram
alerta". Como o default de `tab_produto` é 0, a condição precisa excluir o limite 0 para não
alertar produto recém-cadastrado com estoque zerado (caso em que `0 <= 0` seria verdadeiro).
A regra é documentada como derivada dos requisitos (CONVENTIONS §12).

**Alternatives considered**: `limite_minimo` nulo quando não definido (mudança de tipo e
tratamento de nulo em toda consulta); alertar no limite 0 (falso positivo para todo produto
novo).

## Decisão 5: Dispensa exclusivamente por entrada de caminhão

**Decision**: O alerta é dispensado apenas quando um carregamento (HU-006) eleva
`estoque_cheios` acima do limite; não existe dispensa manual nem efeito de alteração do
limite sobre um alerta ativo.

**Rationale**: RGN-004 e CT-003 definem a entrada de caminhão como única via de dispensa; a
spec reforça que "alterar o limite não interfere na condição de dispensa" e que carregamento
parcial mantém o alerta. Como o alerta é derivado, a dispensa acontece naturalmente pelo
recálculo do saldo na transação do carregamento (mesma transação, RNF-005).

**Alternatives considered**: botão "dispensar alerta" no painel (regra inexistente nos
requisitos; proibido criar regra fora deles, CONSTITUICAO §VI.5); dispensa ao alterar o
limite (contradiz a spec).

## Decisão 6: Endpoint dedicado de alteração do limite

**Decision**: A alteração do limite usa `PUT /api/produtos/{id}/limite-minimo` com validação
de valor não negativo no Service (transacional).

**Rationale**: Separar a operação do cadastro completo permite ao painel de estoque ajustar o
limite sem reenviar preços; a validação de negócio (`limite >= 0`) fica no Service
(CONVENTIONS §6), com mensagem pt-BR ("O limite mínimo de estoque não pode ser negativo.").

**Alternatives considered**: alterar o limite apenas pelo `PUT /api/produtos/{id}` completo
(exige payload inteiro para uma operação pontual); endpoint sem validação (aceitaria limite
negativo, contrariando a natureza de quantidade).

## Decisão 7: Alerta persistente no painel e no dashboard

**Decision**: A listagem de alertas é derivada da consulta de estoque e exibida de forma
persistente nas telas de painel de estoque (HU-012) e dashboard (RF-032, RF-053), um alerta
por produto abaixo do limite.

**Rationale**: RF-053 exige os alertas ativos no dashboard; o CT-002 exige persistência
visual enquanto a condição valer. Múltiplos produtos abaixo do limite geram alertas
simultâneos (edge case da spec), cada um derivado da mesma consulta.

**Alternatives considered**: componente de alerta com estado local no app (perde a
persistência entre navegações e sessões); endpoint separado de "notificações" (redundante
com a consulta de estoque).
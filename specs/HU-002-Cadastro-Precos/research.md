# Pesquisa Técnica: Cadastro de Preços (Custo e Venda)

**HU**: HU-002 - Cadastro de Preços (Custo e Venda)
**Fase**: 0 (decisões de arquitetura)
**Branch**: `HU-002-Cadastro-Precos`

Formato por decisão: **Decision** / **Rationale** / **Alternatives considered**.

---

## Decisão 1: Preços como colunas NUMERIC na tab_produto

**Decision**: Armazenar `preco_custo` e `preco_venda` como colunas `NUMERIC(12,2)` na própria
`tab_produto` (migration V1), sem tabela de histórico de preços.

**Rationale**: A spec exige que o cadastro "salve ambos os valores junto ao produto" (CT-001)
e que a alteração valha apenas para lançamentos futuros (CT-003). Com a venda congelando o
valor unitário no momento do lançamento, o histórico de preços da venda fica preservado sem
tabela própria; a DECISÃO TÉCNICA do projeto admite explicitamente "colunas na tab_produto:
preco_custo, preco_venda". O tipo `NUMERIC(12,2)` é o padrão para valores monetários
(CONVENTIONS §4, nomenclatura e tipos).

**Alternatives considered**: tabela `tab_produto_preco` com vigência por período (histórico
completo de reajustes: mais complexo do que o CT exige, pois a venda já congelaria o valor
de qualquer forma); `DOUBLE PRECISION` para preços (perda de precisão monetária).

## Decisão 2: Validação RGN-005 no Service, dentro da transação

**Decision**: A regra "preço de venda não pode ser menor que o preço de custo" é aplicada no
`ProdutoService.validar`, executada pelo `BaseService.salvar/atualizar` dentro da transação
de gravação, com mensagem pt-BR.

**Rationale**: RGN-005 define a regra como política do negócio; o CONVENTIONS §6 determina
que regra de negócio vive no Service (nunca no Controller). A validação é executada no
cadastro e na alteração (CT-002), e a mensagem "O preço de venda não pode ser menor que o
preço de custo." é devolvida como erro estruturado 422 (CONSTITUICAO §XI.3).

**Alternatives considered**: validação no Controller com bean validation (validação sintática,
mas a regra de comparação entre dois campos é negócio, não formato); validação apenas no
frontend (contornável via API, viola RGN-005).

## Decisão 3: Snapshot do preço na venda (valor unitário congelado)

**Decision**: No lançamento da venda (HU-007), o `valor_unitario` é gravado na `tab_venda`
com o preço vigente no momento; relatórios e fechamento de caixa usam esse valor, nunca o
preço atual do produto.

**Rationale**: O CT-003 exige que vendas antigas não sejam recalculadas após alteração de
preço; o RNF-007 exige integridade do histórico financeiro. Congelar o valor no registro da
venda garante fidelidade mesmo com o preço do produto mudando depois (edge case da spec).

**Alternatives considered**: resolver o preço por JOIN com o preço atual do produto na hora
do relatório (vendas antigas apareceriam com o preço novo, violando CT-003); historiar
preços com vigência e buscar o vigente na data da venda (funciona, porém com custo de
complexidade maior que o snapshot).

## Decisão 4: Atualização de preço pelo PUT /api/produtos/{id}

**Decision**: A alteração de preço acontece pelo `PUT /api/produtos/{id}`, que atualiza o
cadastro do produto (incluindo preços), sem endpoint dedicado de preço.

**Rationale**: Preço é atributo do produto (RF-002) e o padrão REST define `PUT` para
atualização de recurso (CONVENTIONS §6). Um único contrato de atualização evita duplicidade
de rotas e mantém a validação RGN-005 concentrada no `ProdutoService`.

**Alternatives considered**: endpoint dedicado `PUT /api/produtos/{id}/precos` (rota extra
sem ganho, pois a validação e o destino são os mesmos); `PATCH` parcial (o padrão do projeto
é `PUT` com corpo completo, CONVENTIONS §6).

## Decisão 5: Bloqueio de valores zerados, negativos e não numéricos

**Decision**: Preços devem ser maiores que zero (rejeitando zero, negativo e não numérico) e
a comparação usa `BigDecimal.compareTo`, sem arredondamento implícito.

**Rationale**: O edge case da spec exige bloquear valores zerados, negativos ou não
numéricos no campo de preço; a RGN-005 permite venda igual ao custo, mas não abaixo. O uso de
`BigDecimal` evita erros de aritmética de ponto flutuante em relatórios financeiros
(RF-041/043).

**Alternatives considered**: aceitar zero como "preço a definir" (contaminaria venda e
fechamento de caixa com valor sem sentido); `double` com comparação direta (risco de
precisão).

## Decisão 6: Alteração de preço liberada a qualquer momento

**Decision**: Não há trava de edição de preço: o produto pode ter preços alterados a qualquer
momento, e a alteração vale apenas para lançamentos futuros.

**Rationale**: O CT-003 exige explicitamente essa liberdade de alteração; o RGN-005 é a única
restrição sobre o valor. Como o snapshot na venda (Decisão 3) protege o histórico, não há
motivo de negócio para bloquear a edição.

**Alternatives considered**: bloquear alteração de preço quando existirem vendas no dia
(regra inexistente nos requisitos; seria regra de negócio inventada, proibida pela
CONSTITUICAO §VI.5).

---

## Rastreabilidade

- RF-002 (cadastro de custo e venda), RGN-005 (venda não inferior ao custo), CT-001 a CT-003.
# Pesquisa Técnica: Cadastro de Clientes

**HU**: HU-004 - Cadastro de Clientes
**Fase**: 0 (decisões de arquitetura)
**Branch**: `HU-004-Cadastro-Clientes`

Formato por decisão: **Decision** / **Rationale** / **Alternatives considered**.

---

## Decisão 1: tab_cliente com nome obrigatório e contato opcional

**Decision**: `tab_cliente` (migration V3) com `nome` obrigatório e `telefone` e `endereco`
opcionais; o campo `documento` foi removido do schema (migration V6, por LGPD).

**Rationale**: O CT-001 exige nome, telefone, endereço e documento (opcional); a migration
V6 removeu `documento` para atender à LGPD (não armazenar CPF/CNPJ). O cadastro captura
identificação e contato mínimos para o controle de vasilhames "em rua" (RF-028) e venda de
vasilhame novo (RF-024), com nome como único campo obrigatório (edge case: cliente sem
documento é aceito).

**Alternatives considered**: manter `documento` na tabela (contraria a decisão LGPD já
aplicada na migration V6); tornar telefone/endereço obrigatórios (o CT-001 não exige; a
spec diz que documento é o único opcional, mas o schema vigente mantém contato e endereço
como opcionais).

## Decisão 2: Busca por nome ou telefone via parâmetro q

**Decision**: A busca do cliente usa `GET /api/clientes?q=<termo>`, que filtra por nome ou
telefone (consulta no `ClienteRepository`), retornando apenas clientes ativos.

**Rationale**: O CT-002 exige busca por nome ou telefone no momento da venda, sem sair da
tela de venda (RNF-001). Um único parâmetro cobre os dois campos, com termo parcial, e o
resultado alimenta o seletor da venda.

**Alternatives considered**: busca client-side na lista completa (forçaria carregar todos os
clientes em toda abertura de venda, contrário ao lançamento rápido RNF-001); dois parâmetros
separados (`?nome=` e `?telefone=`) (mais complexo sem ganho, pois um único termo já resolve).

## Decisão 3: Validação de negócio no Service

**Decision**: A validação de nome obrigatório fica no `ClienteService.validar`, dentro da
transação do `BaseService` (salvar/atualizar), com mensagem pt-BR.

**Rationale**: Regra de negócio no Service conforme CONVENTIONS §6; mensagem "Informe o nome
do cliente." devolvida como erro estruturado 422 (CONSTITUICAO §XI.3).

**Alternatives considered**: validação apenas com bean validation no Controller (cobre o
formato, mas a regra de cadastro é de negócio e deve ser a fonte única no Service).

## Decisão 4: Exclusão lógica com histórico de "em rua" preservado

**Decision**: A exclusão de cliente usa `DELETE /api/clientes/{id}` que marca `ativo =
false` (exclusão lógica via `BaseService.excluirLogico`).

**Rationale**: O cliente pode ter vasilhames "em rua" vinculados (RF-028) e vendas
históricas; apagar fisicamente quebraria a rastreabilidade dos vasilhames "em rua" (RNF-007). A exclusão
lógica mantém o registro e o histórico, e o cliente deixa de aparecer na busca.

**Alternatives considered**: `DELETE` físico quando sem vínculos (quebra o padrão de
exclusão lógica do CONVENTIONS §6 e arrisca perder histórico); bloqueio de exclusão quando
houver vasilhames em rua (regra inexistente nos requisitos para esta feature).

## Decisão 5: Cliente obrigatório na venda de vasilhame novo no fluxo de venda

**Decision**: A obrigatoriedade do cliente na venda de vasilhame novo (CT-003) é aplicada no
`VendaService` (HU-007), que rejeita a confirmação sem cliente e vincula o vasilhame "em
rua" ao cliente; o cadastro de clientes apenas fornece o dado.

**Rationale**: A regra de obrigatoriedade pertence ao fluxo de venda (RF-024/RF-026), não ao
cadastro; o CONVENTIONS §12 manda codificar a regra onde o requisito a define. O vínculo de
"em rua" usa `tab_cliente_vasilhame` (migration V3), por cliente e vasilhame (RF-028,
RDN-008).

**Alternatives considered**: bloquear o cadastro de venda por regra no Controller de clientes
(sem sentido: o cadastro não sabe da venda); validar apenas na tela de venda (contornável
via API, viola a regra).

## Decisão 6: Sem forma de pagamento Fiado

**Decision**: O cadastro e o fluxo de venda não oferecem a forma de pagamento Fiado; as
formas aceitas são configuráveis e não incluem Fiado (RGN-002, RF-052).

**Rationale**: A RGN-002 define que Fiado não existe no sistema (CONSTITUICAO §XIII.1). A
lista de formas de pagamento vem de `tab_configuracao` (migration V9) e é consumida pela
venda (HU-007), garantindo que nenhuma venda use Fiado (CT-003).

**Alternatives considered**: incluir Fiado como forma desabilitada (viola RGN-002 e a
Constituição; reativação exigiria emenda).

## Decisão 7: Vinculação de "em rua" por cliente e vasilhame

**Decision**: O controle de vasilhames "em rua" usa `tab_cliente_vasilhame` com a combinação
única `(cliente_id, vasilhame_id)` e contagem, atualizada pelo `ClienteVasilhameService` nos
fluxos de venda (HU-007) e devolução (HU-011).

**Rationale**: O RF-028 exige controlar a saída de vasilhames "em rua" por cliente, com
baixa na devolução; a RDN-008 define que "em rua" só aumenta quando o cliente leva um
vasilhame sem devolver vazio equivalente. A unicidade por par `(cliente, vasilhame)` evita
linhas duplicadas por combinação (CONVENTIONS §4).

**Alternatives considered**: contagem agregada global por vasilhame (perde o "por cliente"
do RF-028); tabela de histórico linha a linha por vasilhame (mais granular que o necessário
para o controle de saldo).
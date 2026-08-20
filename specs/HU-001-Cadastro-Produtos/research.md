# Pesquisa Técnica: Cadastro de Produtos (Carga + Vasilhame)

**HU**: HU-001 - Cadastro de Produtos (Carga + Vasilhame)
**Fase**: 0 (decisões de arquitetura)
**Branch**: `HU-001-Cadastro-Produtos`

Formato por decisão: **Decision** / **Rationale** / **Alternatives considered**.

---

## Decisão 1: Composição carga + vasilhame por entidades de referência

**Decision**: Modelar `tab_carga` e `tab_vasilhame` como tabelas de referência próprias e
`tab_produto` com chaves estrangeiras `carga_id` e `vasilhame_id`, mais unicidade
`UNIQUE (carga_id, vasilhame_id)`.

**Rationale**: A RDN-001 define todo item vendido como composição Carga/Conteúdo +
Vasilhame/Casco, e a RDN-007 define capacidades fixas por carga, controladas pela
distribuidora. Entidades de referência garantem que só existam combinações válidas e
permitem a constraint de unicidade (CT-003) diretamente no banco (RNF-008). A tabela
`tab_produto` nasce da migration `V1__criar_tabelas_base.sql` com essa forma
(CONVENTIONS §4 e §8).

**Alternatives considered**: campo texto livre com nome do produto (perde a semântica de
composição e a unicidade estrutural, violaria RDN-001); colunas `carga` e `vasilhame` como
texto dentro de `tab_produto` (permitiria duplicidade por variação de escrita e não
modelaria a capacidade fixa da carga).

## Decisão 2: Nome combinado derivado, não persistido

**Decision**: O nome do produto é derivado em tempo de execução pela concatenação da carga e
do vasilhame (`getNome()` retorna `carga.getNome() + " " + vasilhame.getNome()`), sem coluna
própria em `tab_produto`.

**Rationale**: O CT-002 exige a exibição pelo nome combinado (ex.: "Gás P13"). Persistir o
nome derivado criaria redundância com risco de divergência entre o nome e a composição real.
A derivação garante exibição sempre consistente (CONVENTIONS §12: regra derivada dos
requisitos, sem estado duplicado).

**Alternatives considered**: coluna `nome` persistida e atualizada manualmente no cadastro
(risco de divergência); armazenamento do nome na criação (exigiria sincronização ao editar a
carga ou o vasilhame).

## Decisão 3: Exclusão lógica com bloqueio por movimentações vinculadas

**Decision**: A exclusão é lógica (`ativo = false`), via `DELETE /api/produtos/{id}` que
marca inativo, e é bloqueada quando o produto possui vendas ou carregamentos vinculados.

**Rationale**: O CT-004 permite excluir somente produto sem movimentações; com movimentações,
o sistema bloqueia e preserva o histórico. A RNF-007 proíbe apagar registros de movimento, e
o CONVENTIONS §6 padroniza `DELETE` como marcação de inativo. O bloqueio acontece no Service
(`ProdutoService.excluirLogico`), consultando `tab_venda` e `tab_carregamento_item`, para que
nenhuma outra tela contorne a regra (edge case da spec).

**Alternatives considered**: `DELETE` físico quando não houver movimentações (quebra o padrão
de exclusão lógica do CONVENTIONS §6 e perde o rastro de auditoria do RNF-007); exclusão sem
verificação (violaria CT-004).

## Decisão 4: Validação de negócio no Service + constraint no banco

**Decision**: A validação de carga/vasilhame obrigatórios e de duplicidade fica no
`ProdutoService.validar`, executada dentro da transação de gravação, com a constraint
`uk_produto_carga_vasilhame` como garantia final no banco.

**Rationale**: O CONVENTIONS §6 concentra regra de negócio no Service; a mensagem de erro em
pt-BR ("Já existe um produto com a combinação de carga e vasilhame informada.") é emitida
como erro estruturado (CONVENTIONS §6 e §XI). A constraint no banco cobre a janela de
concorrência entre a checagem e a gravação (RNF-008). A validação dupla (aplicação + banco) é
barata para uma revenda de 1 a 5 usuários simultâneos (RNF-010).

**Alternatives considered**: validação apenas com bean validation no Controller (não cobre a
duplicidade, que exige consulta); validação apenas no banco (mensagem de erro crua do
PostgreSQL, sem pt-BR amigável).

## Decisão 5: Persistência com Spring Data JPA

**Decision**: Persistência com Spring Data JPA, repositórios por entidade
(`ProdutoRepository`, `CargaRepository`, `VasilhameRepository`), com consultas derivadas de
método (ex.: `findByCargaIdAndVasilhameIdAndAtivoTrue`).

**Rationale**: A stack definida usa Spring Data JPA (DECISÕES TÉCNICAS), o mapeamento das
entidades é direto e o volume de dados é pequeno (revenda única, RNF-010). O mapeamento JPA
das três entidades está nas migrations V1 (CONVENTIONS §8).

**Alternatives considered**: JDBC manual com SQL escrito à mão (aumenta código e risco de
erro sem ganho de performance relevante para a escala); JOOQ (dependência nova fora da stack
definida).

## Decisão 6: Endpoints REST por recurso

**Decision**: API REST com rotas por recurso: `GET/POST /api/cargas`, `GET/POST /api/vasilhames`
e `GET/POST/PUT/DELETE /api/produtos`, com respostas JSON do próprio recurso.

**Rationale**: O CONVENTIONS §4 e §6 definem rotas `/api/<modulo>` minúsculas e o padrão
REST (GET listar/obter, POST criar, PUT atualizar, DELETE marcar inativo). A separação por
recurso permite ao app carregar as listas de carga e vasilhame para montar o formulário de
produto (CT-001).

**Alternatives considered**: estilo RPC com verbos na rota (`/api/produtos/criar`,
`/api/produtos/listar`) (diverge do CONVENTIONS §6); endpoints únicos por tela que misturam
recursos (acoplamento API/UI).

## Decisão 7: Estados de auditoria e listagem por ativo

**Decision**: Toda tabela herda de `BaseModel` as colunas `criado_em`, `atualizado_em` e
`ativo`, e as listagens retornam somente registros ativos (`findByAtivoTrue`).

**Rationale**: A exclusão lógica (Decisão 3) exige a flag `ativo` para o filtro de listagem;
produtos inativos não aparecem na venda nem na listagem, mas permanecem no banco para
preservar o histórico de movimentações (RNF-007). As colunas de auditoria atendem ao rastro
de data/hora exigido pelo RNF-007 e ao padrão do CONVENTIONS §4.

**Alternatives considered**: tabela de auditoria separada por entidade (sobrecarga desnecessária
para a escala); remoção física com backup (viola RNF-007 e CONVENTIONS §5.2).
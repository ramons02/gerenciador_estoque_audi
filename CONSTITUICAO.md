# Constituição do Premium Gás

Este documento define os princípios não negociáveis do projeto. Toda decisão, código e
documentação DEVE respeitar esta Constituição. Em caso de conflito, a Constituição prevalece
sobre os demais documentos.

**Versão:** 1.3.0
**Ratificação:** 2026-08-17
**Última emenda:** 2026-08-19

---

## §I. Identidade do Projeto

- **Projeto:** Premium Gás - revenda de Gás (GLP) e Água com vasilhame retornável.
- **Estrutura:** monorepo em repositórios irmãos:
  - `gerenciador_estoque_api` - backend.
  - `gerenciador_estoque_app` - frontend.
  - `gerenciador_estoque_infra` - infraestrutura e banco de dados.
  - `gerenciador_estoque_audi` - documentação e auditoria (requisitos, HUs, convenções, constituição).
  - `gerenciador_estoque_jar` - JAR executável (privado, com credenciais embutidas).
- **Fonte da verdade de comportamento:** `gerenciador_estoque_audi/requisitos/REQUISITOS.md`.
  Nenhuma tela, regra ou relatório pode divergir dele sem emendar primeiro a Constituição (§XIV).
- **Fonte da verdade de entrega:** as HUs em `gerenciador_estoque_audi/HUs/`. Toda implementação
  DEVE rastrear seus critérios de aceitação (CT-xxx) até a HU correspondente.
- **Fonte da verdade de implementação:** as pastas `gerenciador_estoque_audi/specs/<HU>/` com
  `spec.md`, `plan.md` e `task.md`. Toda implementação DEVE partir desses artefatos (§V).

## §I-A. Artefatos por HU (spec/plan/task)

Toda HU em `HUs/` tem uma pasta correspondente em `specs/<HU>/` com três artefatos
obrigatórios, criados ANTES de qualquer código:

| Artefato | Papel |
|---|---|
| `spec.md` | Especificação: objetivo, critérios de aceitação (CT) e requisitos vinculados. |
| `plan.md` | Plano de implementação: escopo, decisões de arquitetura e etapas. |
| `task.md` | Tarefas executáveis: checklist de CTs com status (pendente/em andamento/concluído). |

**Regras:**
1. Nenhuma HU entra em desenvolvimento sem os 3 artefatos criados e consistentes com a HU.
2. `task.md` só move um critério para "concluído" com CT provado (teste ou evidência).
3. `spec.md` é a fonte do comportamento esperado; divergência entre spec e código é defeito.
4. Alterar comportamento da HU = alterar HU + spec na mesma entrega.

## §II. Domínio: vocabulário obrigatório

Os termos abaixo são a linguagem única do projeto. Proibido usar sinônimos (ex.: "botijão" em vez
de "vasilhame", "galão" em vez de "carga") fora das telas de domínio quando o termo canônico existe.

| Termo | Definição |
|---|---|
| Carga / Conteúdo | O produto em si (Gás ou Água). |
| Vasilhame / Casco | O recipiente retornável (P13, P45, Galão 20L). |
| Cheio | Vasilhame pronto para venda (no estoque). |
| Vazio | Vasilhame no pátio aguardando recarga/devolução. |
| Em rua | Vasilhame com cliente em haver/comodato. |
| Troca | Venda onde o cliente entrega 1 vazio e leva 1 cheio. |
| Carregamento | Entrada de caminhão (cheios recebidos + vazios devolvidos). |

## §III. Estados do estoque (invariantes)

1. Todo vasilhame existente está em exatamente um estado: **Cheio**, **Vazio** ou **Em rua**.
2. Estoque nunca pode ficar negativo em nenhum estado.
3. Toda movimentação altera estoque e caixa na MESMA transação. Estado parcial é defeito grave.
4. Nenhuma venda ou entrada pode ser apagada: apenas cancelada/estornada, com rastro de auditoria.

## §IV. Fonte da verdade sobre dados

1. Afirmação sobre dado se prova na base de dados, nunca no código ou no que a tela mostra.
2. Contagem antes de conclusão: `SELECT count(*)` na tabela de origem antes de declarar
   "não existe", "está vazio" ou "é falta de massa".

## §V. Desenvolvimento orientado a requisitos

1. Toda funcionalidade DEVE nascer de uma HU em `HUs/`, que referencia requisitos em
   `requisitos/REQUISITOS.md`. Funcionalidade sem HU não existe.
2. Toda HU DEVE ter os artefatos `spec.md`, `plan.md` e `task.md` em `specs/<HU>/` antes de
   qualquer código (§I-A).
3. Critério de aceitação fechado = CT (Critério de Teste) aprovado e provado, não apenas
   código escrito.
4. Hipótese entra marcada como hipótese, ou não entra. Toda suposição de causa DEVE ser
   provada antes de ser gravada como explicação.

## §VI. Proibido

1. ❌ Apagar registros de movimento (venda, entrada, devolução). Só cancelamento com reversão.
2. ❌ Atualizar estoque fora de transação atômica com o registro do movimento.
3. ❌ Vender com estoque insuficiente (estoque negativo) - mesmo sob pressão de tempo.
4. ❌ Divergir do vocabulário do domínio (§II) em código e documentação.
5. ❌ Implementar regra de negócio que não esteja documentada em requisitos/HUs sem antes
   registrar a emenda (§XIV).
6. ❌ Incluir referência a ferramenta de IA em commits, MRs ou documentação (§IX).

## §VII. Documentação

1. Toda convenção nova ou descoberta no código DEVE ser registrada em `CONVENTIONS.md`.
2. Toda regra que a equipe decidiu não seguir DO O QUE ESTÁ PREVISTO nos requisitos exige
   emenda constitucional (§XIV) - nada de exceção silenciosa.
3. Documentos são escritos em português (pt-BR).
4. Comunicação escrita (código, commit, docs) usa hífen normal. Proibido travessão (em-dash).

## §VIII. Auditoria

1. `gerenciador_estoque_audi/` é o local canônico de auditoria: requisitos, HUs, decisões e
   relatórios de QA.
2. Fechar um item de QA exige reauditoria contra o código na branch principal - nunca contra o
   resumo do que foi feito.

## §IX. Commits e MRs

1. Mensagens descrevem a MUDANÇA no domínio do sistema, em português, formato
   `tipo(escopo): descrição` (ex.: `feat(entrada): registra chegada de caminhão`).
2. Proibido: referência a IA/assistente, coautoria de ferramenta, tipo `chore` sem valor.
3. Autor do commit é a pessoa (identidade git do desenvolvedor).

## §X. Rastreabilidade de requisitos

1. Todo código entrega DEVE referenciar o ID do requisito (RF-xxx) ou da HU (HU-xxx) no
   commit e, quando aplicável, na documentação do módulo.
2. Regra de negócio implementada sem RF correspondente é dívida técnica: registrar no
   `requisitos/REQUISITOS.md` na mesma entrega.

## §XI. Qualidade

1. Nenhuma entrega sem provar os critérios de aceitação da HU (por teste automatizado ou
   evidência de execução registrada).
2. Lógica pura é testada diretamente (sem mocks). Regras de estoque e caixa têm prioridade
   máxima de cobertura.
3. Erro de validação não pode colapsar em estado de negócio (ex.: venda bloqueada virar
   "sistema indisponível").

## §XII. Falha de processo

1. Bug de processo tem prioridade sobre bug de código: se o processo permitiu o erro, o
   processo muda primeiro.
2. Todo defeito classificado como "falta de massa de teste" exige prova da ausência no dado
   (§IV.2) antes de ser arquivado.

## §XIII. Segurança

1. A forma de pagamento **Fiado não existe no sistema** (RGN-002). Não reativar sem emenda.
2. Cancelamento/estorno registra quem, quando e o motivo. Sem exceção.

## §XIV. Emendas e Governança

1. **Alterar a Constituição** (incluindo regras novas de "não fazer") exige:
   - Registrar a emenda nesta seção com data, autor e justificativa;
   - Atualizar versão semântica: MAJOR para remoção/redefinição de princípio, MINOR para
     princípio/seção nova, PATCH para esclarecimento de redação.
2. **Alterar regra de negócio** (requisitos): emendar `requisitos/REQUISITOS.md` e atualizar a
   HU afetada na mesma entrega.
3. **Rever a Constituição:** revisão obrigatória a cada 6 meses ou a cada mudança MAJOR de
   escopo do produto.
4. Conflito entre documentos resolve nesta ordem: Constituição > REQUISITOS.md > HUs > CONVENTIONS.md.

### Histórico de emendas

| Versão | Data | Autor | Mudança |
|---|---|---|---|
| 1.0.0 | 2026-08-17 | - | Ratificação inicial. |
| 1.1.0 | 2026-08-17 | - | Adicionado §I-A (artefatos spec/plan/task por HU) e §V.2. |
| 1.2.0 | 2026-08-18 | - | Renomeado projeto para Premium Gás (§I) e inclusão do repositório jar. |
| 1.3.0 | 2026-08-19 | - | Removida a regra de venda fiado (§XIII.1) e RGN-002 reescrito: Fiado não existe; cartão é forma única com acréscimo fixo configurável (RF-021/021-A/051/052). |

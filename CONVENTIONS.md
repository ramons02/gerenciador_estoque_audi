# Gerenciador de Estoque - Convenções

Fonte única das convenções técnicas e de processo do projeto. Todo agente ou pessoa que
trabalha neste repositório DEVE ler e seguir estas convenções. Em caso de conflito com a
Constituição, vale a Constituição (ver `CONSTITUICAO.md`).

---

## 1. Estrutura do repositório

```
Gerenciador de Estoque/
  gerenciador_estoque_api/    # Backend: Java (Spring Boot)
  gerenciador_estoque_app/    # Frontend: React (TypeScript)
  gerenciador_estoque_infra/  # Infraestrutura, banco (PostgreSQL), deploy
  gerenciador_estoque_audi/   # Documentação e auditoria (NUNCA código)
    requisitos/REQUISITOS.md  # Requisitos funcionais, não funcionais, RDN e RGN
    HUs/HU-xxx-*.md           # Histórias de usuário com critérios de aceitação
    specs/HU-xxx-*/           # Artefatos por HU (obrigatórios antes do código):
      spec.md                 #   especificação: objetivo, CTs, requisitos
      plan.md                 #   plano: escopo, arquitetura, etapas
      task.md                 #   tarefas: checklist de CTs com status
    CONSTITUICAO.md           # Princípios não negociáveis
    CONVENTIONS.md            # Este documento
```

**Regras de estrutura:**
- `audi` é SOMENTE LEITURA de código: só documentação e auditoria. Nunca editar código ali.
- Toda funcionalidade nasce de uma HU em `HUs/`; a HU referencia os RFs em `requisitos/`.
- Toda HU DEVE ter sua pasta `specs/<HU>/` com `spec.md`, `plan.md` e `task.md` criados
  ANTES de qualquer código (Constituição §I-A).
- Documentação nova ou descoberta de convenção DEVE ser registrada aqui (§Convenções).

---

## 2. Vocabulário do domínio (obrigatório)

Usar SEMPRE os termos canônicos (Constituição §II). Proibido sinônimos:

| Termo canônico | Sinônimos proibidos |
|---|---|
| Carga / Conteúdo | produto, mercadoria, "o gás", "o galão" (quando referindo à carga) |
| Vasilhame / Casco | botijão, galão, recipiente, garrafa |
| Cheio | cheio, estocado (estado: pronto para venda) |
| Vazio | vasilha, "o vazio" em texto técnico fora do contexto de pátio |
| Em rua | comodato, em haver, na rua do cliente |
| Troca | permuta, "devolve e leva" |
| Carregamento | chegada, compra, abastecimento |
| Pátio | depósito de vazios, área de vazios |

---

## 3. Stack de tecnologia

| Camada | Tecnologia |
|---|---|
| Backend | Java + Spring Boot (REST API) |
| Frontend | React + TypeScript |
| Banco de dados | PostgreSQL |
| Migrações de banco | Flyway (forward-only) |

## 4. Nomenclatura de código

| Conceito | Padrão |
|---|---|
| Classes (Java/TS) | `PascalCase` |
| Métodos/funções | `camelCase` |
| Constantes/enums | `SCREAMING_SNAKE_CASE` |
| Rotas de API | `/api/<modulo>` minúsculo, sem acento |
| Pacotes Java | `com.gerenciador.estoque.<modulo>` minúsculo, sem acento |
| Componentes React | `PascalCase.tsx`, pasta por feature |
| Tabelas | `tab_<entidade>` |
| Sequências | `seq_<entidade>` |
| Colunas | `snake_case` |
| Migrations | `V<N>__descricao_snake_case.sql` (Flyway, forward-only) |

Módulos candidatos (conforme requisitos): `produto`, `cliente`, `fornecedor`, `entrada`
(carregamento), `venda` (saída), `estoque` (pátio), `relatorio`, `dashboard`, `caixa`.

---

## 5. Regras de estoque e transações (não negociáveis)

1. Toda movimentação de estoque acontece NA MESMA transação do registro do movimento
   (venda/entrada/devolução). Proibido transação separada (Constituição §III.3).
2. Nunca usar `DELETE` em tabela de movimento (venda, entrada, devolução). Só cancelamento
   com reversão e status "cancelado".
3. Estoque nunca pode ficar negativo. Validar saldo ANTES de gravar, e garantir
   serialização de concorrência (lock por produto na transação).
4. Venda com troca: baixa 1 cheio + adiciona 1 vazio ao pátio na MESMA operação atômica.
5. Custo médio recalculado a cada carregamento, no mesmo commit da entrada.

---

## 6. API (backend - Java + Spring Boot)

- **Stack:** Java 21 + Spring Boot 3.x, Maven ou Gradle (a definir no bootstrap).
- **Estrutura de pacotes:** `com.gerenciador.estoque.<modulo>` - Controller (REST), Service
  (regra de negócio + `@Transactional`), Repository (JPA/JDBC), dto/ (Request/Response).
- **Persistência:** Spring Data JPA ou JDBC (a definir no bootstrap); o uso de Flyway (§8) é
  obrigatório em qualquer caso.
- Padrão REST: `GET` listar/obter, `POST` criar, `PUT` atualizar, `DELETE` marcar cancelado
  (nunca apagar fisicamente - ver §5.2).
- Validação de negócio retorna erro claro com mensagem em pt-BR (ex.: "Estoque insuficiente
  para 3 unidade(s) de Gás P13. Disponível: 1.").
- Toda resposta de erro DEVE ser estruturada (nunca texto puro/HTML).
- Endpoint de venda DEVE ser transacional e reverter estoque+caixa se qualquer passo falhar.
- IDs de referência: HU-xxx e RF-xxx nos commits e, quando aplicável, na doc do endpoint.

---

## 7. Frontend (app - React + TypeScript)

- **Stack:** React + TypeScript, Vite (a definir no bootstrap). Estado: Context API ou
  biblioteca dedicada (a definir no bootstrap).
- Telas seguem as HUs como contrato de comportamento (critérios de aceitação = spec de UI).
- Dashboard (RF-050 a RF-053) é a tela inicial; alerta de estoque baixo (RF-032) visível
  permanentemente enquanto ativo.
- Lançamento de venda deve ser rápido: mínimo de cliques, validação inline, sem navegação
  desnecessária (RNF-001).
- Exportações (RF-040 a RF-044) usam cabeçalhos em pt-BR e formato aberto Excel/CSV (RNF-009).
- Mensagens de erro amigáveis e consistentes, em pt-BR.
- Consumo da API: cliente HTTP tipado (types por endpoint), nunca `fetch` solto no componente.

---

## 8. Banco de dados (PostgreSQL)

- PostgreSQL é o ÚNICO banco suportado (Constituição §I). Nunca usar dialeto de outro banco
  (sem `NVL`, `SYSDATE`, `ROWNUM`, etc.).
- Toda mudança de schema entra como migration Flyway versionada no repositório, forward-only.
- Migration já aplicada não se corrige editando: exige migration nova.
- Nomes em snake_case, em pt-BR (`tab_venda`, `coluna_qtd_cheios`, etc.).
- Dados de configuração (taxa de entrega, limite mínimo) ficam em tabela própria, editável
  pelo admin, nunca hardcoded no código.
- Transações de estoque usam lock por produto (consultar e atualizar na MESMA transação) para
  garantir a invariante de estoque não negativo sob concorrência (§5.3).

---

## 9. Commit e MR

- Formato: `tipo(escopo): descrição` - ex.: `feat(entrada): registra chegada de caminhão`.
- Tipos permitidos: `feat`, `fix`, `refactor`, `test`, `docs`, `perf`, `build`, `ci`.
- PROIBIDO: `chore`, referência a IA/assistente, coautoria de ferramenta, travessão (em-dash).
- Mensagem em pt-BR, descrevendo a mudança no domínio do sistema (Constituição §IX).
- MRs referenciam HU e RF quando aplicável: `Closes HU-009` / `RF-025`.

---

## 10. Relatórios e exportação (contrato de formato)

Planilhas geradas conforme RF-041/042/043, com estas colunas EXATAS:

**Relatório de Vendas (RF-041):** Data/Hora, Produto, Qtd, Valor Unitário, Total (R$),
Forma de Pagamento, Tipo (Balcão/Entrega).

**Relatório de Carregamentos (RF-042):** Data, Fornecedor, Produto, Qtd Cheios Entraram,
Qtd Vazios Saíram, Custo Total.

**Fechamento de Caixa e Balanço (RF-043):** Produto, Estoque Inicial, (+) Entradas,
(-) Vendas, Estoque Final, Vazios em Pátio.

Períodos: Hoje, Últimos 7 dias, Mês Atual, personalizado (RF-040). Botão de exportação em
cada relatório (RF-044).

---

## 11. Testes e QA

- Critério de aceitação = CT aprovado e provado (Constituição §V.2). Evidência registrada.
- Regras de estoque e caixa têm prioridade máxima de cobertura (transação, reversão,
  bloqueio por saldo).
- Teste de concorrência: vendas simultâneas não podem gerar estoque negativo (RNF-008).
- Toda correção de bug DEVE ter teste que reproduz o defeito antes de corrigir (regressão).
- Não usar mocks para lógica pura (cálculo de total, custo médio, reversão de estoque).

---

## 12. Regras de negócio em código (de onde vêm)

As regras de negócio NUNCA são inventadas no código. Elas vêm de:

1. `requisitos/REQUISITOS.md` (RGN-xxx) - regras de negócio e RDN-xxx (regras de domínio).
2. `HUs/` - critérios de aceitação por funcionalidade.

Se a regra não existe em nenhum dos dois: REGISTRAR no requisito na mesma entrega
(Constituição §X.2) - nunca codificar "no automático" e documentar depois.

---

## 13. Convenções de processo (feitas/não fazer)

**FAZER:**
- Ler `CONSTITUICAO.md` e `CONVENTIONS.md` antes de qualquer trabalho.
- Provar toda afirmação de dado no banco (§4 da Constituição).
- Registrar decisão nova na Convenção correspondente.

**NÃO FAZER:**
- ❌ Não editar `audi/` com código.
- ❌ Não criar regra de negócio fora dos requisitos.
- ❌ Não apagar movimento (venda/entrada) com DELETE.
- ❌ Não atualizar estoque fora de transação atômica.
- ❌ Não vender sem validar saldo.
- ❌ Não usar termos fora do vocabulário do domínio.
- ❌ Não usar travessão (—) em texto; sempre hífen.
- ❌ Não citar IA em commits/MRs/docs.

---

## 14. Fluxo de trabalho padrão (feature)

1. HU existe em `HUs/` com CTs claros.
2. Criar `specs/<HU>/` com `spec.md`, `plan.md` e `task.md` ANTES de qualquer código
   (Constituição §I-A). Spec deriva da HU; plan define arquitetura; task é o checklist.
3. Implementar com referência explícita à HU/RF.
4. Rodar testes e provar cada CT.
5. Atualizar o status no `task.md` (pendente → em andamento → concluído) a cada CT provado.
6. Commit no formato `tipo(escopo): descrição` + `Closes HU-xxx`.
7. Registrar evidência em `audi/` se aplicável (QA).

---

## 15. Histórico de mudanças desta convenção

| Data | Mudança |
|---|---|
| 2026-08-17 | Criação inicial. |
| 2026-08-17 | Definição da stack: Java/Spring Boot, React/TypeScript, PostgreSQL. |
| 2026-08-17 | Adicionado padrão `specs/<HU>/` (spec.md, plan.md, task.md) na estrutura e no fluxo de trabalho. |

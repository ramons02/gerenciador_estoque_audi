# Gerenciador de Estoque — Documento de Requisitos

**Sistema:** Gerenciamento de estoque e vendas de revenda de Gás (GLP) e Água (vasilhames retornáveis)
**Versão:** 1.0
**Data:** 17/08/2026
**Status:** Proposta

---

## 1. Visão Geral

Sistema de controle de estoque, vendas e fechamento de caixa para revendedor de Gás (P13, P45) e Água (galões 20L), com o modelo de negócio de **casco retornável**: o cliente devolve 1 vasilhame vazio e leva 1 cheio.

O sistema opera com **3 estados de estoque por produto**:

| Estado | Significado |
|---|---|
| **Cheios** | Prontos para venda (no estoque) |
| **Vazios** | No pátio, aguardando recarga/devolução |
| **Em rua** | Com clientes em haver/comodato |

---

## 2. Requisitos Funcionais (RF)

### 2.1 Cadastro Base (Produtos)

| ID | Requisito |
|---|---|
| **RF-001** | O sistema deve permitir cadastrar produtos com a composição **Carga/Conteúdo** (ex.: Gás, Água) e **Vasilhame/Casco** (ex.: P13, P45, Galão 20L), combinando ambos na definição do item vendido (ex.: "Gás P13" = carga Gás + casco P13). |
| **RF-002** | O sistema deve permitir cadastrar, por item, o **preço de custo** e o **preço de venda**. |
| **RF-003** | O sistema deve permitir cadastrar o **limite mínimo de estoque** (alerta) por produto cheio. |
| **RF-004** | O sistema deve permitir o cadastro de **clientes** com dados de identificação e endereço (necessário para venda de vasilhame novo e controle de vasilhames em rua). |
| **RF-005** | O sistema deve permitir o cadastro de **fornecedores/distribuidoras** (ex.: Ultragaz, Nacional). |

### 2.2 Gestão de Entradas (Carregamento / Compra)

| ID | Requisito |
|---|---|
| **RF-010** | O sistema deve registrar a **chegada de caminhão**, com: fornecedor, produto, data, **quantidade de cascos cheios recebidos**, **quantidade de cascos vazios devolvidos** à distribuidora, **custo total da carga** e **valor unitário**. |
| **RF-011** | Ao confirmar a entrada, o sistema deve **incrementar automaticamente** o estoque de cheios e **ajustar o saldo de vazios** do pátio (entrada de cheios; saída de vazios devolvidos). |
| **RF-012** | O sistema deve recalcular o **custo médio** do produto após cada carregamento (para apuração de custo e margem). |

### 2.3 Gestão de Saídas (Vendas / Entregas)

| ID | Requisito |
|---|---|
| **RF-020** | O sistema deve permitir o **lançamento rápido de venda** com: produto, quantidade, data/hora e tipo de venda (**Balcão/Portaria** ou **Entrega**). |
| **RF-021** | O sistema deve registrar a **forma de pagamento**: **Dinheiro**, **PIX** ou **Cartão (Crédito/Débito como forma única)**. As formas aceitas são configuráveis (RF-052) e vendas não possuem mais a forma **Fiado**. |
| **RF-021-A** | O sistema deve aplicar um **acréscimo fixo (R$ por unidade)** nas vendas pagas com **Cartão**, com valor configurável na tela de Configurações (RF-052). O acréscimo se aplica **somente aos produtos de carga Gás**; produtos de carga Água usam o **preço normal** em qualquer forma de pagamento (Dinheiro, PIX ou Cartão). Dinheiro e PIX usam o preço normal em todos os produtos. |
| **RF-022** | Na venda por **Entrega**, o sistema deve **acrescentar automaticamente a taxa de entrega** ao total, com valor configurável por entrega. |
| **RF-023** | O sistema deve tratar a **troca de vasilhame** na venda: **Venda Normal** — cliente entrega 1 vazio e leva 1 cheio (o vazio recebido entra no pátio). |
| **RF-024** | O sistema deve tratar a **venda de vasilhame novo** — cliente compra o casco novo + a carga (sem devolução de vazio). |
| **RF-025** | Ao confirmar a venda, o sistema deve **baixar automaticamente** o item cheio do estoque e **adicionar 1 casco vazio ao pátio** (na venda com troca). |
| **RF-026** | Na venda sem devolução de vazio (vasilhame novo ou venda avulsa), o sistema deve **baixar o casco** do estoque de cheios e **registrar o vasilhame como "em rua"** (comodato do cliente). |
| **RF-027** | O sistema deve permitir o **recebimento de vasilhames vazios avulsos** (devoluções fora de venda), incrementando o pátio de vazios. |
| **RF-028** | O sistema deve controlar a **saída de vasilhames "em rua"** por cliente, com baixa quando o vazio é devolvido. |

### 2.4 Controle de Estoque (Pátio) — Tempo Real

| ID | Requisito |
|---|---|
| **RF-030** | O sistema deve exibir, em **tempo real**, por produto: quantidade de **Cheios**, quantidade de **Vazios** (pátio) e quantidade **Em rua** (clientes). |
| **RF-031** | O sistema deve **bloquear a venda** quando o estoque de cheios for insuficiente para a quantidade solicitada. |
| **RF-032** | O sistema deve gerar **alerta visual de estoque baixo** quando o saldo de cheios atingir ou ficar abaixo do limite mínimo configurado (RF-003). |

### 2.5 Relatórios e Exportação (Excel / CSV)

| ID | Requisito |
|---|---|
| **RF-040** | O sistema deve consolidar as movimentações e gerar planilhas (Excel/CSV) por período: **Hoje, Últimos 7 dias, Mês Atual ou período personalizado**. |
| **RF-041** | **Planilha 1 — Relatório de Vendas** (diário/mensal): Data/Hora, Produto, Qtd, Valor Unitário, Total (R$), Forma de Pagamento, Tipo (Balcão/Entrega). |
| **RF-042** | **Planilha 2 — Relatório de Carregamentos (Entradas)**: Data, Fornecedor, Produto, Qtd Cheios Entraram, Qtd Vazios Saíram, Custo Total. |
| **RF-043** | **Planilha 3 — Fechamento de Caixa e Balanço de Estoque**: Produto, Estoque Inicial, (+) Entradas, (−) Vendas, Estoque Final, Vazios em Pátio. |
| **RF-044** | O sistema deve disponibilizar **botão de exportação** no painel para cada relatório, no período selecionado. |

### 2.6 Dashboard (Resumo do Dia)

| ID | Requisito |
|---|---|
| **RF-050** | O sistema deve exibir no painel o **Total Faturado no Dia (R$)**. |
| **RF-051** | O sistema deve exibir o **total por forma de pagamento** no dia (**Dinheiro, PIX e Cartão** — crédito e débito somados em um único valor). |
| **RF-052** | O sistema deve permitir na tela de **Configurações**: (a) definir as **formas de pagamento aceitas** (Dinheiro, PIX, Cartão); (b) definir o **acréscimo do cartão** em R$ por unidade (aplicado aos produtos de carga Gás, conforme RF-021-A); (c) definir a **taxa de entrega**. |
| **RF-052** | O sistema deve exibir o **total de botijões/galões vendidos** no dia (por produto e total geral). |
| **RF-053** | O sistema deve exibir no dashboard os **alertas de estoque baixo** ativos (RF-032). |

---

## 3. Requisitos Não Funcionais (RNF)

| ID | Requisito |
|---|---|
| **RNF-001** | **Usabilidade:** O lançamento de venda deve exigir o mínimo de passos possível (lançamento rápido), executável em poucos segundos na portaria. |
| **RNF-002** | **Usabilidade:** Interface simples, legível em tela de computador e tablet, com números grandes nos campos de venda. |
| **RNF-003** | **Performance:** O lançamento de venda deve responder em até 2 segundos, mesmo com o histórico do dia aberto. |
| **RNF-004** | **Disponibilidade:** O sistema deve funcionar offline tolerando quedas curtas de internet (venda na portaria não pode parar) e sincronizar quando reconectar. |
| **RNF-005** | **Confiabilidade:** As atualizações de estoque (RF-011, RF-025) devem ser **atômicas** — nunca permitir estado parcial (ex.: baixar cheio sem adicionar vazio). |
| **RNF-006** | **Segurança:** Controle de acesso por usuário com papéis (ex.: vendedor/caixa e administrador); vendas registradas com o usuário responsável. |
| **RNF-007** | **Integridade:** Nenhuma venda ou entrada pode ser apagada — apenas cancelada/estornada, com rastro (log de auditoria de data, hora e usuário). |
| **RNF-008** | **Concorrência:** Vendas simultâneas não podem gerar estoque negativo (bloqueio/transação por produto). |
| **RNF-009** | **Exportação:** Relatórios Excel/CSV compatíveis com Excel, LibreOffice e Google Sheets (UTF-8, cabeçalhos claros). |
| **RNF-010** | **Retenção de dados:** Histórico de movimentações armazenado por pelo menos 12 meses, com backup diário automático. |
| **RNF-011** | **Portabilidade:** Acesso via navegador (web) e, idealmente, aplicativo mobile para entregador registrar entrega na rua. |

---

## 4. Regras de Domínio (RDN)

*Invariantes do mundo real do negócio — não dependem de decisão gerencial.*

| ID | Regra |
|---|---|
| **RDN-001** | Todo item vendido é composto por **Carga/Conteúdo + Vasilhame/Casco** (ex.: Gás P13). |
| **RDN-002** | O estoque de um produto é a soma dos 3 estados: **Cheios = Vazios(pátio) + Em rua + disponível para venda** (todo vasilhame existente está em exatamente um estado). |
| **RDN-003** | O **saldo de vazios é contabilmente finito**: só é possível devolver à distribuidora a quantidade de vazios existente no pátio. |
| **RDN-004** | Uma venda com troca altera **dois estoques simultaneamente**: −1 cheio e +1 vazio no pátio. |
| **RDN-005** | Estoque nunca pode ficar **negativo** em nenhum estado. |
| **RDN-006** | O vasilhame tem vida de uso: um casco só pode ser revendido cheio se estiver em condições de recarga (validade/trava de vasilhame impróprio). |
| **RDN-007** | O produto **Água 20L** e o **Gás P13/P45** possuem cargas de capacidades fixas distintas — a recarga é feita pela distribuidora, não pelo revendedor. |
| **RDN-008** | "Em rua" (comodato) só aumenta quando o cliente **leva um vasilhame sem devolver** um vazio equivalente (venda de vasilhame novo ou avulsa). |
| **RDN-009** | Um carregamento de caminhão envolve **dois fluxos opostos**: entram cheios (compra) e saem vazios (devolução). |

---

## 5. Regras de Negócio (RGN)

*Políticas operacionais e comerciais do estabelecimento.*

| ID | Regra |
|---|---|
| **RGN-001** | Venda do tipo **Entrega** deve adicionar **taxa de entrega** ao total; o valor da taxa é configurável pelo administrador. |
| **RGN-002** | A forma de pagamento **Fiado não existe no sistema**. As formas aceitas são configuráveis (RF-052); **Cartão** sempre aplica o acréscimo configurado (RF-021-A) **aos produtos de carga Gás**; produtos de carga Água seguem o preço normal em qualquer forma de pagamento. |
| **RGN-003** | O vendedor pode aplicar **desconto** na venda apenas até um limite percentual configurado; acima disso exige aprovação do administrador. |
| **RGN-004** | Ao atingir o **limite mínimo** de cheios (RF-003), o sistema emite alerta visual persistente até a nova entrada de caminhão. |
| **RGN-005** | O **preço de venda** é validado para nunca ser inferior ao **preço de custo** no cadastro de produto (exceto se o administrador autorizar). |
| **RGN-006** | O fechamento de caixa do dia só pode ser concluído se **todas as vendas do dia** estiverem conciliadas (sem vendas em aberto/em edição). |
| **RGN-007** | O cancelamento/estorno de venda **reverte o estoque** e o caixa automaticamente, mantendo o registro histórico com status "cancelado". |
| **RGN-008** | O relatório de vendas deve sempre discriminar **forma de pagamento** e **tipo (Balcão/Entrega)** para conferência do caixa físico. |
| **RGN-009** | O alerta de estoque baixo deve considerar a **média de vendas diárias** do produto como sugestão de reposição (quantidade a comprar no próximo carregamento). |
| **RGN-010** | Venda de **vasilhame novo** (casco + carga) tem preço diferente da venda com troca; o preço do casco é configurado à parte. |

---

## 6. Rastreabilidade (Resumo)

```
Entrada (caminhão)        → RF-010/011 → Cheios (+) e Vazios (−)      → Planilha 2 (RF-042)
Venda balcão/entrega      → RF-020/025 → Cheios (−) e Vazios (+)      → Planilha 1 (RF-041)
Vasilhame novo / em rua   → RF-024/026 → Cheios (−) e Em rua (+)      → Balanço (RF-043)
Fechamento do dia         → RF-050/053 → Dashboard + Planilha 3
```

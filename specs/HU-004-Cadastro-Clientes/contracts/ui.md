# Contrato de UI: Cadastro de Clientes

**HU**: HU-004 - Cadastro de Clientes
**Branch**: `HU-004-Cadastro-Clientes`

Telas: listagem/formulário de clientes (`src/features/clientes/`) e busca no lançamento de
venda (`src/features/vendas/`, HU-007, que consome o mesmo serviço). Consumo da API por
cliente HTTP tipado (`src/api/clientes.ts`). Vocabulário canônico (CONSTITUICAO §II):
"em rua" para vasilhame com cliente.

## 1. Listagem de clientes

| Elemento | Descrição |
|---|---|
| Título | "Clientes" |
| Campo de busca | Filtra por nome ou telefone (CT-002) |
| Colunas | Nome, Telefone, Endereço |
| Ações | Botão "Novo cliente" e botão "Editar" por linha |

**Comportamento:** carrega `GET /api/clientes` e, ao digitar na busca, chama
`GET /api/clientes?q=<termo>`.

## 2. Formulário de cliente

| Campo | Tipo | Obrigatório | Observação |
|---|---|---|---|
| Nome | Texto | sim | Único campo obrigatório |
| Telefone | Texto (máscara de telefone) | não | Usado na busca |
| Endereço | Texto | não | - |

**Ações:** "Salvar" (POST ou PUT) e "Cancelar".

**Validações inline:**
- Nome vazio: "Informe o nome do cliente." (bloqueia o salvar).
- Validações da API exibidas sem colapsar em erro genérico (CONSTITUICAO §XI.3).

**Nota:** o campo "documento" previsto na spec não aparece na tela (removido do schema por
LGPD, migration V6).

## 3. Busca de cliente no lançamento de venda (CT-002)

| Elemento | Descrição |
|---|---|
| Onde | Tela de lançamento de venda (HU-007), campo de busca de cliente |
| Comportamento | Ao digitar nome ou telefone, o sistema retorna os clientes correspondentes na mesma tela, sem navegação (RNF-001) |
| Seleção | Cliente encontrado é associado à venda em andamento |

**Regras de venda (CT-003, aplicadas pela HU-007):**
- Na venda de vasilhame novo, o cliente é obrigatório: sem cliente selecionado, a confirmação
  é bloqueada com a mensagem de cliente obrigatório.
- As formas de pagamento exibidas não incluem Fiado (RGN-002).

## 4. Exclusão de cliente

| Elemento | Descrição |
|---|---|
| Ação | Botão "Excluir" por linha, com confirmação |
| Comportamento | `DELETE /api/clientes/{id}` (exclusão lógica, 204) |
| Consequência | Cliente some da listagem e da busca, mas vendas e vasilhames "em rua" permanecem preservados |

## 5. Mensagens

| Situação | Mensagem |
|---|---|
| Sucesso ao salvar | "Cliente salvo com sucesso." |
| Sucesso ao excluir | "Cliente excluído." |
| Erro de validação | "Informe o nome do cliente." ou mensagem 422 da API |
| Falha de rede/API | Mensagem da API quando estruturada; caso contrário mensagem genérica em pt-BR |

## 6. Rastreabilidade

- CT-001 (cadastro com identificação e endereço), CT-002 (busca por nome/telefone), CT-003
  (cliente obrigatório em vasilhame novo e sem Fiado), RF-004, RF-028, RGN-002.
- Commits referenciam HU-004 (CONVENTIONS §9).
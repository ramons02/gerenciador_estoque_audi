# Premium Gás - Documentação e Auditoria

Repositório de documentação do sistema **Premium Gás** (revenda de Gás GLP e Água
com vasilhame retornável). Contém requisitos, especificações, convenções e evidências
de auditoria.

## Estrutura

| Pasta/Arquivo | Conteúdo |
|---|---|
| `CONSTITUICAO.md` | Princípios não negociáveis do projeto (governança) |
| `CONVENTIONS.md` | Convenções de código e de implementação |
| `HUs/` | Histórias de usuário (`HU-001` a `HU-010`) com critérios de aceite |
| `requisitos/` | Levantamento de requisitos do negócio |
| `specs/` | Especificações detalhadas das funcionalidades |
| `aplicacao/RODAR-LOCAL-WINDOWS.md` | Guia para rodar o sistema localmente no Windows |

## HUs

- `HU-001` Cadastro de produtos (carga + vasilhame)
- `HU-002` Cadastro de preços
- `HU-003` Limite de estoque baixo (alerta + sugestão de reposição)
- `HU-004` Cadastro de clientes (inclui comodato de vasilhames)
- `HU-005` Cadastro de fornecedores
- `HU-006` Chegada de caminhão (entrada de cheios)
- `HU-007` Lançamento de venda
- `HU-008` Venda com entrega
- `HU-009` Troca de vasilhame
- `HU-010` Venda de vasilhame novo

## Repositórios irmãos

- `gerenciador_estoque_api` - backend (Spring Boot)
- `gerenciador_estoque_app` - frontend (React + Vite)
- `gerenciador_estoque_jar` - JAR executável para apresentação
- `gerenciador_estoque_infra` - infraestrutura, variáveis de ambiente e deploy
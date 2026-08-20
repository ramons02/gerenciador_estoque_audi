# Implementation Plan: Cadastro de Clientes

**Branch**: `HU-004-Cadastro-Clientes` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/HU-004-Cadastro-Clientes/spec.md`

## Summary

Permitir ao vendedor cadastrar clientes com nome, telefone, endereço e (opcionalmente)
documento, com busca por nome ou telefone no momento da venda e obrigatoriedade do cliente na
venda de vasilhame novo (RF-004, RF-028, RGN-002, CT-001 a CT-003). Abordagem: tabela
`tab_cliente` (migration V3), CRUD REST em `/api/clientes` com busca por parâmetro `q`,
exclusão lógica (`ativo = false`), e a regra de cliente obrigatório aplicada no fluxo de
venda (HU-007), que também vincula o vasilhame "em rua" ao cliente via `tab_cliente_vasilhame`
(RF-028).

## Technical Context

**Language/Version**: Java 21 + Spring Boot 3.x (backend); React 18 + TypeScript 5.x (frontend)
**Primary Dependencies**: Spring Data JPA, Spring Validation, Flyway, Lombok (backend); React, Vite, axios, react-router (frontend)
**Storage**: PostgreSQL (único banco) + migrações Flyway forward-only
**Testing**: JUnit 5 + Mockito (Service/Controller), integração com @SpringBootTest; Vitest + React Testing Library (app)
**Target Platform**: navegador web (SPA) + API REST
**Project Type**: full-stack web (REST API + SPA)
**Performance Goals**: busca de cliente por nome/telefone e cadastro em menos de 2 s, apoiando o lançamento rápido de venda (RNF-001, RNF-003)
**Constraints**: cliente obrigatório na venda de vasilhame novo (RF-024/RF-026), forma Fiado inexistente (RGN-002), exclusão lógica com histórico de em rua preservado (RNF-007), erro JSON com mensagem pt-BR
**Scale/Scope**: revenda única, 1 a 5 usuários simultâneos, histórico de 12 meses (RNF-010)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate | Verificação | Resultado |
|---|---|---|
| §I-A Artefatos por HU | spec.md, plan.md, research.md, data-model.md, contracts/ e quickstart.md existem antes de qualquer código | OK |
| §II Vocabulário | Documentação usa Carga, Vasilhame, Cheio, Vazio, Em rua, Troca, Carregamento e Pátio | OK |
| §III Invariantes de estoque | Feature não altera estoque; vincula "em rua" ao cliente sem mexer em saldo | OK |
| §V Critério fechado = CT provado | Cada CT (CT-001 a CT-003) só fecha com prova registrada no tasks.md | OK |
| §VI Proibido | Sem DELETE físico; sem Fiado (RGN-002); sem regra fora dos requisitos; sem termo não canônico | OK |
| §VII Documentação | pt-BR, hífen normal, sem travessão | OK |
| §X Rastreabilidade | RF-004, RF-028, RGN-002 e HU-004 referenciados em commits e doc do módulo | OK |
| §XI Qualidade | Validação de nome obrigatório no Service (lógica pura testada); erro estruturado | OK |

**GATE: PASS**

## Project Structure

### Documentation (this feature)

```text
specs/HU-004-Cadastro-Clientes/
├── plan.md              # Este arquivo
├── research.md          # Fase 0: decisões técnicas
├── data-model.md        # Fase 1: modelo de dados
├── quickstart.md        # Fase 1: validação end-to-end
├── contracts/
│   ├── api.md           # Contrato da API REST
│   └── ui.md            # Contrato de UI
└── tasks.md             # Fase 2: checklist de CTs
```

### Source Code (repository root)

```text
gerenciador_estoque_api/
├── pom.xml
└── src/
    ├── main/java/com/gerenciador/estoque/
    │   ├── core/
    │   │   ├── exception/         # BusinessException, NotFoundException, GlobalExceptionHandler
    │   │   ├── model/             # BaseModel
    │   │   ├── repository/        # BaseRepository
    │   │   └── service/           # BaseService
    │   ├── cliente/
    │   │   ├── ClienteController.java    # GET /api/clientes?q=, POST, PUT, DELETE
    │   │   ├── ClienteService.java       # buscar(nome/telefone), validar nome
    │   │   ├── ClienteRepository.java    # buscarPorNomeOuTelefone
    │   │   └── Cliente.java
    │   └── vasilhame/
    │       ├── ClienteVasilhame.java          # tab_cliente_vasilhame (em rua por cliente)
    │       ├── ClienteVasilhameService.java   # adicionar/baixar em rua (RF-028)
    │       └── ClienteVasilhameRepository.java
    ├── main/resources/db/migration/     # V3__estoque_clientes_carregamentos_vendas.sql (tab_cliente)
    └── test/java/com/gerenciador/estoque/
        └── business/cliente/ClienteServiceTest.java   # JUnit 5 + Mockito

gerenciador_estoque_app/
└── src/
    ├── api/
    │   ├── client.ts            # Cliente HTTP tipado (axios)
    │   ├── types.ts             # Cliente, ClienteInput
    │   └── clientes.ts          # listarClientes(termo), criarCliente, atualizarCliente
    └── features/
        ├── clientes/           # Listagem e formulário de clientes (HU-004)
        └── vendas/             # Busca de cliente no lançamento de venda (HU-007, consume daqui)
```

**Structure Decision**: monorepo com repositórios irmãos conforme Constituição §I. A feature
vive no módulo `com.gerenciador.estoque.cliente` (camadas Controller/Service/Repository) e se
apoia no módulo `vasilhame` para o controle de "em rua" por cliente (`tab_cliente_vasilhame`,
RF-028). No app, a feature vive em `src/features/clientes/`; a busca por nome/telefone é
reutilizada pelo lançamento de venda (`src/features/vendas/`) sem sair da tela de venda
(RNF-001).

## Complexity Tracking

> Nenhuma violação. A busca e o cadastro usam tabela única e consulta simples no
> Repository, sem necessidade de índice dedicado ou cache nesta escala (RNF-010).
# Quickstart: HU-006 - Registro de Chegada de Caminhão (Entrada)

**HU**: HU-006 | **Feature**: Chegada de Caminhão (Carregamento) | **Date**: 2026-08-20 | **Spec**: [spec.md](spec.md)
**Referências**: [api.md](contracts/api.md) | [ui.md](contracts/ui.md) | [data-model.md](data-model.md)

Guia de validação end-to-end dos CTs da HU-006. Não substitui os contratos: consultar os links acima para campos e mensagens exatas.

---

## Pré-requisitos

- PostgreSQL rodando localmente (config em `gerenciador_estoque_infra/env`); migrations Flyway aplicadas automaticamente na subida da API (V1 a V9 existentes).
- API: `cd gerenciador_estoque_api && mvn spring-boot:run` (porta 8080).
- App: `cd gerenciador_estoque_app && npm install && npm run dev` (porta 5173).
- Massa mínima: 1 fornecedor cadastrado (HU-005) e 1 produto ativo com preços (HU-001/HU-002), e saldo de vazios no pátio conhecido (conferir em `GET /api/estoque` ou painel de estoque).

---

## Cenários de prova (roteirizados)

### CT-001 - Captura e valor unitário calculado

Dado um fornecedor e um produto cadastrados, quando o administrador abre a tela de Carregamentos e preenche fornecedor, produto, 100 cheios, 20 vazios devolvidos e custo total R$ 3.000,00, então o campo "Valor unitário" exibe R$ 30,00 (custo total ÷ cheios) antes de confirmar, e o request não aceita valor unitário manual (Service calcula). Contrato: [api.md](contracts/api.md) POST /api/carregamentos.

### CT-002 - Devolução de vazios limitada ao saldo do pátio

Dado um saldo de 10 vazios no pátio (conferido em `GET /api/estoque`), quando o administrador tenta registrar 15 vazios devolvidos, então o sistema bloqueia com mensagem "Devolução de 15 vazios excede o saldo do pátio. Disponível: 10." (status 409). Quando registra exatamente 10 vazios, a confirmação é permitida (limite exato). Contrato: [api.md](contracts/api.md) POST /api/carregamentos; regra em [data-model.md](data-model.md).

### CT-003 - Estoque refletido automaticamente

Dado um registro preenchido e válido, quando o administrador confirma, então `GET /api/estoque` passa a mostrar cheios +N e vazios -N imediatamente, e a lista de carregamentos exibe o registro com data, fornecedor, produto, cheios, vazios e custo total. Verificação no banco (Constituição §IV.1): `SELECT id_produto, tipo, quantidade, id_referencia FROM tab_movimentacao_estoque WHERE id_referencia = <id do carregamento>;` deve conter ENTRADA_CHEIO com +N e SAIDA_VAZIO com -M na mesma transação (sem estado parcial).

### CT-004 - Custo médio recalculado

Dado um produto com custo médio anterior, quando uma chegada com novo custo é confirmada, então o custo médio do produto é atualizado no mesmo commit da entrada. Verificação: `SELECT custo_medio FROM tab_produto WHERE id = <produto>;` antes e depois da confirmação.

### CT-005 - Registro no Relatório de Carregamentos

Dado um carregamento confirmado, quando o Relatório de Carregamentos é gerado no período correspondente (HU-017), então o registro aparece com Data, Fornecedor, Produto, Qtd Cheios Entraram, Qtd Vazios Saíram e Custo Total (RF-042, CONVENTIONS §10).

---

## Cenário de resiliência (edge cases)

- Valor unitário com custo não divisível exato: custo total R$ 1.000,00 com 3 cheios → R$ 333,33 (arredondado, 2 casas).
- Chegada com 0 cheios: bloqueada com "A quantidade de cheios deve ser maior que zero." (evita divisão por zero).
- Concorrência (RNF-008): duas confirmações simultâneas de devoluções que somam mais que o saldo do pátio → exatamente uma deve falhar com 409, nunca saldo negativo. Verificação no banco: `SELECT qtd_vazios FROM tab_estoque WHERE id_produto = <p>;` >= 0.
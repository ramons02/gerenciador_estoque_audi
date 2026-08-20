# Feature Specification: Registro de Chegada de Caminhão (Entrada)

**Feature Branch**: `HU-006-Chegada-Caminhao`

**Created**: 2026-08-19

**Status**: Draft

**Input**: User description: "Como administrador, quero registrar a chegada de caminhão com cheios recebidos, vazios devolvidos e custo total, para que o estoque e o caixa reflitam a compra da carga."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Registrar a chegada de caminhão (Priority: P1)
O administrador informa fornecedor, produto, data, a quantidade de cheios recebidos, a quantidade de vazios devolvidos à distribuidora e o custo total da carga. O sistema calcula automaticamente o valor unitário (custo total ÷ cheios). Ao confirmar o registro, o estoque de cheios aumenta em N e o saldo de vazios do pátio diminui em N, refletindo a compra da carga.

**Why this priority**: É a operação que alimenta o estoque de cheios e o caixa da revenda — sem ela nenhuma venda subsequente tem estoque para vender, sendo o fluxo mais crítico do lado de entradas.

**Independent Test**: Criar um registro de chegada com fornecedor, produto e quantidades válidas, confirmar e conferir no estoque que cheios aumentaram em N e vazios diminuíram em N.

**Acceptance Scenarios**:

1. **Given** um fornecedor e um produto cadastrados, **When** o administrador registra uma chegada com 100 cheios, 20 vazios devolvidos e custo total de R$ 3.000,00, **Then** o sistema exibe o valor unitário calculado de R$ 30,00 (custo total ÷ cheios).
2. **Given** um registro de chegada preenchido e válido, **When** o administrador confirma o registro, **Then** o estoque de cheios do produto aumenta em N e o saldo de vazios do pátio diminui em N automaticamente.
3. **Given** um registro de chegada confirmado, **When** o administrador consulta o Relatório de Carregamentos, **Then** o registro aparece com data, fornecedor, produto, cheios, vazios e custo total.

---

### User Story 2 - Validar a devolução de vazios contra o saldo do pátio (Priority: P2)
Ao registrar a chegada, o sistema compara a quantidade de vazios devolvidos à distribuidora com o saldo de vazios existente no pátio e bloqueia a confirmação quando a devolução exceder o saldo disponível.

**Why this priority**: Protege o invariante contábil do pátio (RDN-003) — não é possível devolver mais vazios do que existe — evitando saldo negativo e inconsistência com a distribuidora.

**Independent Test**: Informar uma devolução de vazios maior que o saldo atual do pátio e verificar que o sistema bloqueia a confirmação com mensagem de erro.

**Acceptance Scenarios**:

1. **Given** um saldo de 10 vazios no pátio, **When** o administrador tenta registrar uma chegada com 15 vazios devolvidos, **Then** o sistema bloqueia a confirmação indicando que a devolução excede o saldo.
2. **Given** um saldo de 10 vazios no pátio, **When** o administrador registra uma chegada com exatamente 10 vazios devolvidos, **Then** o sistema permite a confirmação.

---

### User Story 3 - Recalcular o custo médio e exibir no relatório (Priority: P3)
Após cada chegada confirmada, o sistema recalcula o custo médio do produto para apuração de custo e margem, e mantém o registro visível no Relatório de Carregamentos.

**Why this priority**: É um requisito de apuração contábil e visibilidade que não bloqueia a operação do dia, mas é necessário para o fechamento e conferência do caixa.

**Independent Test**: Confirmar uma chegada e verificar que o custo médio do produto foi recalculado e que o registro consta no Relatório de Carregamentos do período.

**Acceptance Scenarios**:

1. **Given** um produto com custo médio anterior, **When** uma chegada com novo custo é confirmada, **Then** o sistema recalcula e atualiza o custo médio do produto.
2. **Given** um registro de chegada confirmado, **When** o Relatório de Carregamentos é gerado no período correspondente, **Then** o registro aparece no relatório.

### Edge Cases

- Custo total não divisível exato pela quantidade de cheios — o valor unitário deve ser arredondado com 2 casas decimais.
- Chegada registrada com zero cheios recebidos — a divisão por zero no cálculo do valor unitário deve ser tratada.
- Devolução de vazios igual ao saldo do pátio (limite exato) — deve ser permitida.
- Registro confirmado por engano — não pode ser apagado, apenas estornado/cancelado com rastro de auditoria.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema deve capturar no registro de chegada: fornecedor, produto, data, quantidade de cheios recebidos, quantidade de vazios devolvidos e custo total da carga.
- **FR-002**: O sistema deve calcular automaticamente o valor unitário como custo total ÷ quantidade de cheios.
- **FR-003**: O sistema deve bloquear a confirmação quando a quantidade de vazios devolvidos exceder o saldo de vazios do pátio.
- **FR-004**: Ao confirmar a chegada, o sistema deve incrementar o estoque de cheios e decrementar o saldo de vazios do pátio na quantidade informada.
- **FR-005**: O sistema deve recalcular o custo médio do produto após cada entrada confirmada.
- **FR-006**: O sistema deve exibir o registro de chegada no Relatório de Carregamentos.

### Key Entities

- **Entrada de Caminhão (Carregamento)**: registro da chegada com fornecedor, produto, data, cheios recebidos, vazios devolvidos, custo total e valor unitário calculado.
- **Produto**: item composto por carga + casco, com estoque de cheios, saldo de vazios no pátio e custo médio atualizado a cada entrada.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% das chegadas confirmadas refletem imediatamente no estoque (cheios +N e vazios −N).
- **SC-002**: Zero confirmações com devolução de vazios acima do saldo do pátio.
- **SC-003**: 100% dos registros de chegada confirmados aparecem no Relatório de Carregamentos do período.
- **SC-004**: O valor unitário exibido é sempre igual ao custo total ÷ quantidade de cheios.

## Assumptions

- O saldo de vazios do pátio é mantido e consultável pelo sistema (RDN-003).
- Fornecedores e produtos já estão cadastrados (RF-001, RF-002, RF-005).
- O custo médio é a base de apuração de custo e margem da revenda (RF-012).
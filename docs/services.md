# Serviços

A Credit Journey Platform é composta por microservices independentes, cada um com responsabilidade clara dentro da jornada de crédito.

## Visão geral

| Serviço | Responsabilidade |
|---|---|
| credit-customer-service | Gerencia cadastro, status e perfil do cliente |
| credit-rules-engine-service | Avalia elegibilidade e políticas de crédito |
| credit-limit-service | Calcula e mantém limites de crédito |
| credit-communication-service | Dispara comunicações para o cliente |
| credit-audit-service | Registra histórico e timeline da jornada |
| credit-config-server | Centraliza configurações dos serviços |
| credit-platform-infra | Centraliza infraestrutura local e Kubernetes |
| credit-shared-contracts | Centraliza contratos OpenAPI, AsyncAPI e schemas |

## credit-customer-service

Responsável pelo domínio de clientes.

### Responsabilidades

- Criar cliente
- Atualizar dados cadastrais
- Consultar cliente
- Alterar status do cliente
- Registrar histórico de status
- Publicar eventos de cliente

### Endpoints planejados

| Método | Endpoint | Descrição |
|---|---|---|
| POST | /api/v1/customers | Cria cliente |
| GET | /api/v1/customers/{id} | Consulta cliente por ID |
| PUT | /api/v1/customers/{id} | Atualiza cliente |
| PATCH | /api/v1/customers/{id}/status | Atualiza status |

### Eventos publicados

- CustomerCreated
- CustomerUpdated
- CustomerStatusChanged

### Banco

- customer_db

### Tabelas principais

- customers
- customer_status_history
- outbox_events

## credit-rules-engine-service

Responsável pelas regras de elegibilidade e decisão de crédito.

### Responsabilidades

- Avaliar elegibilidade do cliente
- Aplicar políticas de risco
- Validar regras de negócio
- Consultar feature toggles
- Publicar decisão de aprovação ou rejeição

### Endpoints planejados

| Método | Endpoint | Descrição |
|---|---|---|
| POST | /api/v1/rules/evaluate | Avalia elegibilidade |
| GET | /api/v1/rules/customers/{customerId}/eligibility | Consulta elegibilidade |

### Eventos consumidos

- CustomerCreated
- CustomerStatusChanged

### Eventos publicados

- CustomerEligibilityApproved
- CustomerEligibilityRejected
- CreditPolicyEvaluated

### Banco

- rules_db

### Tabelas principais

- credit_policies
- eligibility_decisions
- rule_evaluation_history

## credit-limit-service

Responsável pelo cálculo e manutenção de limite de crédito.

### Responsabilidades

- Calcular limite inicial
- Consultar limite disponível
- Atualizar limite
- Bloquear limite
- Liberar limite
- Registrar histórico de alterações
- Publicar eventos de limite

### Endpoints planejados

| Método | Endpoint | Descrição |
|---|---|---|
| GET | /api/v1/limits/customers/{customerId} | Consulta limite do cliente |
| POST | /api/v1/limits/calculate | Calcula limite |
| PATCH | /api/v1/limits/{id} | Atualiza limite |
| PATCH | /api/v1/limits/{id}/block | Bloqueia limite |
| PATCH | /api/v1/limits/{id}/release | Libera limite |

### Eventos consumidos

- CustomerEligibilityApproved

### Eventos publicados

- LimitCalculated
- LimitUpdated
- LimitBlocked
- LimitReleased

### Banco

- limit_db

### Tabelas principais

- credit_limits
- limit_change_history
- outbox_events
- inbox_events

## credit-communication-service

Responsável pelo envio de comunicações.

### Responsabilidades

- Receber solicitação de comunicação
- Escolher canal
- Montar template
- Enviar comunicação fake
- Controlar retry
- Enviar falhas para DLQ
- Registrar histórico de envio

### Endpoints planejados

| Método | Endpoint | Descrição |
|---|---|---|
| POST | /api/v1/communications | Solicita comunicação |
| GET | /api/v1/communications/{id} | Consulta comunicação |
| GET | /api/v1/communications/customers/{customerId} | Lista comunicações do cliente |

### Eventos consumidos

- LimitCalculated
- LimitUpdated
- CustomerEligibilityRejected

### Eventos publicados

- CommunicationRequested
- CommunicationSent
- CommunicationFailed
- CommunicationDiscarded

### Banco

- communication_db

### Tabelas principais

- communication_templates
- communication_requests
- communication_delivery_history

### Filas RabbitMQ

- communication.send.queue
- communication.send.retry.queue
- communication.send.dlq

## credit-audit-service

Responsável pela trilha histórica da jornada.

### Responsabilidades

- Consumir eventos dos demais serviços
- Registrar timeline do cliente
- Guardar payloads de eventos
- Permitir consulta por cliente, evento ou período
- Apoiar auditoria técnica e de negócio

### Endpoints planejados

| Método | Endpoint | Descrição |
|---|---|---|
| GET | /api/v1/audits/customers/{customerId}/timeline | Consulta timeline |
| GET | /api/v1/audits/events/{eventId} | Consulta evento |
| GET | /api/v1/audits | Consulta por filtros |

### Eventos consumidos

- CustomerCreated
- CustomerStatusChanged
- CustomerEligibilityApproved
- CustomerEligibilityRejected
- LimitCalculated
- LimitUpdated
- CommunicationSent
- CommunicationFailed

### Banco

- audit_db

### Tabelas principais

- audit_events
- customer_timeline
- event_payloads

## credit-config-server

Responsável pela configuração centralizada.

### Responsabilidades

- Servir configurações por ambiente
- Centralizar properties dos serviços
- Controlar configurações externas
- Apoiar feature toggles simples

### Porta padrão

- 8888

## credit-platform-infra

Responsável pela infraestrutura local e Kubernetes.

### Responsabilidades

- Docker Compose
- MySQL
- Kafka
- RabbitMQ
- Prometheus
- Grafana
- OpenTelemetry Collector
- Kubernetes manifests

## credit-shared-contracts

Responsável pela documentação e versionamento de contratos.

### Responsabilidades

- OpenAPI dos endpoints REST
- AsyncAPI dos eventos
- Schemas JSON dos eventos
- Versionamento de contratos
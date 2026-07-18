# Serviços

Este documento descreve os serviços da **Credit Journey Platform** e a responsabilidade de cada componente dentro da jornada de crédito.

A plataforma é composta por microservices independentes, serviços de apoio e bibliotecas compartilhadas.

---

## Visão geral

| Serviço | Tipo | Responsabilidade |
|---|---|---|
| `credit-api-gateway` | Microservice | Entrada única da plataforma, roteamento, JWT, CORS e correlationId |
| `credit-auth-service` | Microservice | Autenticação, login, refresh token, roles e permissões |
| `credit-customer-service` | Microservice | Cadastro, consulta, atualização, status e exclusão de clientes |
| `credit-limit-service` | Microservice | Criação, consulta, cálculo e manutenção de limites de crédito |
| `credit-rules-engine-service` | Microservice | Avaliação de elegibilidade e regras de crédito |
| `credit-orchestrator-service` | Microservice | Coordenação da jornada de crédito entre serviços |
| `credit-communication-service` | Microservice | Solicitação, envio, retry e DLQ de comunicações |
| `credit-audit-service` | Microservice | Auditoria, eventos e timeline da jornada |
| `credit-config-server` | Microservice de apoio | Configuração centralizada dos serviços |
| `credit-observability-starter` | Lib compartilhada | Logs automáticos, correlationId, MDC e padrões de observabilidade |
| `credit-platform-infra` | Repositório de infra | Docker Compose, Kubernetes e infraestrutura local |
| `credit-shared-contracts` | Repositório de contratos | OpenAPI, AsyncAPI, schemas e contratos compartilhados |

---

# Serviços principais

---

## credit-api-gateway

Responsável por ser a entrada única da plataforma.

### Responsabilidades

- Receber chamadas externas;
- Validar JWT;
- Aplicar CORS;
- Aplicar rate limit;
- Propagar `correlationId`;
- Propagar headers técnicos;
- Roteiar requisições para os serviços internos;
- Centralizar filtros técnicos de entrada.

### Rotas planejadas

| Rota externa | Serviço destino |
|---|---|
| `/api/v1/auth/**` | `credit-auth-service` |
| `/api/v1/customers/**` | `credit-customer-service` |
| `/api/v1/limits/**` | `credit-limit-service` |
| `/api/v1/rules/**` | `credit-rules-engine-service` |
| `/api/v1/credit-journeys/**` | `credit-orchestrator-service` |
| `/api/v1/communications/**` | `credit-communication-service` |
| `/api/v1/audits/**` | `credit-audit-service` |

### Observação

O gateway não deve conter regra de negócio.

Ele é responsável por entrada técnica, segurança, roteamento e filtros.

---

## credit-auth-service

Responsável por autenticação e autorização.

### Responsabilidades

- Registrar usuário;
- Realizar login;
- Validar senha;
- Gerar JWT;
- Gerar refresh token;
- Controlar roles;
- Controlar permissões;
- Invalidar ou revogar tokens quando necessário.

### Endpoints planejados

| Método | Endpoint | Descrição |
|---|---|---|
| POST | `/api/v1/auth/register` | Registra usuário |
| POST | `/api/v1/auth/login` | Realiza login |
| POST | `/api/v1/auth/refresh` | Atualiza token |
| POST | `/api/v1/auth/logout` | Encerra sessão |

### Banco

```text
auth_db
```

### Tabelas principais

```text
users
roles
user_roles
refresh_tokens
login_attempts
```

---

## credit-customer-service

Responsável pelo domínio de clientes.

### Responsabilidades

- Criar cliente;
- Consultar cliente por ID;
- Listar clientes com paginação;
- Filtrar clientes por status, busca, nome e renda;
- Atualizar dados cadastrais;
- Alterar status do cliente;
- Excluir cliente;
- Validar dados de domínio;
- Publicar eventos de cliente.

### Endpoints

| Método | Endpoint | Descrição |
|---|---|---|
| POST | `/api/v1/customers` | Cria cliente |
| GET | `/api/v1/customers/{customerId}` | Consulta cliente por ID |
| GET | `/api/v1/customers` | Lista clientes com paginação e filtros |
| PATCH | `/api/v1/customers/{customerId}` | Atualiza parcialmente o cliente |
| PUT | `/api/v1/customers/{customerId}/status` | Altera status do cliente |
| DELETE | `/api/v1/customers/{customerId}` | Exclui cliente |

### Eventos publicados

```text
CustomerCreated
CustomerUpdated
CustomerStatusChanged
CustomerDeleted
```

### Banco

```text
customer_db
```

### Tabelas principais

```text
customers
customer_status_history
outbox_events
```

---

## credit-limit-service

Responsável pelo cálculo e manutenção de limites de crédito.

### Responsabilidades

- Criar limite de crédito;
- Consultar limite por cliente;
- Calcular limite inicial;
- Atualizar limite;
- Bloquear limite;
- Liberar limite;
- Registrar histórico de alterações;
- Consumir eventos de elegibilidade;
- Publicar eventos de limite.

### Endpoints planejados

| Método | Endpoint | Descrição |
|---|---|---|
| POST | `/api/v1/limits` | Cria limite |
| GET | `/api/v1/limits/customers/{customerId}` | Consulta limite do cliente |
| POST | `/api/v1/limits/calculate` | Calcula limite inicial |
| PATCH | `/api/v1/limits/{limitId}` | Atualiza limite |
| PATCH | `/api/v1/limits/{limitId}/block` | Bloqueia limite |
| PATCH | `/api/v1/limits/{limitId}/release` | Libera limite |

### Eventos consumidos

```text
CustomerEligibilityApproved
```

### Eventos publicados

```text
LimitCalculated
LimitUpdated
LimitBlocked
LimitReleased
```

### Banco

```text
limit_db
```

### Tabelas principais

```text
credit_limits
limit_change_history
outbox_events
inbox_events
```

---

## credit-rules-engine-service

Responsável pelas regras de elegibilidade e decisão de crédito.

### Responsabilidades

- Avaliar elegibilidade do cliente;
- Aplicar políticas de risco;
- Validar status do cliente;
- Validar renda mínima;
- Aplicar regras configuráveis;
- Consultar feature toggles;
- Persistir decisão de elegibilidade;
- Publicar decisão de aprovação, rejeição ou análise manual.

### Endpoints planejados

| Método | Endpoint | Descrição |
|---|---|---|
| POST | `/api/v1/rules/evaluate` | Avalia elegibilidade |
| GET | `/api/v1/rules/customers/{customerId}/eligibility` | Consulta elegibilidade do cliente |

### Eventos consumidos

```text
CustomerCreated
CustomerUpdated
CustomerStatusChanged
```

### Eventos publicados

```text
CustomerEligibilityApproved
CustomerEligibilityRejected
CustomerManualAnalysisRequested
```

### Banco

```text
rules_db
```

### Tabelas principais

```text
credit_policies
eligibility_decisions
rule_evaluation_history
outbox_events
inbox_events
```

---

## credit-orchestrator-service

Responsável por coordenar a jornada de crédito entre os serviços.

### Responsabilidades

- Iniciar jornada de crédito;
- Consultar dados do cliente;
- Solicitar avaliação de elegibilidade;
- Solicitar cálculo de limite;
- Solicitar comunicação;
- Solicitar registro de auditoria;
- Controlar status da jornada;
- Registrar falhas por etapa;
- Propagar `correlationId`.

### Endpoints planejados

| Método | Endpoint | Descrição |
|---|---|---|
| POST | `/api/v1/credit-journeys` | Inicia jornada de crédito |
| GET | `/api/v1/credit-journeys/{journeyId}` | Consulta jornada |
| GET | `/api/v1/credit-journeys/customers/{customerId}` | Consulta jornadas do cliente |

### Serviços chamados

```text
credit-customer-service
credit-rules-engine-service
credit-limit-service
credit-communication-service
credit-audit-service
```

### Eventos publicados

```text
CreditJourneyStarted
CreditJourneyCompleted
CreditJourneyFailed
```

### Banco

```text
orchestrator_db
```

### Tabelas principais

```text
credit_journeys
credit_journey_steps
orchestrator_errors
outbox_events
```

---

## credit-communication-service

Responsável pelo envio de comunicações.

### Responsabilidades

- Receber solicitação de comunicação;
- Escolher canal;
- Montar template;
- Enviar comunicação fake;
- Controlar retry;
- Enviar falhas para DLQ;
- Registrar histórico de envio;
- Publicar eventos de comunicação.

### Endpoints planejados

| Método | Endpoint | Descrição |
|---|---|---|
| POST | `/api/v1/communications` | Solicita comunicação |
| GET | `/api/v1/communications/{id}` | Consulta comunicação |
| GET | `/api/v1/communications/customers/{customerId}` | Lista comunicações do cliente |

### Eventos consumidos

```text
LimitCalculated
LimitUpdated
CustomerEligibilityRejected
```

### Eventos publicados

```text
CommunicationRequested
CommunicationSent
CommunicationFailed
CommunicationDiscarded
```

### Banco

```text
communication_db
```

### Tabelas principais

```text
communication_templates
communication_requests
communication_delivery_history
outbox_events
inbox_events
```

### Filas RabbitMQ

```text
communication.send.queue
communication.send.retry.queue
communication.send.dlq
```

---

## credit-audit-service

Responsável pela trilha histórica da jornada.

### Responsabilidades

- Consumir eventos dos demais serviços;
- Registrar eventos auditados;
- Registrar timeline do cliente;
- Guardar payload bruto dos eventos;
- Permitir consulta por cliente;
- Permitir consulta por evento;
- Apoiar rastreabilidade técnica e de negócio.

### Endpoints planejados

| Método | Endpoint | Descrição |
|---|---|---|
| GET | `/api/v1/audits/customers/{customerId}/timeline` | Consulta timeline do cliente |
| GET | `/api/v1/audits/events/{eventId}` | Consulta evento por ID |
| GET | `/api/v1/audits` | Consulta eventos por filtros |

### Eventos consumidos

```text
CustomerCreated
CustomerUpdated
CustomerStatusChanged
CustomerDeleted
CustomerEligibilityApproved
CustomerEligibilityRejected
LimitCalculated
LimitUpdated
LimitBlocked
LimitReleased
CommunicationSent
CommunicationFailed
CommunicationDiscarded
CreditJourneyStarted
CreditJourneyCompleted
CreditJourneyFailed
```

### Banco

```text
audit_db
```

### Tabelas principais

```text
audit_events
customer_timeline
event_payloads
inbox_events
```

---

# Serviços de apoio

---

## credit-config-server

Responsável pela configuração centralizada.

### Responsabilidades

- Servir configurações por ambiente;
- Centralizar properties dos serviços;
- Controlar configurações externas;
- Apoiar feature toggles simples;
- Evitar configuração fixa no código.

### Porta padrão

```text
8888
```

### Exemplos de configuração

```text
URLs internas dos serviços
tópicos Kafka
filas RabbitMQ
feature toggles
nível de log
timeouts
```

---

## credit-platform-infra

Responsável pela infraestrutura local e preparação para Kubernetes.

### Responsabilidades

- Docker Compose;
- MySQL;
- Kafka;
- RabbitMQ;
- RabbitMQ Management;
- Prometheus;
- Grafana;
- OpenTelemetry Collector;
- Jaeger;
- Kubernetes manifests;
- Configuração de rede local;
- Documentação de portas e comandos.

### Observação

Esse repositório não é um microservice.

Ele centraliza infraestrutura.

---

## credit-shared-contracts

Responsável pela documentação e versionamento de contratos.

### Responsabilidades

- OpenAPI dos endpoints REST;
- AsyncAPI dos eventos;
- Schemas JSON dos eventos;
- Exemplos de payloads;
- Versionamento de contratos;
- Documentação de tópicos Kafka;
- Documentação de filas RabbitMQ.

### Estrutura esperada

```text
openapi/
asyncapi/
schemas/
examples/
```

---

## credit-observability-starter

Lib compartilhada responsável por padronizar observabilidade entre os serviços.

### Responsabilidades

- Fornecer `@LogInfo`;
- Fornecer `@LogParameter`;
- Aplicar logs automáticos com AOP;
- Configurar `correlationId`;
- Configurar MDC;
- Mascarar dados sensíveis;
- Padronizar campos de log;
- Apoiar tracing distribuído.

### Observação

Esse repositório não é um microservice.

Ele será usado como dependência pelos serviços.

---

# Comunicação entre serviços

## REST

Usado quando a resposta imediata é necessária.

Exemplos:

```text
orchestrator -> customer-service
orchestrator -> rules-engine-service
orchestrator -> limit-service
gateway -> serviços internos
```

---

## Kafka

Usado para eventos de domínio.

Exemplos:

```text
CustomerCreated
CustomerEligibilityApproved
LimitCalculated
CommunicationSent
CreditJourneyCompleted
```

---

## RabbitMQ

Usado para tarefas assíncronas, retry e DLQ.

Exemplos:

```text
SendCommunicationCommand
ReprocessLimitCommand
```

---

# Ordem prática de implementação

```text
1. credit-customer-service
2. credit-limit-service
3. credit-observability-starter
4. credit-shared-contracts
5. credit-platform-infra
6. credit-rules-engine-service
7. credit-orchestrator-service
8. credit-auth-service
9. credit-api-gateway
10. credit-communication-service
11. credit-audit-service
12. credit-config-server
```
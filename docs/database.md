# Banco de Dados

Este documento descreve a estratégia de banco de dados da **Credit Journey Platform**.

A plataforma utiliza a abordagem `database per service`, onde cada microservice é dono dos seus próprios dados.

Essa decisão evita acoplamento direto entre domínios e permite que cada serviço evolua seu modelo de dados de forma independente.

---

## Estratégia principal

Cada serviço possui seu próprio banco ou schema.

Um serviço não deve acessar diretamente o banco de outro serviço.

A comunicação entre serviços deve ocorrer por:

- API REST;
- Eventos Kafka;
- Filas RabbitMQ.

Exemplo:

```text
credit-customer-service não acessa limit_db
credit-limit-service não acessa customer_db
credit-audit-service não acessa diretamente os bancos dos outros serviços
```

Quando um serviço precisar de dados de outro domínio, ele deve usar API ou consumir eventos.

---

## Bancos planejados

| Serviço | Banco |
|---|---|
| `credit-auth-service` | `auth_db` |
| `credit-customer-service` | `customer_db` |
| `credit-rules-engine-service` | `rules_db` |
| `credit-limit-service` | `limit_db` |
| `credit-orchestrator-service` | `orchestrator_db` |
| `credit-communication-service` | `communication_db` |
| `credit-audit-service` | `audit_db` |

Observação:

```text
credit-api-gateway não precisa de banco próprio inicialmente.
credit-observability-starter não possui banco, pois é uma lib compartilhada.
credit-shared-contracts não possui banco, pois centraliza contratos.
credit-platform-infra não possui banco de aplicação, apenas configura a infraestrutura.
```

---

## Banco utilizado

O banco escolhido para o projeto será:

```text
MySQL
```

Motivos:

- simples de executar localmente;
- fácil integração com Spring Boot;
- bom suporte com Docker;
- compatível com Flyway;
- suficiente para demonstrar persistência relacional;
- adequado para o objetivo técnico do portfólio.

---

## Flyway

As alterações de banco serão versionadas com Flyway.

Cada serviço terá sua própria pasta de migrations:

```text
src/main/resources/db/migration/
```

Exemplo:

```text
V1__create_customers_table.sql
V2__add_unique_constraints_to_customers.sql
V3__create_customers_pagination_index.sql
V4__create_outbox_events_table.sql
```

---

## Regras para migrations

- Nunca alterar migration já aplicada.
- Criar uma nova migration para cada mudança.
- Usar nomes claros.
- Evitar mudanças destrutivas.
- Separar mudanças estruturais de inserts de carga inicial.
- Garantir compatibilidade com deploy gradual quando possível.
- Não criar migration genérica com muitas responsabilidades.
- Não depender de tabela de outro serviço.
- Toda mudança de schema deve ser rastreável.

Exemplo ruim:

```text
V2__update.sql
```

Exemplo bom:

```text
V2__add_unique_constraints_to_customers.sql
```

---

## Regras gerais de modelagem

- Usar UUID como identificador externo.
- Usar `BigDecimal` para valores monetários.
- Usar `created_at` e `updated_at` em tabelas transacionais.
- Usar índices em campos de busca frequente.
- Não compartilhar tabelas entre serviços.
- Não fazer joins entre bancos de serviços diferentes.
- Não expor Entity JPA fora do adapter de persistência.
- Separar modelo de domínio de modelo de banco.
- Versionar toda mudança com Flyway.
- Usar tabelas append-only para auditoria e histórico.
- Eventos devem possuir `event_id`.
- Mensagens processadas devem ser registradas para idempotência quando necessário.

---

# Bancos por serviço

---

## credit-auth-service

Banco:

```text
auth_db
```

Responsável por armazenar dados de autenticação, usuários de acesso, roles, refresh tokens e permissões.

---

### users

Armazena usuários de acesso da plataforma.

Campos esperados:

```text
user_id
customer_id
name
email
password_hash
status
created_at
updated_at
```

Observações:

- `user_id` é o identificador do usuário de autenticação.
- `customer_id` pode vincular o usuário ao cliente cadastrado no `credit-customer-service`.
- A senha nunca deve ser salva em texto puro.
- Deve ser usado hash de senha.

---

### roles

Armazena os perfis de acesso.

Campos esperados:

```text
role_id
name
description
created_at
updated_at
```

Exemplos de roles:

```text
CUSTOMER
ADMIN
SUPPORT
SYSTEM
```

---

### user_roles

Relaciona usuários com roles.

Campos esperados:

```text
id
user_id
role_id
created_at
```

---

### refresh_tokens

Armazena refresh tokens emitidos.

Campos esperados:

```text
id
token_id
user_id
token_hash
status
expires_at
created_at
revoked_at
```

Observações:

- O refresh token também não deve ser salvo em texto puro.
- O ideal é salvar hash do token.
- Tokens revogados devem ser mantidos para rastreabilidade.

---

### login_attempts

Armazena tentativas de login.

Campos esperados:

```text
id
user_id
email
success
failure_reason
ip_address
user_agent
attempted_at
```

Uso:

- auditoria;
- investigação de falhas;
- métrica de login;
- possível bloqueio por excesso de tentativa.

---

## credit-customer-service

Banco:

```text
customer_db
```

Responsável por armazenar dados cadastrais do cliente.

---

### customers

Armazena dados principais do cliente.

Campos esperados:

```text
customer_id
name
document
email
phone
income
status
created_at
updated_at
```

Observações:

- `customer_id` deve ser UUID.
- `document` deve ser único.
- `email` deve ser único.
- `phone` pode ser único quando informado.
- `income` deve usar tipo decimal no banco.
- `status` representa o estado cadastral do cliente.

Status possíveis:

```text
ACTIVE
INACTIVE
BLOCKED
```

---

### customer_status_history

Armazena histórico de alteração de status do cliente.

Campos esperados:

```text
id
customer_id
previous_status
current_status
reason
changed_by
changed_at
```

Uso:

- rastrear mudança de status;
- apoiar suporte operacional;
- permitir auditoria de bloqueio, desbloqueio e inativação.

---

### outbox_events

Armazena eventos de cliente a serem publicados.

Campos esperados:

```text
id
event_id
event_type
aggregate_id
aggregate_type
payload
status
created_at
published_at
error_message
attempts
```

Eventos possíveis:

```text
CustomerCreated
CustomerUpdated
CustomerStatusChanged
CustomerDeleted
```

Status possíveis:

```text
PENDING
PUBLISHED
FAILED
```

---

## credit-rules-engine-service

Banco:

```text
rules_db
```

Responsável por armazenar políticas de crédito, decisões de elegibilidade e histórico de regras avaliadas.

---

### credit_policies

Armazena políticas de crédito configuráveis.

Campos esperados:

```text
policy_id
policy_code
description
active
created_at
updated_at
```

Exemplos:

```text
MINIMUM_INCOME_RULE
BLOCKED_CUSTOMER_RULE
INACTIVE_CUSTOMER_RULE
```

---

### eligibility_decisions

Armazena decisões de elegibilidade.

Campos esperados:

```text
decision_id
customer_id
decision
reason
score
evaluated_at
created_at
```

Decisões possíveis:

```text
APPROVED
REJECTED
MANUAL_ANALYSIS
```

---

### rule_evaluation_history

Armazena detalhes das regras avaliadas.

Campos esperados:

```text
id
customer_id
decision_id
rule_code
result
reason
evaluated_at
```

Exemplos de resultado:

```text
PASSED
FAILED
SKIPPED
```

---

### outbox_events

Armazena eventos de elegibilidade a serem publicados.

Campos esperados:

```text
id
event_id
event_type
aggregate_id
aggregate_type
payload
status
created_at
published_at
error_message
attempts
```

Eventos possíveis:

```text
CustomerEligibilityApproved
CustomerEligibilityRejected
CustomerManualAnalysisRequested
```

---

### inbox_events

Armazena eventos já consumidos para garantir idempotência.

Campos esperados:

```text
id
event_id
event_type
source
processed_at
```

Uso:

```text
evitar processar duas vezes o mesmo CustomerCreated
```

---

## credit-limit-service

Banco:

```text
limit_db
```

Responsável por armazenar limite atual, histórico de alterações e eventos de limite.

---

### credit_limits

Armazena o limite atual do cliente.

Campos esperados:

```text
limit_id
customer_id
total_limit
available_limit
used_limit
status
created_at
updated_at
```

Observações:

- `total_limit`, `available_limit` e `used_limit` devem usar decimal.
- O limite deve pertencer a um `customer_id`.
- O serviço não deve consultar diretamente a tabela `customers`.

Status possíveis:

```text
ACTIVE
BLOCKED
CANCELLED
```

---

### limit_change_history

Armazena histórico de alterações de limite.

Campos esperados:

```text
id
limit_id
customer_id
previous_total_limit
new_total_limit
previous_available_limit
new_available_limit
change_type
reason
changed_at
```

Tipos de alteração:

```text
CREATED
INCREASED
DECREASED
BLOCKED
RELEASED
CANCELLED
```

---

### limit_reservations

Armazena reservas temporárias de limite, se necessário no futuro.

Campos esperados:

```text
reservation_id
limit_id
customer_id
amount
status
reason
created_at
expires_at
updated_at
```

Status possíveis:

```text
ACTIVE
CONFIRMED
CANCELLED
EXPIRED
```

---

### outbox_events

Armazena eventos de limite a serem publicados.

Campos esperados:

```text
id
event_id
event_type
aggregate_id
aggregate_type
payload
status
created_at
published_at
error_message
attempts
```

Eventos possíveis:

```text
LimitCalculated
LimitUpdated
LimitBlocked
LimitReleased
```

---

### inbox_events

Armazena eventos já consumidos para idempotência.

Campos esperados:

```text
id
event_id
event_type
source
processed_at
```

Uso:

```text
evitar recalcular limite duas vezes para o mesmo evento de elegibilidade
```

---

## credit-orchestrator-service

Banco:

```text
orchestrator_db
```

Responsável por armazenar o estado da jornada de crédito, quando a orquestração precisar ser rastreada de forma transacional.

---

### credit_journeys

Armazena a jornada principal de crédito.

Campos esperados:

```text
journey_id
customer_id
status
current_step
correlation_id
started_at
finished_at
created_at
updated_at
```

Status possíveis:

```text
STARTED
CUSTOMER_VALIDATED
ELIGIBILITY_EVALUATED
LIMIT_CALCULATED
COMMUNICATION_REQUESTED
COMPLETED
REJECTED
FAILED
```

---

### credit_journey_steps

Armazena as etapas executadas na jornada.

Campos esperados:

```text
id
journey_id
step_name
status
started_at
finished_at
error_message
```

Exemplos de etapas:

```text
VALIDATE_CUSTOMER
EVALUATE_ELIGIBILITY
CALCULATE_LIMIT
SEND_COMMUNICATION
REGISTER_AUDIT
```

---

### orchestrator_errors

Armazena erros da jornada.

Campos esperados:

```text
id
journey_id
step_name
error_code
error_message
created_at
```

---

### outbox_events

Armazena eventos da jornada a serem publicados.

Campos esperados:

```text
id
event_id
event_type
aggregate_id
aggregate_type
payload
status
created_at
published_at
error_message
attempts
```

Eventos possíveis:

```text
CreditJourneyStarted
CreditJourneyCompleted
CreditJourneyFailed
```

---

## credit-communication-service

Banco:

```text
communication_db
```

Responsável por armazenar templates, solicitações de comunicação e histórico de envio.

---

### communication_templates

Armazena templates de comunicação.

Campos esperados:

```text
template_id
template_code
channel
subject
body
active
created_at
updated_at
```

Canais possíveis:

```text
EMAIL
PUSH
SMS
IN_APP
```

---

### communication_requests

Armazena solicitações de comunicação.

Campos esperados:

```text
request_id
customer_id
template_code
channel
status
correlation_id
created_at
updated_at
```

Status possíveis:

```text
PENDING
SENT
FAILED
DISCARDED
```

---

### communication_delivery_history

Armazena histórico de tentativas de envio.

Campos esperados:

```text
id
communication_request_id
attempt
status
error_message
sent_at
created_at
```

---

### inbox_events

Armazena eventos já consumidos para idempotência.

Campos esperados:

```text
id
event_id
event_type
source
processed_at
```

Uso:

```text
evitar envio duplicado de comunicação
```

---

### outbox_events

Armazena eventos de comunicação a serem publicados.

Campos esperados:

```text
id
event_id
event_type
aggregate_id
aggregate_type
payload
status
created_at
published_at
error_message
attempts
```

Eventos possíveis:

```text
CommunicationSent
CommunicationFailed
CommunicationDiscarded
```

---

## credit-audit-service

Banco:

```text
audit_db
```

Responsável por armazenar eventos auditados, payloads e timeline da jornada.

---

### audit_events

Armazena eventos auditados.

Campos esperados:

```text
id
event_id
event_type
source
customer_id
correlation_id
occurred_at
created_at
```

---

### customer_timeline

Armazena visão de timeline do cliente.

Campos esperados:

```text
id
customer_id
event_type
description
occurred_at
correlation_id
```

Exemplo:

```text
10:00 - CustomerCreated
10:01 - CustomerEligibilityApproved
10:02 - LimitCalculated
10:03 - CommunicationSent
```

---

### event_payloads

Armazena payload bruto do evento.

Campos esperados:

```text
id
event_id
payload
created_at
```

Observação:

```text
Essa tabela deve ser append-only.
```

---

### inbox_events

Armazena eventos já consumidos para idempotência.

Campos esperados:

```text
id
event_id
event_type
source
processed_at
```

Uso:

```text
evitar registrar o mesmo evento duas vezes na auditoria
```

---

## Tabelas técnicas comuns

Alguns serviços poderão possuir tabelas técnicas semelhantes.

---

### outbox_events

Usada para garantir publicação segura de eventos.

Campos recomendados:

```text
id
event_id
event_type
aggregate_id
aggregate_type
payload
status
created_at
published_at
error_message
attempts
```

Status recomendados:

```text
PENDING
PUBLISHED
FAILED
```

---

### inbox_events

Usada para garantir idempotência no consumo de eventos.

Campos recomendados:

```text
id
event_id
event_type
source
processed_at
```

Regra:

```text
event_id deve ser único.
```

---

## Índices recomendados

Índices devem ser criados conforme os padrões de consulta de cada serviço.

Exemplos:

### customers

```text
document
email
phone
status
created_at
```

### credit_limits

```text
customer_id
status
created_at
```

### eligibility_decisions

```text
customer_id
decision
evaluated_at
```

### communication_requests

```text
customer_id
status
correlation_id
created_at
```

### audit_events

```text
event_id
customer_id
correlation_id
occurred_at
```

### outbox_events

```text
status
created_at
event_id
```

### inbox_events

```text
event_id
```

---

## Tipos de dados recomendados

| Tipo de dado | Recomendação |
|---|---|
| Identificadores externos | UUID |
| Valores monetários | DECIMAL |
| Datas | DATETIME ou TIMESTAMP |
| Status | VARCHAR |
| Payload de evento | JSON ou TEXT |
| E-mail | VARCHAR |
| Documento | VARCHAR |
| Telefone | VARCHAR |

Para valores monetários no código:

```text
BigDecimal
```

---

## Boas práticas

- Usar UUID como identificador externo.
- Usar BigDecimal para valores monetários.
- Usar índices em campos de busca frequente.
- Não compartilhar tabelas entre serviços.
- Não usar joins entre bancos de serviços diferentes.
- Não expor Entity JPA fora do adapter de persistência.
- Separar modelo de domínio de modelo de banco.
- Versionar toda mudança com Flyway.
- Usar created_at e updated_at em tabelas transacionais.
- Usar tabelas append-only para auditoria.
- Usar outbox para publicação segura de eventos.
- Usar inbox para idempotência de consumo.
- Evitar migrations destrutivas.
- Não alterar migrations já aplicadas.
- Não salvar senha em texto puro.
- Não salvar refresh token em texto puro.
- Não salvar dados sensíveis desnecessários.
- Mascarar dados sensíveis nos logs.

---

## Objetivo da estratégia de banco

A estratégia de banco da plataforma deve permitir:

- independência entre serviços;
- baixo acoplamento;
- evolução isolada de schemas;
- rastreabilidade;
- consistência eventual;
- segurança na publicação de eventos;
- idempotência no consumo de mensagens;
- facilidade de testes;
- simulação realista de produção.
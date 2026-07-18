# Roadmap de Desenvolvimento

Este roadmap organiza o desenvolvimento completo da **Credit Journey Platform** em fases incrementais.

A ordem foi pensada para facilitar aprendizado, validação prática e evolução natural da arquitetura, sem construir todos os microservices isoladamente antes de ter um fluxo real funcionando.

---

## Legenda

- `[ ]` Pendente
- `[x]` Concluído

---

## Status atual

```text
credit-customer-service: CRUD básico implementado
```

Funcionalidades já implementadas no `credit-customer-service`:

- criação de cliente;
- consulta por ID;
- listagem com paginação;
- filtros por status, search, name e income;
- atualização parcial;
- alteração de status;
- exclusão real;
- validações de domínio;
- exceptions específicas;
- handler global de erro;
- Swagger/OpenAPI;
- Actuator;
- Flyway;
- MySQL;
- testes unitários;
- JaCoCo;
- ktlint;
- logs automáticos com AOP.

---

# Fase 1 — Documentação e visão geral da plataforma

- [x] Criar repositório `credit-journey-platform`
- [x] Criar `README.md` principal da plataforma
- [x] Criar pasta `docs`
- [x] Criar `docs/architecture.md`
- [x] Criar `docs/services.md`
- [x] Criar `docs/messaging.md`
- [x] Criar `docs/database.md`
- [x] Criar `docs/deployment.md`
- [x] Criar `docs/observability.md`
- [x] Criar `docs/roadmap.md`
- [x] Atualizar documentação com `credit-auth-service`
- [x] Atualizar documentação com `credit-api-gateway`
- [x] Atualizar documentação com `credit-orchestrator-service`
- [x] Atualizar documentação com `credit-observability-starter`
- [ ] Atualizar README principal com links para todos os repositórios
- [ ] Adicionar diagrama textual da arquitetura geral
- [ ] Adicionar explicação do fluxo ponta a ponta da jornada de crédito
- [ ] Revisar documentação final da fase inicial

---

# Fase 2 — Customer Service: estrutura inicial

- [x] Criar repositório `credit-customer-service`
- [x] Criar projeto Kotlin com Spring Boot
- [x] Configurar Java 21
- [x] Configurar Gradle Kotlin DSL
- [x] Configurar `application.yml`
- [x] Configurar Docker Compose com MySQL
- [x] Configurar conexão com MySQL
- [x] Configurar Flyway
- [x] Validar criação da tabela `flyway_schema_history`
- [x] Configurar Actuator
- [x] Validar `/actuator/health`
- [x] Validar `/actuator/metrics`
- [x] Criar estrutura inicial de Arquitetura Hexagonal
- [x] Criar README técnico do serviço
- [x] Configurar Swagger/OpenAPI
- [x] Configurar ktlint
- [x] Configurar JaCoCo
- [ ] Remover arquivos desnecessários do repositório, se existirem
- [ ] Padronizar branch principal como `main`, se necessário

---

# Fase 3 — Customer Service: domínio e CRUD básico

- [x] Criar enum `CustomerStatus`
- [x] Criar domínio `Customer`
- [x] Criar value object `Document`
- [x] Criar value object `Email`
- [x] Criar value object `Income`
- [x] Criar exception `CustomerAlreadyExistsException`
- [x] Criar exception `CustomerNotFoundException`
- [x] Criar porta de entrada `CreateCustomerPort`
- [x] Criar porta de entrada `FindCustomerByIdPort`
- [x] Criar porta de entrada `FindAllCustomersPort`
- [x] Criar porta de entrada `UpdateCustomerPort`
- [x] Criar porta de entrada `ChangeCustomerStatusPort`
- [x] Criar porta de entrada `DeleteCustomerPort`
- [x] Criar porta de saída `CustomerRepositoryPort`
- [x] Criar caso de uso `CreateCustomerUseCase`
- [x] Criar caso de uso `FindCustomerByIdUseCase`
- [x] Criar caso de uso `FindAllCustomersUseCase`
- [x] Criar caso de uso `UpdateCustomerUseCase`
- [x] Criar caso de uso `ChangeCustomerStatusUseCase`
- [x] Criar caso de uso `DeleteCustomerUseCase`
- [x] Criar DTO `CreateCustomerRequest`
- [x] Criar DTO `UpdateCustomerRequest`
- [x] Criar DTO `ChangeCustomerStatusRequest`
- [x] Criar DTO `CustomerResponse`
- [x] Criar DTO `CustomerSliceResponse`
- [x] Criar mappers web
- [x] Criar controller `CustomerController`
- [x] Criar interface Swagger `CustomerApi`
- [x] Criar endpoint `POST /api/v1/customers`
- [x] Criar endpoint `GET /api/v1/customers/{customerId}`
- [x] Criar endpoint `GET /api/v1/customers`
- [x] Criar endpoint `PATCH /api/v1/customers/{customerId}`
- [x] Criar endpoint `PUT /api/v1/customers/{customerId}/status`
- [x] Criar endpoint `DELETE /api/v1/customers/{customerId}`
- [x] Criar migration `V1__create_customers_table.sql`
- [x] Criar migrations de constraints e índices
- [x] Criar entity `CustomerEntity`
- [x] Criar repository Spring Data
- [x] Criar adapter de persistência
- [x] Validar criação de cliente via curl/Postman
- [x] Validar persistência no MySQL
- [x] Validar filtros e paginação
- [x] Validar update parcial
- [x] Validar alteração de status
- [x] Validar delete real

---

# Fase 4 — Customer Service: qualidade, logs e testes

- [x] Criar testes unitários de `CreateCustomerUseCase`
- [x] Criar testes unitários de `FindCustomerByIdUseCase`
- [x] Criar testes unitários de `FindAllCustomersUseCase`
- [x] Criar testes unitários de `UpdateCustomerUseCase`
- [x] Criar testes unitários de `ChangeCustomerStatusUseCase`
- [x] Criar testes unitários de `DeleteCustomerUseCase`
- [x] Criar builders de teste
- [x] Configurar JaCoCo para cobertura do core
- [x] Validar cobertura dos use cases
- [x] Criar annotation `@LogInfo`
- [x] Criar annotation `@LogParameter`
- [x] Criar `LogInfoAspect`
- [x] Aplicar logs automáticos nos use cases
- [x] Validar logs automáticos
- [x] Rodar `ktlintFormat`
- [x] Rodar `ktlintCheck`
- [x] Rodar `clean test`
- [ ] Revisar README técnico do `credit-customer-service`
- [ ] Criar commit final do CRUD básico

---

# Fase 5 — Credit Limit Service: estrutura inicial

- [ ] Criar repositório `credit-limit-service`
- [ ] Criar projeto Kotlin com Spring Boot
- [ ] Configurar Java 21
- [ ] Configurar Gradle Kotlin DSL
- [ ] Configurar `application.yml`
- [ ] Configurar Docker Compose com MySQL
- [ ] Configurar banco `limit_db`
- [ ] Configurar Flyway
- [ ] Configurar Actuator
- [ ] Configurar Swagger/OpenAPI
- [ ] Configurar ktlint
- [ ] Configurar JaCoCo
- [ ] Criar estrutura inicial de Arquitetura Hexagonal
- [ ] Criar README técnico do serviço
- [ ] Validar `/actuator/health`
- [ ] Validar conexão com banco
- [ ] Validar `flyway_schema_history`

---

# Fase 6 — Credit Limit Service: domínio e CRUD básico

- [ ] Criar domínio `CreditLimit`
- [ ] Criar enum `LimitStatus`
- [ ] Criar value object `Money`
- [ ] Garantir uso de `BigDecimal` para valores monetários
- [ ] Criar exception `CreditLimitNotFoundException`
- [ ] Criar exception `CreditLimitAlreadyExistsException`
- [ ] Criar porta de entrada `CreateCreditLimitPort`
- [ ] Criar porta de entrada `FindCreditLimitByCustomerIdPort`
- [ ] Criar porta de entrada `UpdateCreditLimitPort`
- [ ] Criar porta de entrada `BlockCreditLimitPort`
- [ ] Criar porta de entrada `ReleaseCreditLimitPort`
- [ ] Criar porta de saída `CreditLimitRepositoryPort`
- [ ] Criar caso de uso `CreateCreditLimitUseCase`
- [ ] Criar caso de uso `FindCreditLimitByCustomerIdUseCase`
- [ ] Criar caso de uso `UpdateCreditLimitUseCase`
- [ ] Criar caso de uso `BlockCreditLimitUseCase`
- [ ] Criar caso de uso `ReleaseCreditLimitUseCase`
- [ ] Criar DTO `CreateCreditLimitRequest`
- [ ] Criar DTO `UpdateCreditLimitRequest`
- [ ] Criar DTO `CreditLimitResponse`
- [ ] Criar mappers web
- [ ] Criar controller `CreditLimitController`
- [ ] Criar interface Swagger `CreditLimitApi`
- [ ] Criar endpoint `POST /api/v1/limits`
- [ ] Criar endpoint `GET /api/v1/limits/customers/{customerId}`
- [ ] Criar endpoint `PATCH /api/v1/limits/{limitId}`
- [ ] Criar endpoint `PATCH /api/v1/limits/{limitId}/block`
- [ ] Criar endpoint `PATCH /api/v1/limits/{limitId}/release`
- [ ] Criar migration `V1__create_credit_limits_table.sql`
- [ ] Criar migration `V2__create_limit_change_history_table.sql`
- [ ] Criar entity `CreditLimitEntity`
- [ ] Criar repository Spring Data
- [ ] Criar adapter de persistência
- [ ] Validar criação de limite
- [ ] Validar consulta por `customerId`
- [ ] Validar atualização de limite
- [ ] Validar bloqueio de limite
- [ ] Validar liberação de limite
- [ ] Validar histórico de alteração

---

# Fase 7 — Credit Limit Service: cálculo inicial

- [ ] Criar porta de entrada `CalculateInitialLimitPort`
- [ ] Criar caso de uso `CalculateInitialLimitUseCase`
- [ ] Criar regra simples de cálculo por renda
- [ ] Criar input `CalculateInitialLimitInput`
- [ ] Criar endpoint `POST /api/v1/limits/calculate`
- [ ] Validar cálculo com renda baixa
- [ ] Validar cálculo com renda média
- [ ] Validar cálculo com renda alta
- [ ] Validar limite mínimo
- [ ] Validar limite máximo
- [ ] Persistir limite calculado
- [ ] Registrar histórico `CREATED`
- [ ] Criar testes unitários do cálculo
- [ ] Criar testes unitários dos use cases
- [ ] Validar cobertura JaCoCo
- [ ] Rodar `ktlintCheck`
- [ ] Rodar `clean test`

---

# Fase 8 — Observability Starter

- [ ] Criar repositório `credit-observability-starter`
- [ ] Criar projeto Kotlin/Java para lib compartilhada
- [ ] Configurar Gradle
- [ ] Configurar publicação local da lib
- [ ] Criar README técnico da lib
- [ ] Migrar `@LogInfo` para a lib
- [ ] Migrar `@LogParameter` para a lib
- [ ] Migrar `LogInfoAspect` para a lib
- [ ] Criar `CorrelationIdFilter`
- [ ] Criar configuração de MDC
- [ ] Criar utilitário de máscara de dados sensíveis
- [ ] Criar padrão de campos de log
- [ ] Criar auto configuration, se aplicável
- [ ] Criar testes da lib
- [ ] Integrar lib no `credit-customer-service`
- [ ] Integrar lib no `credit-limit-service`
- [ ] Remover duplicação de logs dos serviços
- [ ] Validar logs padronizados nos dois serviços

---

# Fase 9 — Shared Contracts

- [ ] Criar repositório `credit-shared-contracts`
- [ ] Criar README do repositório de contratos
- [ ] Criar pasta `openapi`
- [ ] Criar pasta `asyncapi`
- [ ] Criar pasta `schemas`
- [ ] Documentar OpenAPI do `credit-customer-service`
- [ ] Documentar OpenAPI do `credit-limit-service`
- [ ] Criar padrão comum de envelope de evento
- [ ] Definir campos obrigatórios: `eventId`, `eventType`, `eventVersion`, `source`, `correlationId`, `occurredAt`, `payload`
- [ ] Criar schema JSON do evento `CustomerCreated`
- [ ] Criar schema JSON do evento `CustomerUpdated`
- [ ] Criar schema JSON do evento `CustomerStatusChanged`
- [ ] Criar schema JSON do evento `CustomerDeleted`
- [ ] Criar schema JSON do evento `LimitCalculated`
- [ ] Criar schema JSON do evento `LimitUpdated`
- [ ] Criar schema JSON do evento `LimitBlocked`
- [ ] Criar schema JSON do evento `LimitReleased`
- [ ] Documentar estratégia de versionamento de eventos

---

# Fase 10 — Infraestrutura local compartilhada

- [ ] Criar repositório `credit-platform-infra`
- [ ] Criar README do repositório de infra
- [ ] Criar Docker Compose principal da plataforma
- [ ] Adicionar MySQL com múltiplos bancos
- [ ] Criar banco `auth_db`
- [ ] Criar banco `customer_db`
- [ ] Criar banco `limit_db`
- [ ] Criar banco `rules_db`
- [ ] Criar banco `orchestrator_db`
- [ ] Criar banco `communication_db`
- [ ] Criar banco `audit_db`
- [ ] Adicionar Kafka
- [ ] Adicionar RabbitMQ
- [ ] Adicionar RabbitMQ Management
- [ ] Adicionar Prometheus
- [ ] Adicionar Grafana
- [ ] Adicionar OpenTelemetry Collector
- [ ] Adicionar Jaeger
- [ ] Criar rede Docker compartilhada
- [ ] Documentar portas usadas localmente
- [ ] Documentar comandos para subir, parar e limpar ambiente
- [ ] Validar subida completa da infraestrutura

---

# Fase 11 — Rules Engine Service: estrutura inicial

- [ ] Criar repositório `credit-rules-engine-service`
- [ ] Criar projeto Kotlin com Spring Boot
- [ ] Configurar Java 21
- [ ] Configurar Gradle Kotlin DSL
- [ ] Criar estrutura hexagonal
- [ ] Configurar MySQL `rules_db`
- [ ] Configurar Flyway
- [ ] Configurar Actuator
- [ ] Configurar Swagger/OpenAPI
- [ ] Configurar ktlint
- [ ] Configurar JaCoCo
- [ ] Adicionar `credit-observability-starter`
- [ ] Criar README técnico do serviço
- [ ] Validar `/actuator/health`
- [ ] Validar conexão com banco
- [ ] Validar `flyway_schema_history`

---

# Fase 12 — Rules Engine Service: elegibilidade

- [ ] Criar domínio `EligibilityDecision`
- [ ] Criar enum `EligibilityStatus`
- [ ] Criar domínio `CreditPolicy`
- [ ] Criar policy `BlockedCustomerPolicy`
- [ ] Criar policy `InactiveCustomerPolicy`
- [ ] Criar policy `MinimumIncomePolicy`
- [ ] Criar porta de entrada `EvaluateCustomerEligibilityPort`
- [ ] Criar porta de saída `EligibilityDecisionRepositoryPort`
- [ ] Criar caso de uso `EvaluateCustomerEligibilityUseCase`
- [ ] Criar migration `V1__create_eligibility_decisions_table.sql`
- [ ] Criar migration `V2__create_rule_evaluation_history_table.sql`
- [ ] Criar entity `EligibilityDecisionEntity`
- [ ] Criar repository Spring Data
- [ ] Criar adapter de persistência
- [ ] Criar endpoint `POST /api/v1/rules/evaluate`
- [ ] Criar endpoint `GET /api/v1/rules/customers/{customerId}/eligibility`
- [ ] Validar cliente aprovado
- [ ] Validar cliente rejeitado
- [ ] Validar cliente em análise manual
- [ ] Persistir decisão de elegibilidade
- [ ] Persistir histórico das regras avaliadas
- [ ] Criar testes unitários
- [ ] Validar cobertura JaCoCo

---

# Fase 13 — Orchestrator Service

- [ ] Criar repositório `credit-orchestrator-service`
- [ ] Criar projeto Kotlin com Spring Boot
- [ ] Configurar Java 21
- [ ] Criar estrutura hexagonal
- [ ] Configurar MySQL `orchestrator_db`
- [ ] Configurar Flyway
- [ ] Configurar Actuator
- [ ] Configurar Swagger/OpenAPI
- [ ] Adicionar `credit-observability-starter`
- [ ] Criar README técnico do serviço
- [ ] Criar domínio `CreditJourney`
- [ ] Criar enum `CreditJourneyStatus`
- [ ] Criar enum `CreditJourneyStep`
- [ ] Criar porta de entrada `StartCreditJourneyPort`
- [ ] Criar porta de saída `CustomerClientPort`
- [ ] Criar porta de saída `RulesEngineClientPort`
- [ ] Criar porta de saída `CreditLimitClientPort`
- [ ] Criar porta de saída `CommunicationClientPort`
- [ ] Criar porta de saída `AuditClientPort`
- [ ] Criar caso de uso `StartCreditJourneyUseCase`
- [ ] Criar clients HTTP para serviços internos
- [ ] Criar migration `V1__create_credit_journeys_table.sql`
- [ ] Criar migration `V2__create_credit_journey_steps_table.sql`
- [ ] Criar endpoint `POST /api/v1/credit-journeys`
- [ ] Criar endpoint `GET /api/v1/credit-journeys/{journeyId}`
- [ ] Validar fluxo com customer + rules + limit
- [ ] Registrar etapa atual da jornada
- [ ] Registrar falhas por etapa
- [ ] Criar testes unitários
- [ ] Criar testes com clients mockados

---

# Fase 14 — Auth Service

- [ ] Criar repositório `credit-auth-service`
- [ ] Criar projeto Kotlin com Spring Boot
- [ ] Configurar Java 21
- [ ] Criar estrutura hexagonal
- [ ] Configurar MySQL `auth_db`
- [ ] Configurar Flyway
- [ ] Configurar Spring Security
- [ ] Configurar geração de JWT
- [ ] Configurar refresh token
- [ ] Adicionar `credit-observability-starter`
- [ ] Criar README técnico do serviço
- [ ] Criar domínio `User`
- [ ] Criar domínio `Role`
- [ ] Criar value object `Password`
- [ ] Criar porta de entrada `RegisterUserPort`
- [ ] Criar porta de entrada `LoginPort`
- [ ] Criar porta de entrada `RefreshTokenPort`
- [ ] Criar porta de saída `UserRepositoryPort`
- [ ] Criar caso de uso `RegisterUserUseCase`
- [ ] Criar caso de uso `LoginUseCase`
- [ ] Criar caso de uso `RefreshTokenUseCase`
- [ ] Criar migration `V1__create_users_table.sql`
- [ ] Criar migration `V2__create_roles_table.sql`
- [ ] Criar migration `V3__create_user_roles_table.sql`
- [ ] Criar migration `V4__create_refresh_tokens_table.sql`
- [ ] Criar endpoint `POST /api/v1/auth/register`
- [ ] Criar endpoint `POST /api/v1/auth/login`
- [ ] Criar endpoint `POST /api/v1/auth/refresh`
- [ ] Validar login com sucesso
- [ ] Validar login inválido
- [ ] Validar geração de JWT
- [ ] Validar refresh token
- [ ] Criar testes unitários
- [ ] Criar testes de segurança

---

# Fase 15 — API Gateway

- [ ] Criar repositório `credit-api-gateway`
- [ ] Criar projeto Spring Cloud Gateway
- [ ] Configurar Java 21
- [ ] Configurar porta `8080`
- [ ] Configurar rotas para `credit-auth-service`
- [ ] Configurar rotas para `credit-customer-service`
- [ ] Configurar rotas para `credit-limit-service`
- [ ] Configurar rotas para `credit-rules-engine-service`
- [ ] Configurar rotas para `credit-orchestrator-service`
- [ ] Configurar rotas para `credit-communication-service`
- [ ] Configurar rotas para `credit-audit-service`
- [ ] Configurar validação de JWT
- [ ] Criar filtro de `correlationId`
- [ ] Propagar headers técnicos
- [ ] Configurar CORS
- [ ] Configurar rate limit simples
- [ ] Configurar Actuator
- [ ] Configurar logs estruturados
- [ ] Criar README técnico do gateway
- [ ] Validar chamada externa para customer via gateway
- [ ] Validar chamada externa para limit via gateway
- [ ] Validar bloqueio sem token
- [ ] Validar acesso com token válido
- [ ] Validar propagação de `correlationId`

---

# Fase 16 — Communication Service: estrutura inicial

- [ ] Criar repositório `credit-communication-service`
- [ ] Criar projeto Kotlin com Spring Boot
- [ ] Configurar Java 21
- [ ] Criar estrutura hexagonal
- [ ] Configurar MySQL `communication_db`
- [ ] Configurar Flyway
- [ ] Configurar Actuator
- [ ] Configurar RabbitMQ
- [ ] Configurar Swagger/OpenAPI
- [ ] Adicionar `credit-observability-starter`
- [ ] Criar README técnico do serviço
- [ ] Validar `/actuator/health`
- [ ] Validar conexão com banco
- [ ] Validar conexão com RabbitMQ

---

# Fase 17 — Communication Service: envio de comunicação

- [ ] Criar domínio `CommunicationRequest`
- [ ] Criar enum `CommunicationChannel`
- [ ] Criar enum `CommunicationStatus`
- [ ] Criar domínio `CommunicationTemplate`
- [ ] Criar porta de entrada `RequestCommunicationPort`
- [ ] Criar porta de saída `CommunicationRepositoryPort`
- [ ] Criar porta de saída `CommunicationProviderPort`
- [ ] Criar provider fake de e-mail
- [ ] Criar provider fake de push
- [ ] Criar migration `V1__create_communication_templates_table.sql`
- [ ] Criar migration `V2__create_communication_requests_table.sql`
- [ ] Criar migration `V3__create_communication_delivery_history_table.sql`
- [ ] Criar endpoint `POST /api/v1/communications`
- [ ] Criar endpoint `GET /api/v1/communications/{id}`
- [ ] Criar endpoint `GET /api/v1/communications/customers/{customerId}`
- [ ] Validar solicitação de comunicação
- [ ] Validar envio fake
- [ ] Validar histórico de envio
- [ ] Criar testes unitários

---

# Fase 18 — Communication Service: RabbitMQ, retry e DLQ

- [ ] Criar fila `communication.send.queue`
- [ ] Criar fila `communication.send.retry.queue`
- [ ] Criar fila `communication.send.dlq`
- [ ] Criar consumer da fila principal
- [ ] Criar publisher para fila de retry
- [ ] Criar publisher para DLQ
- [ ] Implementar controle de tentativas
- [ ] Implementar erro transitório com retry
- [ ] Implementar erro não recuperável com DLQ
- [ ] Criar campos `messageId`, `correlationId`, `attempt`
- [ ] Garantir idempotência por `messageId`
- [ ] Validar mensagem processada com sucesso
- [ ] Validar mensagem indo para retry
- [ ] Validar mensagem indo para DLQ
- [ ] Validar reprocessamento manual da DLQ

---

# Fase 19 — Audit Service: estrutura inicial

- [ ] Criar repositório `credit-audit-service`
- [ ] Criar projeto Kotlin com Spring Boot
- [ ] Configurar Java 21
- [ ] Criar estrutura hexagonal
- [ ] Configurar MySQL `audit_db`
- [ ] Configurar Flyway
- [ ] Configurar Actuator
- [ ] Configurar Swagger/OpenAPI
- [ ] Adicionar `credit-observability-starter`
- [ ] Criar README técnico do serviço
- [ ] Validar `/actuator/health`
- [ ] Validar conexão com banco
- [ ] Validar `flyway_schema_history`

---

# Fase 20 — Audit Service: timeline e eventos

- [ ] Criar domínio `AuditEvent`
- [ ] Criar domínio `CustomerTimeline`
- [ ] Criar porta de entrada `RegisterAuditEventPort`
- [ ] Criar porta de saída `AuditEventRepositoryPort`
- [ ] Criar caso de uso `RegisterAuditEventUseCase`
- [ ] Criar migration `V1__create_audit_events_table.sql`
- [ ] Criar migration `V2__create_customer_timeline_table.sql`
- [ ] Criar migration `V3__create_event_payloads_table.sql`
- [ ] Criar endpoint `GET /api/v1/audits/customers/{customerId}/timeline`
- [ ] Criar endpoint `GET /api/v1/audits/events/{eventId}`
- [ ] Criar endpoint `GET /api/v1/audits`
- [ ] Persistir evento auditado
- [ ] Persistir payload bruto
- [ ] Persistir visão de timeline
- [ ] Implementar idempotência por `eventId`
- [ ] Validar timeline completa de um cliente
- [ ] Criar testes unitários

---

# Fase 21 — Eventos, Outbox e Inbox

- [ ] Criar tabela `outbox_events` no `credit-customer-service`
- [ ] Criar tabela `outbox_events` no `credit-limit-service`
- [ ] Criar tabela `outbox_events` no `credit-rules-engine-service`
- [ ] Criar tabela `outbox_events` no `credit-communication-service`
- [ ] Criar tabela `outbox_events` no `credit-orchestrator-service`
- [ ] Criar tabela `inbox_events` nos serviços consumidores
- [ ] Implementar Outbox Pattern no Customer Service
- [ ] Implementar Outbox Pattern no Limit Service
- [ ] Implementar Outbox Pattern no Rules Engine Service
- [ ] Implementar Outbox Pattern no Communication Service
- [ ] Implementar Inbox Pattern no Rules Engine Service
- [ ] Implementar Inbox Pattern no Limit Service
- [ ] Implementar Inbox Pattern no Communication Service
- [ ] Implementar Inbox Pattern no Audit Service
- [ ] Configurar Kafka producer
- [ ] Configurar Kafka consumer
- [ ] Publicar `CustomerCreated`
- [ ] Consumir `CustomerCreated`
- [ ] Publicar `CustomerEligibilityApproved`
- [ ] Publicar `CustomerEligibilityRejected`
- [ ] Consumir `CustomerEligibilityApproved`
- [ ] Publicar `LimitCalculated`
- [ ] Consumir `LimitCalculated`
- [ ] Publicar `CommunicationSent`
- [ ] Consumir eventos no Audit Service
- [ ] Validar idempotência
- [ ] Validar reprocessamento básico

---

# Fase 22 — Config Server e feature toggles

- [ ] Criar repositório `credit-config-server`
- [ ] Criar projeto Spring Cloud Config Server
- [ ] Configurar porta `8888`
- [ ] Criar README técnico do serviço
- [ ] Criar configs locais do `credit-customer-service`
- [ ] Criar configs locais do `credit-limit-service`
- [ ] Criar configs locais do `credit-rules-engine-service`
- [ ] Criar configs locais do `credit-orchestrator-service`
- [ ] Criar configs locais do `credit-auth-service`
- [ ] Criar configs locais do `credit-api-gateway`
- [ ] Criar configs locais do `credit-communication-service`
- [ ] Criar configs locais do `credit-audit-service`
- [ ] Configurar serviços para buscar configs no Config Server
- [ ] Criar toggle `enable-risk-rule-v2`
- [ ] Criar toggle `enable-async-communication`
- [ ] Criar toggle `enable-limit-auto-approval`
- [ ] Criar toggle `enable-new-limit-calculation`
- [ ] Usar toggle no Rules Engine
- [ ] Usar toggle no Communication Service
- [ ] Usar toggle no Limit Service
- [ ] Validar mudança de comportamento por configuração

---

# Fase 23 — Fluxo ponta a ponta da jornada de crédito

- [ ] Subir infraestrutura completa
- [ ] Subir `credit-api-gateway`
- [ ] Subir `credit-auth-service`
- [ ] Subir `credit-customer-service`
- [ ] Subir `credit-limit-service`
- [ ] Subir `credit-rules-engine-service`
- [ ] Subir `credit-orchestrator-service`
- [ ] Subir `credit-communication-service`
- [ ] Subir `credit-audit-service`
- [ ] Criar usuário no Auth Service
- [ ] Realizar login
- [ ] Obter JWT
- [ ] Criar cliente via gateway
- [ ] Iniciar jornada pelo Orchestrator
- [ ] Validar consulta do cliente
- [ ] Validar decisão de elegibilidade
- [ ] Validar cálculo de limite
- [ ] Validar envio de comunicação
- [ ] Validar timeline no Audit Service
- [ ] Validar eventos Kafka
- [ ] Validar filas RabbitMQ
- [ ] Validar correlação por `correlationId`
- [ ] Documentar fluxo ponta a ponta no README principal

---

# Fase 24 — Observabilidade

- [ ] Configurar logs estruturados em JSON
- [ ] Adicionar `correlationId` em requests HTTP
- [ ] Propagar `correlationId` via gateway
- [ ] Propagar `correlationId` via orchestrator
- [ ] Propagar `correlationId` em eventos Kafka
- [ ] Propagar `correlationId` em mensagens RabbitMQ
- [ ] Adicionar `traceId` e `spanId`
- [ ] Configurar OpenTelemetry
- [ ] Configurar Jaeger
- [ ] Configurar endpoint `/actuator/prometheus`
- [ ] Configurar Prometheus para coletar métricas dos serviços
- [ ] Configurar Grafana
- [ ] Criar dashboard geral da plataforma
- [ ] Criar dashboard de HTTP requests
- [ ] Criar dashboard de autenticação
- [ ] Criar dashboard de jornada de crédito
- [ ] Criar dashboard de Kafka consumers/producers
- [ ] Criar dashboard de RabbitMQ
- [ ] Criar dashboard de DLQ
- [ ] Criar métrica de clientes criados
- [ ] Criar métrica de clientes aprovados
- [ ] Criar métrica de clientes rejeitados
- [ ] Criar métrica de limites calculados
- [ ] Criar métrica de comunicações enviadas
- [ ] Criar métrica de mensagens em DLQ
- [ ] Criar métrica de logins com sucesso
- [ ] Criar métrica de logins com falha
- [ ] Validar investigação por `correlationId`

---

# Fase 25 — Qualidade e testes

- [ ] Criar testes unitários do domínio no Customer Service
- [ ] Criar testes unitários dos use cases no Customer Service
- [ ] Criar testes de controller no Customer Service
- [ ] Criar testes de persistência no Customer Service
- [ ] Criar testes unitários do Limit Service
- [ ] Criar testes unitários do Rules Engine
- [ ] Criar testes unitários do Orchestrator Service
- [ ] Criar testes unitários do Auth Service
- [ ] Criar testes unitários do Gateway
- [ ] Criar testes unitários do Communication Service
- [ ] Criar testes unitários do Audit Service
- [ ] Criar testes de integração com MySQL
- [ ] Criar testes de integração com Kafka
- [ ] Criar testes de integração com RabbitMQ
- [ ] Criar testes de DLQ
- [ ] Criar testes de Outbox Pattern
- [ ] Criar testes de Inbox Pattern
- [ ] Criar testes de autenticação
- [ ] Criar testes de autorização
- [ ] Criar testes de arquitetura com ArchUnit
- [ ] Bloquear uso de field injection
- [ ] Bloquear uso de DTO no domínio
- [ ] Bloquear dependência de Spring no domínio
- [ ] Bloquear uso de Double para valores monetários
- [ ] Bloquear controller acessando repository diretamente

---

# Fase 26 — Segurança e validações

- [ ] Padronizar responses de erro
- [ ] Criar handler global de exceptions em todos os serviços
- [ ] Criar padrão de erro com `code`, `message`, `details`, `timestamp`
- [ ] Mascarar dados sensíveis em logs
- [ ] Evitar log de documento completo
- [ ] Evitar log de telefone completo
- [ ] Evitar log de e-mail completo quando necessário
- [ ] Evitar log de senha
- [ ] Evitar log de token completo
- [ ] Validar JWT no Gateway
- [ ] Validar roles e permissões
- [ ] Proteger endpoints internos
- [ ] Documentar decisões de segurança
- [ ] Validar fluxo com token inválido
- [ ] Validar fluxo sem token
- [ ] Validar fluxo com role insuficiente

---

# Fase 27 — Kubernetes

- [ ] Criar Dockerfile do `credit-api-gateway`
- [ ] Criar Dockerfile do `credit-auth-service`
- [ ] Criar Dockerfile do `credit-customer-service`
- [ ] Criar Dockerfile do `credit-limit-service`
- [ ] Criar Dockerfile do `credit-rules-engine-service`
- [ ] Criar Dockerfile do `credit-orchestrator-service`
- [ ] Criar Dockerfile do `credit-communication-service`
- [ ] Criar Dockerfile do `credit-audit-service`
- [ ] Criar manifests Kubernetes do Gateway
- [ ] Criar manifests Kubernetes do Auth Service
- [ ] Criar manifests Kubernetes do Customer Service
- [ ] Criar manifests Kubernetes do Limit Service
- [ ] Criar manifests Kubernetes do Rules Engine
- [ ] Criar manifests Kubernetes do Orchestrator
- [ ] Criar manifests Kubernetes do Communication Service
- [ ] Criar manifests Kubernetes do Audit Service
- [ ] Criar Deployment
- [ ] Criar Service
- [ ] Criar ConfigMap
- [ ] Criar Secret
- [ ] Criar Ingress
- [ ] Criar readinessProbe
- [ ] Criar livenessProbe
- [ ] Criar startupProbe quando necessário
- [ ] Configurar requests e limits
- [ ] Configurar HPA
- [ ] Documentar deploy local em Kubernetes

---

# Fase 28 — CI/CD

- [ ] Criar GitHub Actions para build do Customer Service
- [ ] Criar GitHub Actions para build do Limit Service
- [ ] Criar GitHub Actions para build do Rules Engine
- [ ] Criar GitHub Actions para build do Orchestrator Service
- [ ] Criar GitHub Actions para build do Auth Service
- [ ] Criar GitHub Actions para build do API Gateway
- [ ] Criar GitHub Actions para build do Communication Service
- [ ] Criar GitHub Actions para build do Audit Service
- [ ] Criar GitHub Actions para build do Observability Starter
- [ ] Rodar testes no pipeline
- [ ] Rodar ktlint no pipeline
- [ ] Rodar JaCoCo no pipeline
- [ ] Rodar análise estática no pipeline
- [ ] Gerar imagem Docker no pipeline
- [ ] Versionar imagem com commit SHA
- [ ] Publicar imagem
- [ ] Documentar estratégia de pipeline
- [ ] Adicionar badge de build nos READMEs

---

# Fase 29 — Documentação final para portfólio

- [ ] Atualizar README principal da plataforma
- [ ] Adicionar links para todos os repositórios
- [ ] Adicionar explicação do produto
- [ ] Adicionar fluxo ponta a ponta
- [ ] Adicionar mapa dos microservices
- [ ] Adicionar exemplos de payloads
- [ ] Adicionar exemplos de eventos
- [ ] Adicionar instruções para rodar localmente
- [ ] Adicionar instruções para subir infraestrutura
- [ ] Adicionar troubleshooting
- [ ] Adicionar prints do Swagger
- [ ] Adicionar prints do Grafana
- [ ] Adicionar prints do RabbitMQ
- [ ] Adicionar prints do Jaeger
- [ ] Adicionar prints do fluxo funcionando
- [ ] Criar resumo técnico para LinkedIn
- [ ] Criar roteiro de explicação para entrevista
- [ ] Criar seção “Decisões arquiteturais”
- [ ] Criar seção “Problemas reais simulados”
- [ ] Criar seção “Melhorias futuras”

---

# Fase 30 — Validação final

- [ ] Rodar todos os serviços localmente
- [ ] Criar usuário
- [ ] Realizar login
- [ ] Obter JWT
- [ ] Criar cliente via API Gateway
- [ ] Validar banco do Customer Service
- [ ] Iniciar jornada pelo Orchestrator
- [ ] Validar evento no Kafka
- [ ] Validar elegibilidade
- [ ] Validar banco do Rules Engine
- [ ] Validar cálculo de limite
- [ ] Validar banco do Limit Service
- [ ] Validar envio de comunicação
- [ ] Validar RabbitMQ
- [ ] Validar DLQ
- [ ] Validar banco do Communication Service
- [ ] Validar timeline no Audit Service
- [ ] Validar logs com `correlationId`
- [ ] Validar traces no Jaeger
- [ ] Validar métricas no Prometheus
- [ ] Validar dashboards no Grafana
- [ ] Validar documentação
- [ ] Validar todos os READMEs
- [ ] Validar GitHub organizado
- [ ] Criar release inicial da plataforma

---

# Ordem prática recomendada

```text
1. Finalizar ajustes do credit-customer-service
2. Criar credit-limit-service
3. Criar credit-observability-starter
4. Criar credit-shared-contracts
5. Criar credit-platform-infra
6. Criar credit-rules-engine-service
7. Criar credit-orchestrator-service
8. Criar credit-auth-service
9. Criar credit-api-gateway
10. Criar credit-communication-service
11. Criar credit-audit-service
12. Implementar eventos, outbox e inbox
13. Fechar fluxo ponta a ponta
14. Adicionar Config Server e feature toggles
15. Adicionar observabilidade completa
16. Adicionar testes de integração e ArchUnit
17. Adicionar Kubernetes
18. Adicionar CI/CD
19. Finalizar documentação de portfólio
20. Validar release inicial
```

---

# Próximo passo imediato

```text
Criar o repositório credit-limit-service
```

Objetivo do próximo serviço:

```text
criar, consultar, atualizar, bloquear e liberar limite de crédito por customerId
```
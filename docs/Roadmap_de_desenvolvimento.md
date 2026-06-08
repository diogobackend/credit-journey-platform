# Roadmap de Desenvolvimento

Este roadmap organiza o desenvolvimento completo da **Credit Journey Platform** em tasks incrementais.

A ordem foi pensada para facilitar aprendizado, validação prática e evolução natural da arquitetura, sem construir todos os microservices isoladamente antes de ter um fluxo real funcionando.

---

## Legenda

- `[ ]` Pendente
- `[x]` Concluído

---

## Fase 1 — Documentação e visão geral da plataforma

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
- [ ] Atualizar README principal com links para todos os repositórios
- [ ] Adicionar diagrama textual da arquitetura geral
- [ ] Adicionar explicação do fluxo ponta a ponta da jornada de crédito

---

## Fase 2 — Customer Service: estrutura inicial

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
- [ ] Remover arquivos desnecessários do repositório, se existirem
- [ ] Padronizar branch principal como `main`

---

## Fase 3 — Customer Service: domínio e primeiro caso de uso

- [ ] Criar enum `CustomerStatus`
- [ ] Criar domínio `Customer`
- [ ] Criar value object `Document`
- [ ] Criar value object `Email`
- [ ] Criar value object `Income`
- [ ] Criar exception `CustomerAlreadyExistsException`
- [ ] Criar exception `CustomerNotFoundException`
- [ ] Criar porta de entrada `CreateCustomerPort`
- [ ] Criar porta de saída `CustomerRepositoryPort`
- [ ] Criar caso de uso `CreateCustomerUseCase`
- [ ] Criar DTO `CreateCustomerRequest`
- [ ] Criar DTO `CustomerResponse`
- [ ] Criar mapper de request para domínio
- [ ] Criar mapper de domínio para response
- [ ] Criar controller `CustomerController`
- [ ] Criar endpoint `POST /api/v1/customers`
- [ ] Criar migration `V1__create_customers_table.sql`
- [ ] Criar entity `CustomerEntity`
- [ ] Criar repository Spring Data
- [ ] Criar adapter de persistência
- [ ] Validar criação de cliente via curl/Postman
- [ ] Validar persistência no MySQL
- [ ] Criar commit da primeira feature funcional

---

## Fase 4 — Customer Service: consulta e atualização

- [ ] Criar porta de entrada `FindCustomerByIdPort`
- [ ] Criar caso de uso `FindCustomerByIdUseCase`
- [ ] Criar endpoint `GET /api/v1/customers/{id}`
- [ ] Validar consulta por ID
- [ ] Criar porta de entrada `UpdateCustomerPort`
- [ ] Criar caso de uso `UpdateCustomerUseCase`
- [ ] Criar DTO `UpdateCustomerRequest`
- [ ] Criar endpoint `PUT /api/v1/customers/{id}`
- [ ] Validar atualização de cliente
- [ ] Criar porta de entrada `ChangeCustomerStatusPort`
- [ ] Criar caso de uso `ChangeCustomerStatusUseCase`
- [ ] Criar DTO `ChangeCustomerStatusRequest`
- [ ] Criar endpoint `PATCH /api/v1/customers/{id}/status`
- [ ] Criar migration `V2__create_customer_status_history_table.sql`
- [ ] Registrar histórico de alteração de status
- [ ] Validar alteração de status
- [ ] Validar histórico no banco

---

## Fase 5 — Infraestrutura local compartilhada

- [ ] Criar repositório `credit-platform-infra`
- [ ] Criar README do repositório de infra
- [ ] Criar Docker Compose principal da plataforma
- [ ] Adicionar MySQL para `customer_db`
- [ ] Adicionar MySQL para `rules_db`
- [ ] Adicionar MySQL para `limit_db`
- [ ] Adicionar MySQL para `communication_db`
- [ ] Adicionar MySQL para `audit_db`
- [ ] Adicionar Kafka
- [ ] Adicionar RabbitMQ
- [ ] Adicionar RabbitMQ Management
- [ ] Adicionar Prometheus
- [ ] Adicionar Grafana
- [ ] Adicionar OpenTelemetry Collector
- [ ] Criar rede Docker compartilhada
- [ ] Validar subida completa da infraestrutura
- [ ] Documentar portas usadas localmente
- [ ] Documentar comandos para subir, parar e limpar ambiente

---

## Fase 6 — Contratos compartilhados

- [ ] Criar repositório `credit-shared-contracts`
- [ ] Criar README do repositório de contratos
- [ ] Criar pasta `openapi`
- [ ] Criar pasta `asyncapi`
- [ ] Criar pasta `schemas`
- [ ] Documentar OpenAPI inicial do `credit-customer-service`
- [ ] Criar schema JSON do evento `CustomerCreated`
- [ ] Criar schema JSON do evento `CustomerUpdated`
- [ ] Criar schema JSON do evento `CustomerStatusChanged`
- [ ] Definir padrão comum de envelope de evento
- [ ] Definir campos obrigatórios: `eventId`, `eventType`, `eventVersion`, `source`, `correlationId`, `occurredAt`, `payload`
- [ ] Documentar estratégia de versionamento de eventos

---

## Fase 7 — Customer Service: eventos e Outbox Pattern

- [ ] Criar domínio `DomainEvent`
- [ ] Criar evento `CustomerCreatedEvent`
- [ ] Criar evento `CustomerUpdatedEvent`
- [ ] Criar evento `CustomerStatusChangedEvent`
- [ ] Criar porta de saída `CustomerEventPublisherPort`
- [ ] Criar migration `V3__create_outbox_events_table.sql`
- [ ] Criar entity `OutboxEventEntity`
- [ ] Criar repository `OutboxEventRepository`
- [ ] Criar adapter de outbox
- [ ] Salvar evento `CustomerCreated` na outbox ao criar cliente
- [ ] Criar scheduler/publicador de outbox
- [ ] Configurar Kafka producer
- [ ] Publicar evento `CustomerCreated` no tópico `customer.events`
- [ ] Marcar evento como publicado
- [ ] Validar evento salvo na outbox
- [ ] Validar evento publicado no Kafka
- [ ] Validar idempotência básica do publicador

---

## Fase 8 — Rules Engine Service: estrutura inicial

- [ ] Criar repositório `credit-rules-engine-service`
- [ ] Criar projeto Kotlin com Spring Boot
- [ ] Configurar Java 21
- [ ] Configurar Gradle Kotlin DSL
- [ ] Criar estrutura hexagonal
- [ ] Configurar MySQL `rules_db`
- [ ] Configurar Flyway
- [ ] Configurar Actuator
- [ ] Criar README técnico do serviço
- [ ] Validar `/actuator/health`
- [ ] Validar conexão com banco
- [ ] Validar `flyway_schema_history`

---

## Fase 9 — Rules Engine Service: elegibilidade

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
- [ ] Criar adapter de persistência
- [ ] Criar endpoint `POST /api/v1/rules/evaluate`
- [ ] Criar endpoint `GET /api/v1/rules/customers/{customerId}/eligibility`
- [ ] Validar cliente aprovado
- [ ] Validar cliente rejeitado
- [ ] Persistir decisão de elegibilidade

---

## Fase 10 — Rules Engine Service: consumo de eventos

- [ ] Configurar Kafka consumer
- [ ] Consumir evento `CustomerCreated`
- [ ] Criar tabela `inbox_events`
- [ ] Implementar Inbox Pattern
- [ ] Ignorar evento duplicado
- [ ] Avaliar elegibilidade ao receber `CustomerCreated`
- [ ] Publicar evento `CustomerEligibilityApproved`
- [ ] Publicar evento `CustomerEligibilityRejected`
- [ ] Criar schemas dos eventos de elegibilidade
- [ ] Validar fluxo `CustomerCreated -> CustomerEligibilityApproved`
- [ ] Validar fluxo `CustomerCreated -> CustomerEligibilityRejected`

---

## Fase 11 — Limit Service: estrutura inicial

- [ ] Criar repositório `credit-limit-service`
- [ ] Criar projeto Kotlin com Spring Boot
- [ ] Configurar Java 21
- [ ] Criar estrutura hexagonal
- [ ] Configurar MySQL `limit_db`
- [ ] Configurar Flyway
- [ ] Configurar Actuator
- [ ] Criar README técnico do serviço
- [ ] Validar `/actuator/health`
- [ ] Validar conexão com banco
- [ ] Validar `flyway_schema_history`

---

## Fase 12 — Limit Service: cálculo de limite

- [ ] Criar domínio `CreditLimit`
- [ ] Criar enum `LimitStatus`
- [ ] Criar value object `Money`
- [ ] Garantir uso de `BigDecimal` para valores monetários
- [ ] Criar porta de entrada `CalculateInitialLimitPort`
- [ ] Criar porta de saída `CreditLimitRepositoryPort`
- [ ] Criar caso de uso `CalculateInitialLimitUseCase`
- [ ] Criar migration `V1__create_credit_limits_table.sql`
- [ ] Criar migration `V2__create_limit_change_history_table.sql`
- [ ] Criar adapter de persistência
- [ ] Criar endpoint `GET /api/v1/limits/customers/{customerId}`
- [ ] Criar endpoint `POST /api/v1/limits/calculate`
- [ ] Validar cálculo de limite inicial
- [ ] Validar persistência de limite
- [ ] Validar histórico de alteração

---

## Fase 13 — Limit Service: consumo e publicação de eventos

- [ ] Configurar Kafka consumer
- [ ] Consumir evento `CustomerEligibilityApproved`
- [ ] Implementar Inbox Pattern
- [ ] Calcular limite automaticamente ao receber aprovação
- [ ] Criar evento `LimitCalculated`
- [ ] Criar evento `LimitUpdated`
- [ ] Criar evento `LimitBlocked`
- [ ] Criar evento `LimitReleased`
- [ ] Criar tabela `outbox_events`
- [ ] Implementar Outbox Pattern
- [ ] Publicar `LimitCalculated` no tópico `limit.events`
- [ ] Validar fluxo `CustomerEligibilityApproved -> LimitCalculated`

---

## Fase 14 — Limit Service: manutenção de limite

- [ ] Criar caso de uso `UpdateLimitUseCase`
- [ ] Criar caso de uso `BlockLimitUseCase`
- [ ] Criar caso de uso `ReleaseLimitUseCase`
- [ ] Criar endpoint `PATCH /api/v1/limits/{id}`
- [ ] Criar endpoint `PATCH /api/v1/limits/{id}/block`
- [ ] Criar endpoint `PATCH /api/v1/limits/{id}/release`
- [ ] Registrar histórico para cada alteração
- [ ] Publicar `LimitUpdated`
- [ ] Publicar `LimitBlocked`
- [ ] Publicar `LimitReleased`
- [ ] Validar bloqueio de limite
- [ ] Validar liberação de limite
- [ ] Validar atualização de limite

---

## Fase 15 — Communication Service: estrutura inicial

- [ ] Criar repositório `credit-communication-service`
- [ ] Criar projeto Kotlin com Spring Boot
- [ ] Configurar Java 21
- [ ] Criar estrutura hexagonal
- [ ] Configurar MySQL `communication_db`
- [ ] Configurar Flyway
- [ ] Configurar Actuator
- [ ] Configurar RabbitMQ
- [ ] Criar README técnico do serviço
- [ ] Validar `/actuator/health`
- [ ] Validar conexão com banco
- [ ] Validar conexão com RabbitMQ

---

## Fase 16 — Communication Service: envio de comunicação

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

---

## Fase 17 — Communication Service: RabbitMQ, retry e DLQ

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

## Fase 18 — Communication Service: eventos de domínio

- [ ] Consumir evento `LimitCalculated`
- [ ] Consumir evento `LimitUpdated`
- [ ] Consumir evento `CustomerEligibilityRejected`
- [ ] Criar evento `CommunicationRequested`
- [ ] Criar evento `CommunicationSent`
- [ ] Criar evento `CommunicationFailed`
- [ ] Criar evento `CommunicationDiscarded`
- [ ] Publicar eventos no tópico `communication.events`
- [ ] Validar fluxo `LimitCalculated -> CommunicationSent`
- [ ] Validar fluxo `CustomerEligibilityRejected -> CommunicationSent`
- [ ] Validar evento de falha de comunicação

---

## Fase 19 — Audit Service: estrutura inicial

- [ ] Criar repositório `credit-audit-service`
- [ ] Criar projeto Kotlin com Spring Boot
- [ ] Configurar Java 21
- [ ] Criar estrutura hexagonal
- [ ] Configurar MySQL `audit_db`
- [ ] Configurar Flyway
- [ ] Configurar Actuator
- [ ] Criar README técnico do serviço
- [ ] Validar `/actuator/health`
- [ ] Validar conexão com banco
- [ ] Validar `flyway_schema_history`

---

## Fase 20 — Audit Service: timeline e eventos

- [ ] Criar domínio `AuditEvent`
- [ ] Criar domínio `CustomerTimeline`
- [ ] Criar porta de entrada `RegisterAuditEventPort`
- [ ] Criar porta de saída `AuditEventRepositoryPort`
- [ ] Criar caso de uso `RegisterAuditEventUseCase`
- [ ] Criar migration `V1__create_audit_events_table.sql`
- [ ] Criar migration `V2__create_customer_timeline_table.sql`
- [ ] Criar migration `V3__create_event_payloads_table.sql`
- [ ] Consumir `CustomerCreated`
- [ ] Consumir `CustomerStatusChanged`
- [ ] Consumir `CustomerEligibilityApproved`
- [ ] Consumir `CustomerEligibilityRejected`
- [ ] Consumir `LimitCalculated`
- [ ] Consumir `LimitUpdated`
- [ ] Consumir `CommunicationSent`
- [ ] Consumir `CommunicationFailed`
- [ ] Implementar idempotência por `eventId`
- [ ] Persistir payload bruto do evento
- [ ] Persistir visão de timeline
- [ ] Criar endpoint `GET /api/v1/audits/customers/{customerId}/timeline`
- [ ] Criar endpoint `GET /api/v1/audits/events/{eventId}`
- [ ] Criar endpoint `GET /api/v1/audits`
- [ ] Validar timeline completa de um cliente

---

## Fase 21 — Config Server e feature toggles

- [ ] Criar repositório `credit-config-server`
- [ ] Criar projeto Spring Cloud Config Server
- [ ] Configurar porta `8888`
- [ ] Criar README técnico do serviço
- [ ] Criar configs locais do `credit-customer-service`
- [ ] Criar configs locais do `credit-rules-engine-service`
- [ ] Criar configs locais do `credit-limit-service`
- [ ] Criar configs locais do `credit-communication-service`
- [ ] Criar configs locais do `credit-audit-service`
- [ ] Configurar serviços para buscar configs no Config Server
- [ ] Criar toggle `enable-risk-rule-v2`
- [ ] Criar toggle `enable-async-communication`
- [ ] Criar toggle `enable-limit-auto-approval`
- [ ] Usar toggle no Rules Engine
- [ ] Usar toggle no Communication Service
- [ ] Validar mudança de comportamento por configuração

---

## Fase 22 — Fluxo ponta a ponta da jornada de crédito

- [ ] Subir infraestrutura completa
- [ ] Subir `credit-customer-service`
- [ ] Subir `credit-rules-engine-service`
- [ ] Subir `credit-limit-service`
- [ ] Subir `credit-communication-service`
- [ ] Subir `credit-audit-service`
- [ ] Criar cliente via API
- [ ] Validar evento `CustomerCreated`
- [ ] Validar decisão de elegibilidade
- [ ] Validar cálculo de limite
- [ ] Validar envio de comunicação
- [ ] Validar timeline no Audit Service
- [ ] Validar correlação por `correlationId`
- [ ] Documentar fluxo ponta a ponta no README principal

---

## Fase 23 — Observabilidade

- [ ] Configurar logs estruturados em JSON
- [ ] Adicionar `correlationId` em requests HTTP
- [ ] Propagar `correlationId` em eventos Kafka
- [ ] Propagar `correlationId` em mensagens RabbitMQ
- [ ] Adicionar `traceId` e `spanId`
- [ ] Configurar OpenTelemetry
- [ ] Configurar endpoint `/actuator/prometheus`
- [ ] Configurar Prometheus para coletar métricas dos serviços
- [ ] Configurar Grafana
- [ ] Criar dashboard geral da plataforma
- [ ] Criar dashboard de HTTP requests
- [ ] Criar dashboard de Kafka consumers/producers
- [ ] Criar dashboard de RabbitMQ
- [ ] Criar dashboard de DLQ
- [ ] Criar dashboard de jornada de crédito
- [ ] Criar métrica de clientes criados
- [ ] Criar métrica de clientes aprovados
- [ ] Criar métrica de clientes rejeitados
- [ ] Criar métrica de limites calculados
- [ ] Criar métrica de comunicações enviadas
- [ ] Criar métrica de mensagens em DLQ
- [ ] Validar investigação por `correlationId`

---

## Fase 24 — Qualidade e testes

- [ ] Criar testes unitários do domínio no Customer Service
- [ ] Criar testes unitários dos use cases no Customer Service
- [ ] Criar testes de controller no Customer Service
- [ ] Criar testes de persistência no Customer Service
- [ ] Criar testes unitários do Rules Engine
- [ ] Criar testes unitários do Limit Service
- [ ] Criar testes unitários do Communication Service
- [ ] Criar testes unitários do Audit Service
- [ ] Criar testes de integração com MySQL
- [ ] Criar testes de integração com Kafka
- [ ] Criar testes de integração com RabbitMQ
- [ ] Criar testes de DLQ
- [ ] Criar testes de Outbox Pattern
- [ ] Criar testes de Inbox Pattern
- [ ] Criar testes de arquitetura com ArchUnit
- [ ] Bloquear uso de field injection
- [ ] Bloquear uso de DTO no domínio
- [ ] Bloquear dependência de Spring no domínio
- [ ] Bloquear uso de Double para valores monetários

---

## Fase 25 — Segurança e validações

- [ ] Adicionar validações com Bean Validation
- [ ] Padronizar responses de erro
- [ ] Criar handler global de exceptions
- [ ] Criar padrão de erro com `code`, `message`, `details`, `timestamp`
- [ ] Mascarar dados sensíveis em logs
- [ ] Evitar log de documento completo
- [ ] Evitar log de telefone completo
- [ ] Evitar log de e-mail completo quando necessário
- [ ] Preparar estrutura para autenticação futura
- [ ] Documentar decisões de segurança

---

## Fase 26 — Kubernetes

- [ ] Criar Dockerfile do `credit-customer-service`
- [ ] Criar Dockerfile do `credit-rules-engine-service`
- [ ] Criar Dockerfile do `credit-limit-service`
- [ ] Criar Dockerfile do `credit-communication-service`
- [ ] Criar Dockerfile do `credit-audit-service`
- [ ] Criar manifests Kubernetes do Customer Service
- [ ] Criar manifests Kubernetes do Rules Engine
- [ ] Criar manifests Kubernetes do Limit Service
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

## Fase 27 — CI/CD

- [ ] Criar GitHub Actions para build do Customer Service
- [ ] Criar GitHub Actions para build do Rules Engine
- [ ] Criar GitHub Actions para build do Limit Service
- [ ] Criar GitHub Actions para build do Communication Service
- [ ] Criar GitHub Actions para build do Audit Service
- [ ] Rodar testes no pipeline
- [ ] Rodar análise estática no pipeline
- [ ] Gerar imagem Docker no pipeline
- [ ] Versionar imagem com commit SHA
- [ ] Documentar estratégia de pipeline
- [ ] Adicionar badge de build nos READMEs

---

## Fase 28 — Documentação final para portfólio

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
- [ ] Adicionar prints do Grafana
- [ ] Adicionar prints do RabbitMQ
- [ ] Adicionar prints do fluxo funcionando
- [ ] Criar resumo técnico para LinkedIn
- [ ] Criar roteiro de explicação para entrevista
- [ ] Criar seção “Decisões arquiteturais”
- [ ] Criar seção “Problemas reais simulados”
- [ ] Criar seção “Melhorias futuras”

---

## Fase 29 — Validação final

- [ ] Rodar todos os serviços localmente
- [ ] Criar cliente via API
- [ ] Validar banco do Customer Service
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
- [ ] Validar métricas no Prometheus
- [ ] Validar dashboards no Grafana
- [ ] Validar documentação
- [ ] Validar todos os READMEs
- [ ] Validar GitHub organizado
- [ ] Criar release inicial da plataforma

---

## Ordem prática recomendada

1. Finalizar `credit-customer-service`
2. Criar `credit-platform-infra`
3. Criar `credit-shared-contracts`
4. Implementar eventos e outbox no Customer Service
5. Criar `credit-rules-engine-service`
6. Criar `credit-limit-service`
7. Criar `credit-communication-service`
8. Criar `credit-audit-service`
9. Fechar fluxo ponta a ponta
10. Adicionar observabilidade
11. Adicionar Config Server e toggles
12. Adicionar testes e ArchUnit
13. Adicionar Kubernetes
14. Finalizar documentação de portfólio

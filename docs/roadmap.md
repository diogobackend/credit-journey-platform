# Roadmap

Este roadmap organiza o desenvolvimento da Credit Journey Platform em fases.

O objetivo é construir o portfólio de forma incremental, entregando uma task por vez e mantendo qualidade arquitetural.

## Fase 1 — Fundação da plataforma

Objetivo:

Criar os repositórios, documentação inicial e visão arquitetural.

Tasks:

- Criar repositório credit-journey-platform.
- Criar documentação base.
- Criar repositório credit-platform-infra.
- Criar repositório credit-shared-contracts.
- Definir visão da arquitetura.
- Definir serviços da plataforma.
- Definir estratégia de banco.
- Definir estratégia de mensageria.
- Definir estratégia de observabilidade.

Status esperado:

- Documentação inicial pronta.
- Decisões técnicas registradas.
- Plataforma desenhada.

## Fase 2 — Customer Service

Objetivo:

Criar o primeiro microservice com arquitetura hexagonal.

Tasks:

- Criar repositório credit-customer-service.
- Criar projeto Spring Boot com Java 21.
- Criar estrutura hexagonal.
- Criar domínio Customer.
- Criar porta de entrada CreateCustomerUseCase.
- Criar porta de saída CustomerRepositoryPort.
- Criar adapter REST.
- Criar adapter de persistência.
- Configurar MySQL.
- Configurar Flyway.
- Criar migration de customers.
- Criar endpoint POST /api/v1/customers.
- Criar endpoint GET /api/v1/customers/{id}.
- Publicar evento CustomerCreated.
- Criar README técnico do serviço.

Status esperado:

- Serviço criando e consultando clientes.
- Banco versionado com Flyway.
- Estrutura hexagonal funcionando.

## Fase 3 — Rules Engine Service

Objetivo:

Criar serviço de regras de elegibilidade.

Tasks:

- Criar repositório credit-rules-engine-service.
- Criar projeto Spring Boot com Java 21.
- Criar estrutura hexagonal.
- Criar domínio EligibilityDecision.
- Consumir evento CustomerCreated.
- Criar regras iniciais de elegibilidade.
- Criar feature toggles simples.
- Publicar CustomerEligibilityApproved.
- Publicar CustomerEligibilityRejected.
- Persistir histórico de decisão.
- Criar endpoint de consulta de elegibilidade.

Status esperado:

- Serviço avaliando clientes.
- Decisão persistida.
- Evento de aprovação/rejeição publicado.

## Fase 4 — Limit Service

Objetivo:

Criar serviço de limite de crédito.

Tasks:

- Criar repositório credit-limit-service.
- Criar projeto Spring Boot com Java 21.
- Criar estrutura hexagonal.
- Criar domínio CreditLimit.
- Consumir CustomerEligibilityApproved.
- Calcular limite inicial.
- Persistir limite.
- Criar histórico de alteração.
- Publicar LimitCalculated.
- Implementar Outbox Pattern.
- Implementar Inbox Pattern.
- Criar endpoint de consulta de limite.

Status esperado:

- Serviço calculando limite.
- Evento publicado com segurança.
- Idempotência inicial implementada.

## Fase 5 — Communication Service

Objetivo:

Criar serviço de comunicação com RabbitMQ, retry e DLQ.

Tasks:

- Criar repositório credit-communication-service.
- Criar projeto Spring Boot com Java 21.
- Criar estrutura hexagonal.
- Criar domínio CommunicationRequest.
- Consumir eventos de limite.
- Criar fila communication.send.queue.
- Criar fila communication.send.retry.queue.
- Criar fila communication.send.dlq.
- Implementar envio fake.
- Implementar retry.
- Implementar DLQ.
- Registrar histórico de envio.
- Publicar CommunicationSent.
- Publicar CommunicationFailed.

Status esperado:

- Serviço processando comunicação.
- Retry funcionando.
- DLQ demonstrável.

## Fase 6 — Audit Service

Objetivo:

Criar serviço de auditoria e timeline.

Tasks:

- Criar repositório credit-audit-service.
- Criar projeto Spring Boot com Java 21.
- Criar estrutura hexagonal.
- Consumir eventos dos serviços.
- Persistir audit_events.
- Persistir customer_timeline.
- Criar endpoint de timeline por cliente.
- Criar busca por eventId.
- Garantir idempotência por eventId.

Status esperado:

- Timeline da jornada funcionando.
- Eventos auditados.
- Consulta disponível por cliente.

## Fase 7 — Config Server

Objetivo:

Criar configuração centralizada.

Tasks:

- Criar repositório credit-config-server.
- Criar Spring Cloud Config Server.
- Criar configs por serviço.
- Criar configs por ambiente.
- Configurar serviços para buscar configs.
- Adicionar feature toggles simples.
- Documentar propriedades principais.

Status esperado:

- Serviços lendo configuração externa.
- Toggles básicos disponíveis.

## Fase 8 — Infraestrutura local

Objetivo:

Criar ambiente local completo.

Tasks:

- Criar docker-compose.yml.
- Subir MySQL.
- Subir Kafka.
- Subir RabbitMQ.
- Subir Prometheus.
- Subir Grafana.
- Subir OpenTelemetry Collector.
- Configurar redes.
- Configurar volumes.
- Criar scripts de inicialização.

Status esperado:

- Ambiente local funcionando com um comando.
- Infra pronta para rodar os serviços.

## Fase 9 — Observabilidade

Objetivo:

Adicionar logs, métricas e tracing.

Tasks:

- Configurar Actuator.
- Expor endpoint Prometheus.
- Configurar logs estruturados.
- Adicionar correlationId.
- Propagar correlationId em eventos.
- Configurar OpenTelemetry.
- Criar dashboard Grafana.
- Criar métricas de negócio.
- Criar métricas de mensageria.

Status esperado:

- Serviços observáveis.
- Logs rastreáveis.
- Métricas disponíveis no Grafana.

## Fase 10 — Kubernetes

Objetivo:

Preparar deploy em Kubernetes.

Tasks:

- Criar Dockerfile para cada serviço.
- Criar Deployment.
- Criar Service.
- Criar ConfigMap.
- Criar Secret.
- Criar Ingress.
- Criar readinessProbe.
- Criar livenessProbe.
- Criar requests e limits.
- Criar HPA.
- Documentar deploy.

Status esperado:

- Manifests Kubernetes criados.
- Serviços preparados para deploy.

## Fase 11 — Testes

Objetivo:

Adicionar testes unitários, integração e arquitetura.

Tasks:

- Testes unitários de domínio.
- Testes de use cases.
- Testes de mappers.
- Testes de controllers.
- Testes de consumers.
- Testes de integração com banco.
- Testes de integração com mensageria.
- Testes de arquitetura com ArchUnit.

Status esperado:

- Cobertura relevante.
- Arquitetura protegida.
- Fluxos principais testados.

## Fase 12 — Documentação final

Objetivo:

Preparar apresentação do portfólio.

Tasks:

- Atualizar README principal.
- Criar diagramas.
- Documentar fluxo principal.
- Documentar decisões técnicas.
- Documentar como rodar localmente.
- Documentar exemplos de payload.
- Documentar troubleshooting.
- Criar resumo para LinkedIn.
- Criar roteiro para entrevista.

Status esperado:

- Projeto apresentável.
- Documentação clara.
- Narrativa técnica pronta.
# Arquitetura

Este documento descreve a arquitetura da **Credit Journey Platform**, uma plataforma backend baseada em microservices para simular a jornada de crédito de um cliente em um banco digital.

O foco desta documentação é explicar as decisões arquiteturais, a separação de responsabilidades, os padrões adotados e como os serviços devem se comunicar.

---

## Visão geral

A plataforma é composta por microservices independentes, cada um responsável por um contexto específico da jornada de crédito.

```text
Cliente / Frontend
      |
      v
credit-api-gateway
      |
      v
credit-orchestrator-service
      |
      |--> credit-customer-service
      |--> credit-rules-engine-service
      |--> credit-limit-service
      |--> credit-communication-service
      |--> credit-audit-service
```

Serviços de apoio:

```text
credit-auth-service
credit-config-server
credit-observability-starter
credit-shared-contracts
credit-platform-infra
```

---

## Estilo arquitetural

A arquitetura adota os seguintes estilos e padrões:

- Microservices;
- Hexagonal Architecture / Ports and Adapters;
- Event-driven architecture;
- Database per service;
- API Gateway;
- Orchestration Service;
- Auth Service com JWT;
- Outbox Pattern;
- Inbox Pattern;
- Observabilidade distribuída;
- Lib compartilhada para padrões técnicos.

---

## Princípio principal

O domínio não deve depender de detalhes externos.

O núcleo da aplicação não deve conhecer:

- Spring;
- JPA;
- Banco de dados;
- Kafka;
- RabbitMQ;
- HTTP;
- WebClient;
- DTOs;
- Entities;
- Frameworks;
- Bibliotecas de infraestrutura.

O domínio deve conter apenas regras de negócio.

---

## Responsabilidade dos componentes principais

### credit-api-gateway

Responsável pela entrada técnica da plataforma.

Responsabilidades:

- receber requisições externas;
- validar JWT;
- aplicar filtros técnicos;
- controlar CORS;
- aplicar rate limit;
- propagar correlationId;
- rotear chamadas para os serviços internos.

O gateway não deve conter regra de negócio.

Exemplo:

```text
GET /api/v1/customers/{customerId}
```

é roteado para:

```text
credit-customer-service
```

---

### credit-orchestrator-service

Responsável por coordenar a jornada de crédito.

Ele chama os serviços especialistas na ordem correta.

Exemplo de fluxo:

```text
1. recebe solicitação de jornada de crédito
2. consulta dados do cliente
3. solicita avaliação de elegibilidade
4. solicita cálculo de limite
5. solicita comunicação
6. registra auditoria
```

O orchestrator não deve assumir responsabilidade interna dos serviços especialistas.

Ele apenas coordena o fluxo de negócio.

---

### credit-auth-service

Responsável por autenticação e autorização.

Responsabilidades:

- login;
- validação de senha;
- geração de JWT;
- geração de refresh token;
- controle de roles;
- controle de permissões.

Fluxo básico:

```text
1. usuário faz login
2. auth-service gera JWT
3. cliente envia JWT nas próximas chamadas
4. gateway valida o token
5. requisição segue para o serviço interno
```

---

### credit-customer-service

Responsável pelo cadastro e manutenção dos clientes.

Responsabilidades:

- criar cliente;
- consultar cliente;
- listar clientes;
- atualizar dados cadastrais;
- alterar status;
- excluir cliente;
- publicar eventos de cliente.

Eventos planejados:

```text
CustomerCreated
CustomerUpdated
CustomerStatusChanged
CustomerDeleted
```

---

### credit-rules-engine-service

Responsável por avaliar regras de elegibilidade de crédito.

Responsabilidades:

- validar status do cliente;
- aplicar regras de risco;
- avaliar renda mínima;
- avaliar score interno;
- aplicar feature toggles de regras.

Resultados possíveis:

```text
APPROVED
REJECTED
MANUAL_ANALYSIS
```

Eventos planejados:

```text
CustomerEligibilityApproved
CustomerEligibilityRejected
CustomerManualAnalysisRequested
```

---

### credit-limit-service

Responsável pelo cálculo e manutenção do limite de crédito.

Responsabilidades:

- calcular limite inicial;
- consultar limite total;
- consultar limite disponível;
- bloquear limite;
- liberar limite;
- atualizar limite;
- registrar histórico de alterações.

Eventos planejados:

```text
LimitCalculated
LimitUpdated
LimitBlocked
LimitReleased
```

---

### credit-communication-service

Responsável por comunicações com o cliente.

Responsabilidades:

- enviar comunicação de limite aprovado;
- enviar comunicação de limite alterado;
- enviar comunicação de rejeição;
- controlar retry;
- tratar DLQ;
- registrar falhas de envio.

Esse serviço é um bom candidato para uso de RabbitMQ.

---

### credit-audit-service

Responsável por registrar a timeline da jornada.

Responsabilidades:

- registrar eventos da jornada;
- armazenar correlationId;
- permitir consulta histórica;
- apoiar rastreabilidade operacional.

Exemplo de timeline:

```text
10:00 - CustomerCreated
10:01 - CustomerEligibilityApproved
10:02 - LimitCalculated
10:03 - CommunicationSent
```

---

## Bibliotecas e repositórios de apoio

### credit-observability-starter

Lib compartilhada responsável por padronizar observabilidade entre os serviços.

Não é um microservice.

Responsabilidades:

- fornecer annotation `@LogInfo`;
- fornecer annotation `@LogParameter`;
- aplicar logs automáticos com AOP;
- padronizar correlationId;
- configurar MDC;
- mascarar dados sensíveis;
- padronizar campos comuns de log;
- facilitar integração com tracing.

Exemplo de uso:

```kotlin
@LogInfo(logParameters = true, logReturn = true)
fun create(input: CreateCustomerInput): Customer
```

Campos esperados no log:

```text
serviceName
operation
method
parameters
return
correlationId
traceId
spanId
duration
status
```

---

### credit-config-server

Responsável por centralizar configurações externas dos serviços.

Exemplos:

- URLs de serviços;
- configurações de mensageria;
- feature toggles;
- parâmetros técnicos;
- configurações por ambiente.

---

### credit-shared-contracts

Responsável por centralizar contratos compartilhados.

Exemplos:

- OpenAPI;
- AsyncAPI;
- schemas de eventos;
- exemplos de payloads;
- documentação de contratos entre serviços.

---

### credit-platform-infra

Responsável pela infraestrutura local e futura preparação para Kubernetes.

Exemplos:

- Docker Compose;
- Kafka;
- RabbitMQ;
- MySQL;
- Prometheus;
- Grafana;
- Jaeger/OpenTelemetry;
- Kubernetes manifests.

---

## Estrutura padrão dos serviços

Cada microservice deve seguir uma estrutura baseada em Arquitetura Hexagonal.

```text
src/main/kotlin/com/creditjourney/{service}/
├── core/
│   ├── common/
│   ├── domain/
│   │   ├── model/
│   │   ├── exception/
│   │   └── valueobject/
│   ├── port/
│   │   ├── input/
│   │   └── output/
│   └── usecase/
└── app/
    ├── adapter/
    │   ├── input/
    │   │   ├── web/
    │   │   └── messaging/
    │   └── output/
    │       ├── persistence/
    │       ├── messaging/
    │       └── client/
    └── configuration/
```

---

## Responsabilidade das camadas

### core/domain

Contém os conceitos e regras centrais do domínio.

Exemplos:

```text
Customer
CreditLimit
EligibilityDecision
CommunicationRequest
AuditEvent
```

Não deve depender de Spring, JPA, HTTP, Kafka ou RabbitMQ.

---

### core/domain/model

Contém os modelos principais do domínio.

Esses modelos representam conceitos de negócio, não tabelas de banco.

Exemplos:

```text
Customer
CreditLimit
EligibilityDecision
```

---

### core/domain/valueobject

Contém objetos de valor.

Exemplos:

```text
Document
Email
Income
Money
```

Value Objects devem concentrar validações próprias de valores importantes para o negócio.

---

### core/domain/exception

Contém exceptions específicas.

Exemplos:

```text
CustomerNotFoundException
CustomerAlreadyExistsException
InvalidCreditLimitException
EligibilityRejectedException
```

---

### core/port/input

Define o que a aplicação sabe fazer.

Exemplos:

```text
CreateCustomerPort
FindCustomerByIdPort
CalculateLimitPort
EvaluateEligibilityPort
SendCommunicationPort
```

Controllers e consumers devem chamar portas de entrada.

---

### core/port/output

Define dependências externas necessárias para executar os casos de uso.

Exemplos:

```text
CustomerRepositoryPort
EventPublisherPort
LimitRepositoryPort
CommunicationProviderPort
AuditEventRepositoryPort
```

Essas portas escondem detalhes de banco, mensageria, HTTP client ou provedores externos.

---

### core/usecase

Contém a implementação dos casos de uso.

Exemplos:

```text
CreateCustomerUseCase
UpdateCustomerUseCase
CalculateLimitUseCase
EvaluateEligibilityUseCase
```

Use cases podem chamar portas de saída, mas não devem conhecer adapters concretos.

---

### app/adapter/input/web

Contém controllers REST.

Responsabilidades:

- receber requisições HTTP;
- validar requests;
- converter request para input;
- chamar porta de entrada;
- converter domínio para response.

Controller não deve conter regra de negócio.

---

### app/adapter/input/messaging

Contém consumers Kafka ou RabbitMQ.

Responsabilidades:

- receber mensagens;
- converter payload para input;
- validar estrutura da mensagem;
- garantir idempotência quando necessário;
- chamar porta de entrada.

Consumer não deve conter regra de negócio.

---

### app/adapter/output/persistence

Contém persistência.

Inclui:

```text
Entity JPA
Repository Spring Data
Persistence Adapter
Mappers Entity <-> Domain
```

Entity JPA não deve ser usada como modelo de domínio.

---

### app/adapter/output/messaging

Contém producers Kafka ou RabbitMQ.

Responsável por publicar eventos ou comandos para outros serviços.

Exemplos:

```text
CustomerEventProducer
LimitEventProducer
CommunicationQueueProducer
```

---

### app/adapter/output/client

Contém clients HTTP para chamadas síncronas entre serviços.

Exemplos:

```text
CustomerClient
LimitClient
RulesEngineClient
```

Clients devem implementar portas de saída.

---

### app/configuration

Contém configurações Spring.

Exemplos:

- beans;
- configuração de banco;
- configuração de mensageria;
- configuração de segurança;
- configuração de WebClient;
- configuração de observabilidade;
- configuração dos use cases.

---

## Comunicação entre serviços

A plataforma usa três formas principais de comunicação:

```text
REST
Kafka
RabbitMQ
```

---

## REST

Usado para operações síncronas, quando a resposta imediata é necessária.

Exemplos:

```text
GET /api/v1/customers/{customerId}
GET /api/v1/limits/{customerId}
GET /api/v1/audits/{correlationId}
POST /api/v1/credit-journeys
POST /api/v1/auth/login
```

Casos comuns:

- consultar cliente;
- consultar limite;
- iniciar jornada de crédito;
- realizar login;
- consultar timeline.

---

## Kafka

Usado para eventos de domínio.

Eventos representam algo que já aconteceu.

Exemplos:

```text
CustomerCreated
CustomerUpdated
CustomerStatusChanged
CustomerDeleted
CustomerEligibilityApproved
CustomerEligibilityRejected
LimitCalculated
LimitUpdated
CommunicationSent
CommunicationFailed
```

Kafka será usado quando outros serviços precisarem reagir a acontecimentos da jornada.

---

## RabbitMQ

Usado para filas de trabalho, retry e DLQ.

Casos comuns:

- envio de comunicação;
- reprocessamento de mensagens;
- controle de tentativas;
- processamento assíncrono;
- mensagens que precisam de DLQ.

RabbitMQ será usado principalmente para trabalho assíncrono controlado.

---

## API Gateway vs Orchestrator

### API Gateway

Responsável por entrada técnica.

Faz:

```text
roteamento
validação de token
CORS
rate limit
filtros técnicos
propagação de headers
```

Não deve conter regra de negócio pesada.

---

### Orchestrator

Responsável por fluxo de negócio.

Faz:

```text
consulta customer-service
chama rules-engine-service
chama credit-limit-service
chama communication-service
chama audit-service
controla a ordem da jornada
```

---

### Diferença principal

```text
API Gateway = entrada técnica e roteamento
Orchestrator = coordenação de negócio
```

---

## Banco de dados

A plataforma utiliza `database per service`.

Cada serviço possui seu próprio banco.

| Serviço | Banco |
|---|---|
| `credit-auth-service` | `auth_db` |
| `credit-customer-service` | `customer_db` |
| `credit-rules-engine-service` | `rules_db` |
| `credit-limit-service` | `limit_db` |
| `credit-communication-service` | `communication_db` |
| `credit-audit-service` | `audit_db` |

Regras:

- um serviço não acessa diretamente o banco de outro serviço;
- integração entre serviços acontece via API ou mensageria;
- migrations devem ser versionadas com Flyway;
- alterações de schema devem ser rastreáveis.

---

## Outbox Pattern

Usado para garantir consistência entre alteração no banco e publicação de eventos.

Fluxo:

```text
1. caso de uso processa a regra
2. estado é salvo no banco
3. evento é salvo na tabela outbox
4. publicador assíncrono envia o evento
5. evento é marcado como publicado
```

Esse padrão evita perda de evento após uma alteração de banco bem-sucedida.

---

## Inbox Pattern

Usado para evitar processamento duplicado de mensagens recebidas.

Fluxo:

```text
1. consumer recebe uma mensagem
2. verifica se o eventId já foi processado
3. se já foi processado, ignora
4. se não foi, processa
5. registra o eventId na tabela inbox
```

Esse padrão ajuda a garantir idempotência.

---

## Feature Toggle

Usado para ligar ou desligar comportamentos sem novo deploy.

Exemplos:

```text
enable-risk-rule-v2
enable-async-communication
enable-limit-auto-approval
enable-new-limit-calculation
```

---

## Correlation ID

Usado para rastrear uma jornada entre múltiplos serviços.

Exemplo:

```text
correlationId=8b70f8d1-14e2-45cf-a183-f2196ef4b821
```

O correlationId deve ser propagado entre:

- API Gateway;
- Orchestrator;
- Microservices;
- Eventos Kafka;
- Mensagens RabbitMQ;
- Logs;
- Auditoria.

---

## Observabilidade

A arquitetura deve permitir rastreabilidade distribuída.

A plataforma deve possuir:

- logs estruturados;
- métricas;
- tracing;
- dashboards;
- correlationId;
- health checks.

Parte da padronização técnica será centralizada no `credit-observability-starter`.

---

## Logs estruturados

Campos esperados:

```text
serviceName
operation
correlationId
traceId
spanId
customerId
eventId
status
duration
error
```

---

## Métricas planejadas

Exemplos:

```text
requests_total
request_duration_seconds
errors_total
messages_consumed_total
messages_failed_total
messages_dlq_total
customers_created_total
eligibility_approved_total
eligibility_rejected_total
limits_calculated_total
communications_sent_total
auth_login_success_total
auth_login_failed_total
```

---

## Dashboards planejados

O Grafana será usado para visualizar:

- saúde dos serviços;
- taxa de erro;
- latência;
- throughput;
- consumo de mensagens;
- backlog de filas;
- mensagens em DLQ;
- falhas de autenticação;
- jornadas aprovadas;
- jornadas recusadas.

---

## Regras arquiteturais

- Controller não acessa repository diretamente.
- Consumer não executa regra de negócio diretamente.
- Repository não conhece use case.
- DTO não entra no domínio.
- Entity JPA não deve ser usada como modelo de domínio.
- Domínio não conhece Spring.
- Toda entrada passa por uma porta de entrada.
- Toda saída passa por uma porta de saída.
- Exceptions devem ser específicas.
- Injeção de dependência deve ser por construtor.
- Valores monetários devem usar BigDecimal.
- Mensagens assíncronas devem ser idempotentes.
- Eventos devem carregar `eventId`.
- Requisições devem propagar `correlationId`.
- Serviços não devem acessar banco de outro serviço.
- Gateway não deve conter regra de negócio.
- Orchestrator não deve assumir responsabilidade interna dos serviços especialistas.
- Lib compartilhada não deve conter regra de negócio de domínio.

---

## Objetivo da arquitetura

A arquitetura deve permitir:

- evolução independente dos serviços;
- baixo acoplamento;
- alta rastreabilidade;
- facilidade de testes;
- clareza na separação de responsabilidades;
- autenticação centralizada;
- entrada única via gateway;
- coordenação de jornadas via orchestrator;
- reutilização técnica via libs compartilhadas;
- simulação realista de produção.
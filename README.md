# Credit Journey Platform

Plataforma backend fictícia que simula a jornada de crédito de um cliente em um banco digital.

O projeto representa uma arquitetura de microservices usada por um banco emissor de cartão para cadastrar clientes, autenticar usuários, avaliar elegibilidade de crédito, calcular limite, comunicar o cliente e registrar toda a jornada para auditoria.

---

## 1 - Repositórios

| Repositório | Responsabilidade |
|---|---|
| [credit-customer-service](https://github.com/diogobackend/credit-customer-service) | Cadastro, consulta, edição, status e exclusão de clientes |
| [credit-limit-service](https://github.com/diogobackend/credit-limit-service) | Cálculo e manutenção de limite |
| `credit-rules-engine-service` | Regras de elegibilidade de crédito |
| `credit-orchestrator-service` | Orquestração da jornada de crédito |
| `credit-auth-service` | Login, autenticação, JWT e roles |
| `credit-api-gateway` | Entrada única, roteamento e validação de token |
| `credit-communication-service` | Comunicação com cliente, retry e DLQ |
| `credit-audit-service` | Auditoria e timeline da jornada |
| `credit-config-server` | Configuração centralizada |
| `credit-platform-infra` | Infraestrutura local e Kubernetes |
| `credit-shared-contracts` | Contratos de APIs e eventos |
| `credit-observability-starter` | Lib compartilhada de observabilidade |

## 2 - O que este produto resolve?

Em um banco digital, a concessão de crédito não acontece em uma única etapa.

Antes de um cliente receber limite no cartão, o sistema precisa:

- autenticar o usuário;
- cadastrar e manter os dados do cliente;
- avaliar se o cliente está elegível para crédito;
- aplicar regras de risco e políticas internas;
- calcular o limite disponível;
- comunicar o cliente sobre aprovação, rejeição ou alteração de limite;
- registrar tudo para auditoria, rastreabilidade e suporte operacional.

A **Credit Journey Platform** simula esse fluxo usando microservices independentes, banco por serviço, comunicação síncrona e assíncrona, autenticação com JWT, observabilidade e padrões próximos de um ambiente real de produção.

---

## 3 - Exemplo prático

Imagine o seguinte cenário:

1. João faz login na plataforma.
2. O `credit-auth-service` autentica o usuário e gera um token JWT.
3. João acessa a plataforma pelo `credit-api-gateway`.
4. O `credit-orchestrator-service` inicia a jornada de crédito.
5. O [credit-customer-service](https://github.com/diogobackend/credit-customer-service) consulta os dados cadastrais do cliente.
6. O `credit-rules-engine-service` avalia se João pode receber crédito.
7. Se aprovado, o [credit-limit-service](https://github.com/diogobackend/credit-limit-service) calcula o limite inicial.
8. O `credit-communication-service` envia a comunicação ao cliente.
9. O `credit-audit-service` registra toda a jornada.

Exemplo de resultado:

```text
Cliente: Diogo Ferreira
Status: ACTIVE
Elegibilidade: APPROVED
Limite inicial: R$ 3.500,00
Comunicação: enviada
Auditoria: registrada
```

---

## 4 - Fluxo resumido

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

O `credit-auth-service` é responsável por autenticação e emissão de token.

O `credit-api-gateway` valida o token e roteia as requisições para os serviços internos.

O `credit-orchestrator-service` coordena a jornada de negócio entre os microservices.

---

## 5 - Objetivo técnico

Este projeto foi criado como portfólio backend para demonstrar boas práticas de desenvolvimento, arquitetura e operação de sistemas distribuídos.

Principais objetivos:

- aplicar Arquitetura Hexagonal / Ports and Adapters;
- separar domínio, aplicação e infraestrutura;
- construir microservices independentes;
- aplicar autenticação com JWT;
- proteger endpoints com Spring Security;
- usar API Gateway como entrada única da plataforma;
- usar Orchestrator para coordenar jornadas de negócio;
- usar comunicação síncrona e assíncrona;
- aplicar Kafka para eventos de domínio;
- aplicar RabbitMQ para filas de trabalho, retry e DLQ;
- versionar banco de dados com Flyway;
- usar MySQL com banco por serviço;
- implementar Outbox Pattern;
- aplicar Feature Toggles;
- centralizar configurações;
- expor métricas com Prometheus;
- criar dashboards no Grafana;
- gerar logs estruturados com correlationId;
- criar lib compartilhada para observabilidade;
- preparar os serviços para Docker e Kubernetes.

---

## 6 - Microservices

| Serviço | Responsabilidade |
|---|---|
| `credit-auth-service` | Gerencia login, autenticação, usuários, roles e geração de JWT |
| `credit-api-gateway` | Porta de entrada da plataforma, roteamento, validação de token e filtros técnicos |
| [credit-customer-service](https://github.com/diogobackend/credit-customer-service) | Gerencia cadastro, status e perfil do cliente |
| `credit-rules-engine-service` | Avalia elegibilidade e políticas de crédito |
| [credit-limit-service](https://github.com/diogobackend/credit-limit-service) | Calcula e mantém limites de crédito |
| `credit-orchestrator-service` | Coordena a jornada de crédito entre os microservices |
| `credit-communication-service` | Dispara comunicações para o cliente |
| `credit-audit-service` | Registra histórico, eventos e timeline da jornada |
| `credit-config-server` | Centraliza configurações dos serviços |

---

## 7 - Bibliotecas e repositórios de apoio

Além dos microservices, a plataforma também possui repositórios de apoio para infraestrutura, contratos e padronização técnica.

| Repositório | Responsabilidade |
|---|---|
| `credit-platform-infra` | Contém Docker Compose, Kubernetes, Prometheus, Grafana, Kafka, RabbitMQ e MySQL |
| `credit-shared-contracts` | Centraliza contratos OpenAPI, AsyncAPI e schemas de eventos |
| `credit-observability-starter` | Lib compartilhada para logs, correlationId, MDC, tracing e padronização de observabilidade |

O `credit-observability-starter` não é um microservice.

Ele é uma biblioteca compartilhada usada pelos serviços para evitar duplicação de código técnico.

---

## 8 - Responsabilidade dos serviços

### [credit-customer-service](https://github.com/diogobackend/credit-customer-service)

Responsável pelo cadastro e manutenção dos clientes.

Exemplos de responsabilidades:

- criar cliente;
- consultar cliente;
- listar clientes com paginação e filtros;
- atualizar dados cadastrais;
- alterar status;
- excluir cliente;
- publicar eventos como `CustomerCreated`, `CustomerUpdated`, `CustomerStatusChanged` e `CustomerDeleted`.

---

### [credit-limit-service](https://github.com/diogobackend/credit-limit-service)

Responsável pelo cálculo e manutenção do limite de crédito.

Exemplos de responsabilidades:

- calcular limite inicial;
- consultar limite total;
- consultar limite disponível;
- bloquear limite;
- liberar limite;
- atualizar limite;
- registrar histórico de alterações;
- publicar eventos como `LimitCalculated` e `LimitUpdated`.

---

### credit-rules-engine-service

Responsável por avaliar se um cliente está elegível para crédito.

Exemplos de regras:

- cliente bloqueado não pode receber limite;
- cliente inativo não pode seguir no fluxo;
- renda mínima pode ser exigida;
- score interno pode influenciar a aprovação;
- regras novas podem ser ativadas por feature toggle.

Exemplos de resultado:

```text
APPROVED
REJECTED
MANUAL_ANALYSIS
```

---

### credit-orchestrator-service

Responsável por coordenar a jornada de crédito entre os microservices.

Exemplo de fluxo:

```text
1. recebe solicitação de jornada de crédito
2. consulta dados do cliente
3. solicita avaliação de elegibilidade
4. solicita cálculo de limite
5. dispara comunicação
6. registra auditoria
```

Esse serviço concentra a ordem do fluxo de negócio, sem assumir responsabilidade interna de cada domínio.

Ele não substitui os serviços especialistas.

Ele apenas coordena a jornada.

---

### credit-auth-service

Responsável pela autenticação da plataforma.

Exemplos de responsabilidades:

- cadastrar usuário de acesso;
- realizar login;
- validar senha;
- gerar token JWT;
- gerar refresh token;
- controlar roles e permissões;
- expor dados básicos do usuário autenticado.

Exemplo de endpoint:

```text
POST /api/v1/auth/login
```

---

### credit-api-gateway

Responsável por ser a entrada única da plataforma.

Exemplos de responsabilidades:

- receber requisições externas;
- validar token JWT;
- aplicar filtros técnicos;
- controlar CORS;
- aplicar rate limit;
- rotear chamadas para os microservices internos.

Exemplo:

```text
GET /api/v1/customers/{customerId}
```

é encaminhado para:

```text
credit-customer-service
```

O gateway não deve concentrar regra de negócio pesada.

---

### credit-communication-service

Responsável por comunicar o cliente sobre eventos da jornada.

Exemplos de comunicação:

- limite aprovado;
- limite alterado;
- cliente não elegível;
- falha ou pendência no processamento.

Também demonstra uso de:

- RabbitMQ;
- retry;
- DLQ;
- controle de tentativas;
- mensagens assíncronas.

---

### credit-audit-service

Responsável por registrar uma visão auditável da jornada.

Exemplo de timeline:

```text
10:00 - CustomerCreated
10:01 - CustomerEligibilityApproved
10:02 - LimitCalculated
10:03 - CommunicationSent
```

Esse serviço permite consultar:

- o que aconteceu;
- quando aconteceu;
- qual serviço executou;
- qual foi o correlationId;
- qual foi o status da operação.

---

### credit-observability-starter

Lib compartilhada responsável por padronizar logs e observabilidade entre os serviços.

Exemplos de responsabilidades:

- fornecer annotation `@LogInfo`;
- fornecer annotation `@LogParameter`;
- aplicar logs automáticos com AOP;
- padronizar correlationId;
- configurar MDC;
- mascarar dados sensíveis nos logs;
- padronizar campos comuns de log;
- facilitar integração com tracing e observabilidade.

Essa lib evita duplicação de código técnico entre os microservices.

Exemplo de uso planejado:

```kotlin
@LogInfo(logParameters = true, logReturn = true)
fun create(input: CreateCustomerInput): Customer
```

---

## 9 - Arquitetura Hexagonal

Cada microservice segue Arquitetura Hexagonal, mantendo o domínio isolado de frameworks e infraestrutura.

Estrutura padrão:

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

Regras arquiteturais:

- controller não acessa repository diretamente;
- consumer não executa regra de negócio diretamente;
- repository não conhece use case;
- DTO não entra no domínio;
- entidade JPA não é modelo de domínio;
- domínio não depende de Spring;
- toda entrada passa por uma porta de entrada;
- toda saída passa por uma porta de saída.

---

## Comunicação entre serviços

A plataforma usa três formas principais de comunicação.

---

### REST

Usado para operações síncronas, principalmente consultas e comandos diretos.

Exemplos:

- consultar cliente;
- consultar limite;
- consultar timeline de auditoria;
- iniciar jornada pelo orchestrator.

---

### Kafka

Usado para eventos de domínio.

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

---

### RabbitMQ

Usado para filas de trabalho, retry e DLQ.

Exemplos:

- envio de comunicação;
- reprocessamento de mensagens;
- tarefas assíncronas com controle de tentativa;
- mensagens que precisam de DLQ.

---

## 10 - Autenticação e autorização

A autenticação será centralizada no `credit-auth-service`.

Fluxo:

```text
1. Usuário faz login no credit-auth-service
2. O serviço gera um token JWT
3. O cliente envia o token nas próximas requisições
4. O credit-api-gateway valida o token
5. Os serviços internos recebem a requisição autenticada
```

Exemplo de header:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

Exemplo de payload do token:

```json
{
  "sub": "user-id",
  "customerId": "customer-id",
  "role": "CUSTOMER"
}
```

---

## 11 - Banco de dados

A plataforma utiliza a estratégia `database per service`.

Cada microservice possui seu próprio banco, evitando acoplamento direto entre domínios.

| Serviço | Banco |
|---|---|
| `credit-auth-service` | `auth_db` |
| [credit-customer-service](https://github.com/diogobackend/credit-customer-service) | `customer_db` |
| `credit-rules-engine-service` | `rules_db` |
| [credit-limit-service](https://github.com/diogobackend/credit-limit-service) | `limit_db` |
| `credit-communication-service` | `communication_db` |
| `credit-audit-service` | `audit_db` |

As alterações de banco são versionadas com Flyway.

Exemplo:

```text
src/main/resources/db/migration/
├── V1__create_customers_table.sql
├── V2__add_unique_constraints_to_customers.sql
├── V3__create_customers_pagination_index.sql
└── V4__create_outbox_events_table.sql
```

---

## 12 - Padrões aplicados

- Arquitetura Hexagonal;
- Ports and Adapters;
- Use Cases;
- DTOs nas bordas;
- Mappers explícitos;
- Entidades de domínio separadas de entidades JPA;
- Constructor Injection;
- Exceptions específicas;
- BigDecimal para valores monetários;
- API Gateway;
- Orchestration Service;
- JWT;
- Spring Security;
- Lib compartilhada para observabilidade;
- Logs automáticos com AOP;
- MDC;
- Mascaramento de dados sensíveis;
- Idempotência em consumers;
- Outbox Pattern;
- Inbox Pattern;
- Retry e DLQ;
- Feature Toggles;
- Logs estruturados;
- Correlation ID;
- Migrations com Flyway;
- Database per service.

---

## 13 - Observabilidade

A plataforma será preparada para expor logs, métricas e tracing.

Parte da padronização técnica será centralizada no `credit-observability-starter`.

---

### Logs

Os logs devem ser estruturados e conter:

- serviceName;
- operation;
- correlationId;
- traceId;
- spanId;
- customerId, quando aplicável;
- eventId, quando aplicável;
- status da operação.

---

### Logs automáticos

Os serviços poderão usar annotations para gerar logs padronizados sem repetir código manualmente.

Exemplo:

```kotlin
@LogInfo(logParameters = true, logReturn = true)
fun execute(input: SomeInput): SomeOutput
```

Campos esperados:

```text
method
parameters
return
correlationId
serviceName
duration
status
```

---

### Métricas

Métricas planejadas:

- quantidade de requests;
- latência por endpoint;
- taxa de erro;
- mensagens consumidas;
- mensagens com erro;
- mensagens enviadas para DLQ;
- limites calculados;
- comunicações enviadas;
- clientes aprovados;
- clientes recusados;
- tokens emitidos;
- falhas de autenticação.

---

### Dashboards

O Grafana será usado para visualizar:

- saúde dos serviços;
- latência;
- taxa de erro;
- throughput;
- backlog de filas;
- quantidade de mensagens em DLQ;
- eventos de negócio;
- falhas de autenticação;
- volume de jornadas processadas.

---

## 14 - Tecnologias

- Kotlin
- Java 21
- Spring Boot
- Spring Web
- Spring Security
- Spring Data JPA
- Spring Cloud Gateway
- Spring Cloud Config
- Spring Cloud Stream
- MySQL
- Flyway
- Kafka
- RabbitMQ
- Docker
- Kubernetes
- Prometheus
- Grafana
- OpenTelemetry
- Logs estruturados
- JWT

---

Esse Peojeto simula problemas comuns de sistemas backend reais.

| Problema real | Solução aplicada |
|---|---|
| Entrada desorganizada entre serviços | API Gateway |
| Fluxo de negócio espalhado | Orchestrator Service |
| Login e segurança distribuídos incorretamente | Auth Service com JWT |
| Duplicação de logs entre serviços | Observability Starter |
| Perda de evento após salvar no banco | Outbox Pattern |
| Mensagem processada mais de uma vez | Idempotência / Inbox Pattern |
| Falha temporária no envio de comunicação | Retry |
| Mensagem inválida ou erro não recuperável | DLQ |
| Dificuldade para rastrear uma jornada | Correlation ID |
| Mudança de regra sem novo deploy | Feature Toggle |
| Crescimento de responsabilidades | Microservices por contexto |
| Dificuldade de investigação em produção | Logs, métricas e tracing |
| Acoplamento entre banco e domínio | Arquitetura Hexagonal |

---

## 15 - Status

Projeto em desenvolvimento

Construção incremental planejada:

```text
1. credit-customer-service
2. credit-limit-service
3. credit-observability-starter
4. credit-rules-engine-service
5. credit-orchestrator-service
6. credit-auth-service
7. credit-api-gateway
8. credit-communication-service
9. credit-audit-service
10. outbox/eventos
11. observabilidade e infraestrutura
```

Status atual:

```text
credit-customer-service: CRUD básico implementado
credit-limit-service: iniciando desenvolvimento
```

# Credit Journey Platform

Plataforma backend fictícia que simula a jornada de crédito de um cliente em um banco digital.

O projeto representa o funcionamento de uma plataforma usada por um banco emissor de cartão para cadastrar clientes, avaliar elegibilidade de crédito, calcular limite, comunicar o cliente e registrar toda a jornada para auditoria.

## O que este produto resolve?

Em um banco digital, a concessão de crédito não acontece em uma única etapa.

Antes de um cliente receber limite no cartão, o sistema precisa:

- cadastrar e manter os dados do cliente;
- avaliar se o cliente está elegível para crédito;
- aplicar regras de risco e políticas internas;
- calcular o limite disponível;
- comunicar o cliente sobre aprovação, rejeição ou alteração de limite;
- registrar tudo para auditoria, rastreabilidade e suporte operacional.

A Credit Journey Platform simula esse fluxo usando microservices independentes, mensageria, banco por serviço, observabilidade e padrões próximos de um ambiente real de produção.

## Exemplo prático

Imagine o seguinte cenário:

1. Um cliente chamado João se cadastra no banco digital.
2. O `credit-customer-service` salva seus dados e publica o evento `CustomerCreated`.
3. O `credit-rules-engine-service` consome esse evento e avalia se João pode receber crédito.
4. Se João for aprovado, o serviço publica `CustomerEligibilityApproved`.
5. O `credit-limit-service` calcula um limite inicial, por exemplo, R$ 1.000,00.
6. O `credit-communication-service` envia uma notificação informando que o limite foi aprovado.
7. O `credit-audit-service` registra toda a jornada para consulta futura.

Fluxo resumido:

```text
Cliente cadastrado
      |
      v
Elegibilidade avaliada
      |
      v
Limite calculado
      |
      v
Comunicação enviada
      |
      v
Jornada auditada

Objetivo técnico

Este projeto foi criado como portfólio backend para demonstrar boas práticas de desenvolvimento, arquitetura e operação de sistemas distribuídos.

Principais objetivos:

aplicar Arquitetura Hexagonal / Ports and Adapters;
separar domínio, aplicação e infraestrutura;
construir microservices independentes;
usar comunicação síncrona e assíncrona;
aplicar Kafka para eventos de domínio;
aplicar RabbitMQ para filas de trabalho, retry e DLQ;
versionar banco de dados com Flyway;
usar MySQL com banco por serviço;
implementar Outbox Pattern;
aplicar Feature Toggles;
centralizar configurações;
expor métricas com Prometheus;
criar dashboards no Grafana;
gerar logs estruturados com correlationId;
preparar os serviços para Docker e Kubernetes.
Visão da arquitetura

A plataforma é composta por microservices independentes, cada um responsável por um contexto específico da jornada de crédito

credit-customer-service
        |
        | CustomerCreated
        v
credit-rules-engine-service
        |
        | CustomerEligibilityApproved / CustomerEligibilityRejected
        v
credit-limit-service
        |
        | LimitCalculated / LimitUpdated
        v
credit-communication-service
        |
        | CommunicationSent / CommunicationFailed
        v
credit-audit-service

Além dos serviços de negócio, a plataforma também possui repositórios de apoio para configuração, contratos e infraestrutura.

Repositórios
Repositório	Responsabilidade
credit-customer-service	Gerencia cadastro, status e perfil do cliente
credit-rules-engine-service	Avalia elegibilidade e políticas de crédito
credit-limit-service	Calcula e mantém limites de crédito
credit-communication-service	Dispara comunicações para o cliente
credit-audit-service	Registra histórico, eventos e timeline da jornada
credit-config-server	Centraliza configurações dos serviços
credit-platform-infra	Contém Docker Compose, Kubernetes, Prometheus, Grafana, Kafka, RabbitMQ e MySQL
credit-shared-contracts	Centraliza contratos OpenAPI, AsyncAPI e schemas de eventos
Responsabilidade dos serviços
credit-customer-service

Responsável pelo cadastro e manutenção dos clientes.

Exemplos de responsabilidades:

criar cliente;
consultar cliente;
atualizar dados cadastrais;
alterar status;
publicar eventos como CustomerCreated e CustomerStatusChanged.
credit-rules-engine-service

Responsável por avaliar se um cliente está elegível para crédito.

Exemplos de regras:

cliente bloqueado não pode receber limite;
cliente inativo não pode seguir no fluxo;
renda mínima pode ser exigida;
regras novas podem ser ativadas por feature toggle.
credit-limit-service

Responsável pelo cálculo e manutenção do limite de crédito.

Exemplos de responsabilidades:

calcular limite inicial;
consultar limite disponível;
bloquear limite;
liberar limite;
registrar histórico de alterações;
publicar eventos como LimitCalculated e LimitUpdated.
credit-communication-service

Responsável por comunicar o cliente sobre eventos da jornada.

Exemplos de comunicação:

limite aprovado;
limite alterado;
cliente não elegível;
falha ou pendência no processamento.

Também demonstra uso de RabbitMQ, retry e DLQ.

credit-audit-service

Responsável por registrar uma visão auditável da jornada.

Exemplo de timeline:

10:00 - CustomerCreated
10:01 - CustomerEligibilityApproved
10:02 - LimitCalculated
10:03 - CommunicationSent

Esse serviço permite consultar o que aconteceu, quando aconteceu e qual foi o correlationId da operação.

Arquitetura Hexagonal

Cada microservice segue Arquitetura Hexagonal, mantendo o domínio isolado de frameworks e infraestrutura.

Estrutura padrão:

src/main/java/com/portfolio/{service}/
├── core/
│   ├── domain/
│   ├── port/
│   │   ├── input/
│   │   └── output/
│   └── usecase/
├── app/
│   ├── adapter/
│   │   ├── input/
│   │   │   ├── web/
│   │   │   └── messaging/
│   │   └── output/
│   │       ├── persistence/
│   │       ├── messaging/
│   │       └── client/
│   └── configuration/

Regras arquiteturais:

controller não acessa repository diretamente;
consumer não executa regra de negócio diretamente;
repository não conhece use case;
DTO não entra no domínio;
entidade JPA não é modelo de domínio;
domínio não depende de Spring;
toda entrada passa por uma porta de entrada;
toda saída passa por uma porta de saída.
Comunicação entre serviços

A plataforma usa dois tipos principais de comunicação.

REST

Usado para operações síncronas, principalmente consultas e comandos diretos.

Exemplos:

consultar cliente;
consultar limite;
consultar timeline de auditoria.
Kafka

Usado para eventos de domínio.

Exemplos:

CustomerCreated
CustomerEligibilityApproved
CustomerEligibilityRejected
LimitCalculated
LimitUpdated
CommunicationSent
RabbitMQ

Usado para filas de trabalho, retry e DLQ.

Exemplos:

envio de comunicação;
reprocessamento de mensagens;
tarefas assíncronas com controle de tentativa.
Banco de dados

A plataforma utiliza a estratégia database per service.

Cada microservice possui seu próprio banco, evitando acoplamento direto entre domínios.

Serviço	Banco
credit-customer-service	customer_db
credit-rules-engine-service	rules_db
credit-limit-service	limit_db
credit-communication-service	communication_db
credit-audit-service	audit_db

As alterações de banco são versionadas com Flyway.

Exemplo:

src/main/resources/db/migration/
├── V1__create_customers_table.sql
├── V2__create_customer_status_history_table.sql
└── V3__create_outbox_events_table.sql
Padrões aplicados
Arquitetura Hexagonal
Ports and Adapters
Use Cases
DTOs nas bordas
Mappers explícitos
Entidades de domínio separadas de entidades JPA
Constructor Injection
Exceptions específicas
BigDecimal para valores monetários
Idempotência em consumers
Outbox Pattern
Inbox Pattern
Retry e DLQ
Feature Toggles
Logs estruturados
Correlation ID
Migrations com Flyway
Database per service
Observabilidade

A plataforma será preparada para expor logs, métricas e tracing.

Logs

Os logs devem ser estruturados e conter:

serviceName;
operation;
correlationId;
traceId;
spanId;
customerId, quando aplicável;
eventId, quando aplicável;
status da operação.
Métricas

Métricas planejadas:

quantidade de requests;
latência por endpoint;
taxa de erro;
mensagens consumidas;
mensagens com erro;
mensagens enviadas para DLQ;
limites calculados;
comunicações enviadas;
clientes aprovados e recusados.
Dashboards

O Grafana será usado para visualizar:

saúde dos serviços;
latência;
taxa de erro;
throughput;
backlog de filas;
quantidade de mensagens em DLQ;
eventos de negócio.
Tecnologias
Java 21
Spring Boot
Spring Web
Spring Data JPA
Spring Cloud Config
Spring Cloud Stream
MySQL
Flyway
Kafka
RabbitMQ
Docker
Kubernetes
Prometheus
Grafana
OpenTelemetry
Logs estruturados
Por que esse projeto é relevante como portfólio?

Este projeto não é apenas um CRUD.

Ele simula problemas comuns de sistemas backend reais:

Problema real	Solução aplicada
Perda de evento após salvar no banco	Outbox Pattern
Mensagem processada mais de uma vez	Idempotência / Inbox Pattern
Falha temporária no envio de comunicação	Retry
Mensagem inválida ou erro não recuperável	DLQ
Dificuldade para rastrear uma jornada	Correlation ID
Mudança de regra sem novo deploy	Feature Toggle
Crescimento de responsabilidades	Microservices por contexto
Dificuldade de investigação em produção	Logs, métricas e tracing
Acoplamento entre banco e domínio	Arquitetura Hexagonal
Status

Projeto em desenvolvimento.

A construção será feita de forma incremental, uma task por vez, começando pela documentação da plataforma e depois pela criação dos microservices.
# Arquitetura

A Credit Journey Platform é uma plataforma backend fictícia baseada em microservices independentes, criada para simular a jornada de crédito de um cliente em um banco digital.

O objetivo arquitetural é demonstrar boas práticas próximas de um ambiente real de produção, com foco em separação de responsabilidades, baixo acoplamento, mensageria, rastreabilidade e observabilidade.

## Estilo arquitetural

A arquitetura principal adotada é:

- Microservices
- Hexagonal Architecture / Ports and Adapters
- Event-driven architecture
- Database per service
- Configuração externa
- Observabilidade distribuída

## Regra principal

O domínio não deve depender de detalhes externos.

Isso significa que o núcleo da aplicação não deve conhecer:

- Spring
- Banco de dados
- Kafka
- RabbitMQ
- HTTP
- Frameworks
- Bibliotecas de infraestrutura

## Estrutura padrão dos serviços

Cada microservice deve seguir uma estrutura parecida com:

src/main/java/com/portfolio/{service}/

- core/
  - domain/
    - model/
    - exception/
    - valueobject/
  - port/
    - input/
    - output/
  - usecase/

- app/
  - adapter/
    - input/
      - web/
      - messaging/
    - output/
      - persistence/
      - messaging/
      - client/
  - configuration/

## Responsabilidade das camadas

### core/domain

Contém as regras centrais do domínio.

Não deve conhecer Spring, JPA, Kafka, RabbitMQ ou qualquer tecnologia externa.

Exemplos:

- Customer
- CreditLimit
- EligibilityDecision
- CommunicationRequest
- AuditEvent

### core/port/input

Define as portas de entrada da aplicação.

Representa o que o serviço sabe fazer.

Exemplos:

- CreateCustomerUseCase
- CalculateLimitUseCase
- EvaluateEligibilityUseCase
- SendCommunicationUseCase

### core/port/output

Define as dependências externas que o domínio precisa acessar.

Exemplos:

- CustomerRepositoryPort
- EventPublisherPort
- LimitRepositoryPort
- CommunicationProviderPort
- AuditEventRepositoryPort

### core/usecase

Contém a implementação dos casos de uso.

Aqui ficam as regras de aplicação e a orquestração do domínio.

### app/adapter/input/web

Contém controllers REST.

Responsável por receber requisições HTTP, validar DTOs e chamar portas de entrada.

### app/adapter/input/messaging

Contém consumers Kafka ou RabbitMQ.

Responsável por receber mensagens, converter payloads e chamar portas de entrada.

### app/adapter/output/persistence

Contém adapters de persistência.

Inclui entities JPA, repositories Spring Data e mappers entre Entity e Domain.

### app/adapter/output/messaging

Contém producers Kafka/RabbitMQ.

Responsável por publicar eventos ou comandos para outros serviços.

### app/configuration

Contém configurações Spring.

Exemplos:

- Beans
- Configuração de banco
- Configuração de mensageria
- Configuração de observabilidade
- Configuração de WebClient

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

## Comunicação entre serviços

A comunicação entre serviços será feita de duas formas:

### REST

Usado para consultas ou comandos síncronos quando a resposta imediata for necessária.

Exemplo:

- Consultar cliente
- Consultar limite
- Consultar timeline de auditoria

### Mensageria

Usada para eventos de domínio e processamento assíncrono.

Exemplo:

- CustomerCreated
- LimitCalculated
- CommunicationSent
- CustomerEligibilityRejected

## Padrões importantes

### Outbox Pattern

Usado para garantir que alterações no banco e publicação de eventos aconteçam de forma segura.

Fluxo:

1. O caso de uso processa a regra.
2. O estado é salvo no banco.
3. O evento é salvo na tabela de outbox.
4. Um publicador assíncrono envia o evento.
5. O evento é marcado como publicado.

### Inbox Pattern

Usado para evitar processamento duplicado de mensagens recebidas.

Fluxo:

1. O consumer recebe uma mensagem.
2. Verifica se o eventId já foi processado.
3. Se já foi processado, ignora.
4. Se não foi, processa e registra na tabela inbox.

### Feature Toggle

Usado para ligar ou desligar comportamentos sem novo deploy.

Exemplos:

- enable-risk-rule-v2
- enable-async-communication
- enable-limit-auto-approval

## Objetivo da arquitetura

A arquitetura deve permitir:

- Evolução independente dos serviços
- Baixo acoplamento
- Alta rastreabilidade
- Facilidade de testes
- Clareza na separação de responsabilidades
- Simulação realista de produção
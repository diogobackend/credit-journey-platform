# Mensageria

Este documento descreve a estratégia de mensageria da **Credit Journey Platform**.

A plataforma utiliza mensageria para desacoplar serviços, permitir processamento assíncrono, aumentar rastreabilidade e simular uma arquitetura próxima de produção.

A estratégia usa **Kafka** e **RabbitMQ** com responsabilidades diferentes.

---

## Visão geral

A plataforma usa mensageria para dois tipos principais de comunicação:

```text
Kafka    -> eventos de domínio
RabbitMQ -> filas de trabalho, comandos assíncronos, retry e DLQ
```

Resumo:

| Tecnologia | Uso principal |
|---|---|
| Kafka | Propagar fatos de negócio que já aconteceram |
| RabbitMQ | Processar tarefas, comandos, retry e DLQ |

---

## Kafka

Kafka será usado para **eventos de domínio**.

Eventos de domínio representam fatos que já aconteceram no sistema.

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
CreditJourneyStarted
CreditJourneyCompleted
CreditJourneyFailed
```

---

## Quando usar Kafka

Usar Kafka quando:

- o evento representa um fato de negócio;
- mais de um serviço pode ter interesse no mesmo evento;
- o evento precisa ser reprocessável;
- o evento precisa compor histórico ou auditoria;
- o produtor não precisa saber quem vai consumir;
- o evento pode ser usado por serviços futuros;
- a comunicação deve ser desacoplada.

Exemplo:

```text
O cliente foi criado.
```

Evento publicado:

```text
CustomerCreated
```

Consumidores possíveis:

```text
credit-rules-engine-service
credit-audit-service
credit-orchestrator-service
```

---

## Tópicos Kafka planejados

| Tópico | Eventos |
|---|---|
| `customer.events` | `CustomerCreated`, `CustomerUpdated`, `CustomerStatusChanged`, `CustomerDeleted` |
| `rules.events` | `CustomerEligibilityApproved`, `CustomerEligibilityRejected`, `CustomerManualAnalysisRequested` |
| `limit.events` | `LimitCalculated`, `LimitUpdated`, `LimitBlocked`, `LimitReleased` |
| `communication.events` | `CommunicationSent`, `CommunicationFailed`, `CommunicationDiscarded` |
| `credit-journey.events` | `CreditJourneyStarted`, `CreditJourneyCompleted`, `CreditJourneyFailed` |
| `audit.events` | Eventos internos de auditoria, se necessário |

---

## Chave dos eventos Kafka

Eventos Kafka devem usar uma chave estável para manter ordenação por agregado.

Exemplos:

| Evento | Chave recomendada |
|---|---|
| `CustomerCreated` | `customerId` |
| `CustomerUpdated` | `customerId` |
| `CustomerEligibilityApproved` | `customerId` |
| `LimitCalculated` | `customerId` |
| `CommunicationSent` | `customerId` |
| `CreditJourneyStarted` | `journeyId` |

Regra:

```text
Eventos do mesmo cliente devem preferencialmente usar customerId como key.
```

---

## RabbitMQ

RabbitMQ será usado para **filas de trabalho**, **comandos assíncronos**, **retry** e **DLQ**.

Diferente do Kafka, aqui a ideia é processar uma tarefa específica.

Exemplo:

```text
Envie uma comunicação para o cliente.
```

Mensagem:

```text
SendCommunicationCommand
```

Consumidor:

```text
credit-communication-service
```

---

## Quando usar RabbitMQ

Usar RabbitMQ quando:

- existe uma tarefa específica a ser executada;
- a mensagem representa um comando;
- a mensagem precisa de retry controlado;
- existe necessidade clara de DLQ;
- o processamento pode falhar por motivo técnico;
- apenas um consumidor deve processar aquela tarefa;
- o fluxo precisa controlar tentativas.

---

## Filas RabbitMQ planejadas

| Fila | Responsabilidade |
|---|---|
| `communication.send.queue` | Fila principal de envio de comunicação |
| `communication.send.retry.queue` | Fila de retry de comunicação |
| `communication.send.dlq` | Fila de mensagens mortas de comunicação |
| `limit.reprocess.queue` | Reprocessamento de limite, se necessário |
| `limit.reprocess.retry.queue` | Retry de reprocessamento de limite |
| `limit.reprocess.dlq` | DLQ de reprocessamento de limite |
| `audit.register.queue` | Registro assíncrono de auditoria, se necessário |
| `audit.register.dlq` | DLQ de auditoria |

---

## Diferença prática entre Kafka e RabbitMQ

### Kafka

Usado para evento de domínio.

Exemplo:

```text
O cliente foi criado.
```

Evento:

```text
CustomerCreated
```

Possíveis consumidores:

```text
credit-rules-engine-service
credit-audit-service
credit-orchestrator-service
```

O produtor não precisa saber quem vai consumir.

---

### RabbitMQ

Usado para comando ou tarefa.

Exemplo:

```text
Envie uma comunicação para o cliente.
```

Mensagem:

```text
SendCommunicationCommand
```

Consumidor:

```text
credit-communication-service
```

A mensagem tem um destino funcional mais claro.

---

## Regra simples de decisão

```text
Aconteceu algo importante? Kafka.
Precisa executar uma tarefa? RabbitMQ.
```

Exemplos:

| Situação | Tecnologia |
|---|---|
| Cliente criado | Kafka |
| Elegibilidade aprovada | Kafka |
| Limite calculado | Kafka |
| Comunicação enviada | Kafka |
| Enviar e-mail para cliente | RabbitMQ |
| Reprocessar cálculo de limite | RabbitMQ |
| Tentar novamente uma comunicação | RabbitMQ |
| Enviar mensagem para DLQ | RabbitMQ |

---

## Eventos por serviço

### credit-customer-service

Publica eventos relacionados ao cliente.

Eventos:

```text
CustomerCreated
CustomerUpdated
CustomerStatusChanged
CustomerDeleted
```

Tópico:

```text
customer.events
```

---

### credit-rules-engine-service

Consome eventos de cliente e publica decisões de elegibilidade.

Consome:

```text
CustomerCreated
CustomerUpdated
CustomerStatusChanged
```

Publica:

```text
CustomerEligibilityApproved
CustomerEligibilityRejected
CustomerManualAnalysisRequested
```

Tópico:

```text
rules.events
```

---

### credit-limit-service

Consome eventos de elegibilidade e publica eventos de limite.

Consome:

```text
CustomerEligibilityApproved
```

Publica:

```text
LimitCalculated
LimitUpdated
LimitBlocked
LimitReleased
```

Tópico:

```text
limit.events
```

---

### credit-communication-service

Consome comandos de comunicação via RabbitMQ e pode publicar eventos de resultado via Kafka.

Consome fila:

```text
communication.send.queue
```

Publica eventos:

```text
CommunicationSent
CommunicationFailed
CommunicationDiscarded
```

Tópico:

```text
communication.events
```

---

### credit-audit-service

Consome eventos relevantes da jornada para montar histórico e timeline.

Consome:

```text
customer.events
rules.events
limit.events
communication.events
credit-journey.events
```

Objetivo:

```text
registrar o que aconteceu, quando aconteceu e qual correlationId da operação.
```

---

### credit-orchestrator-service

Pode usar REST para coordenar a jornada e Kafka para publicar eventos da jornada.

Publica:

```text
CreditJourneyStarted
CreditJourneyCompleted
CreditJourneyFailed
```

Tópico:

```text
credit-journey.events
```

Também pode solicitar tarefas assíncronas via RabbitMQ quando necessário.

---

## Padrão base dos eventos Kafka

Todo evento Kafka deve conter metadados mínimos.

Campos obrigatórios:

```text
eventId
eventType
eventVersion
occurredAt
correlationId
source
payload
```

Exemplo conceitual:

```json
{
  "eventId": "52cf7e82-cda3-4fd5-a573-79a34a176bb4",
  "eventType": "CustomerCreated",
  "eventVersion": "1.0",
  "occurredAt": "2026-01-01T10:00:00Z",
  "correlationId": "9a6e39fb-d573-4a18-9b3c-1b84cb5a24c2",
  "source": "credit-customer-service",
  "payload": {
    "customerId": "11111111-1111-1111-1111-111111111111",
    "name": "João Silva",
    "document": "12345678900",
    "status": "ACTIVE"
  }
}
```

---

## Padrão base das mensagens RabbitMQ

Toda mensagem RabbitMQ deve conter metadados mínimos.

Campos obrigatórios:

```text
messageId
correlationId
type
createdAt
attempt
payload
```

Exemplo conceitual:

```json
{
  "messageId": "ad4b9f16-c68b-4893-a191-a34c63d0e022",
  "correlationId": "9a6e39fb-d573-4a18-9b3c-1b84cb5a24c2",
  "type": "SendCommunicationCommand",
  "createdAt": "2026-01-01T10:01:00Z",
  "attempt": 1,
  "payload": {
    "customerId": "11111111-1111-1111-1111-111111111111",
    "channel": "EMAIL",
    "templateCode": "LIMIT_APPROVED"
  }
}
```

---

## Versionamento de eventos

Eventos devem possuir versão.

Campo:

```text
eventVersion
```

Exemplo:

```json
{
  "eventType": "CustomerCreated",
  "eventVersion": "1.0"
}
```

Regras:

- não quebrar consumidores existentes sem nova versão;
- adicionar campos opcionais quando possível;
- evitar remover campos de eventos já publicados;
- documentar mudanças no `credit-shared-contracts`;
- manter exemplos de payload por versão.

---

## Contratos de eventos

Os contratos dos eventos devem ficar centralizados no repositório:

```text
credit-shared-contracts
```

Esse repositório pode conter:

- AsyncAPI;
- schemas JSON;
- exemplos de eventos;
- exemplos de comandos;
- documentação de tópicos;
- documentação de filas.

---

## Retry

Erros transitórios devem gerar retry.

Exemplos de erro transitório:

- timeout;
- serviço externo indisponível;
- falha temporária de conexão;
- erro momentâneo de infraestrutura;
- indisponibilidade temporária do broker;
- erro temporário de provedor externo.

Estratégia:

```text
1. consumer tenta processar
2. erro transitório acontece
3. mensagem vai para retry
4. mensagem retorna para processamento
5. se exceder o máximo de tentativas, vai para DLQ
```

---

## DLQ

DLQ significa Dead Letter Queue.

Mensagens devem ir para DLQ quando:

- excederem o número máximo de tentativas;
- possuírem payload inválido;
- tiverem erro de negócio não recuperável;
- falharem por inconsistência de dados;
- não puderem ser processadas com segurança;
- tiverem erro inesperado recorrente.

Exemplo:

```text
communication.send.dlq
```

---

## Estratégia de erro

| Tipo de erro | Ação |
|---|---|
| Erro transitório | Retry |
| Erro de negócio não recuperável | DLQ |
| Payload inválido | DLQ |
| Duplicidade | Ignorar com log |
| Evento já processado | Ignorar com idempotência |
| Erro inesperado recorrente | DLQ |
| Serviço externo indisponível | Retry |
| Dados obrigatórios ausentes | DLQ |

---

## Idempotência

Consumers devem ser idempotentes.

Isso evita que uma mesma mensagem processe duas vezes a mesma operação.

Estratégias:

- registrar `eventId` processado;
- registrar `messageId` processado;
- usar tabela `inbox_events`;
- verificar estado atual antes de alterar dados;
- usar constraints únicas quando fizer sentido;
- evitar efeitos colaterais duplicados.

Exemplo:

```text
Se CommunicationSent já foi processado, o audit-service não deve registrar a mesma timeline duas vezes.
```

---

## Outbox Pattern

Serviços que publicam eventos críticos devem usar Outbox Pattern.

Objetivo:

```text
Evitar que o sistema salve dados no banco, mas falhe ao publicar o evento.
```

Fluxo:

```text
1. caso de uso processa a regra
2. salva alteração de domínio
3. salva evento na tabela outbox_events
4. processo assíncrono publica o evento
5. marca evento como publicado
```

Serviços candidatos:

```text
credit-customer-service
credit-rules-engine-service
credit-limit-service
credit-orchestrator-service
credit-communication-service
```

---

## Inbox Pattern

Serviços que consomem eventos críticos devem usar Inbox Pattern.

Objetivo:

```text
Evitar processamento duplicado.
```

Fluxo:

```text
1. consumer recebe evento
2. verifica se eventId já existe na inbox_events
3. se existir, ignora
4. se não existir, processa
5. registra evento como processado
```

Serviços candidatos:

```text
credit-rules-engine-service
credit-limit-service
credit-communication-service
credit-audit-service
```

---

## Estrutura da tabela outbox_events

Campos recomendados:

```text
id
event_id
event_type
event_version
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

## Estrutura da tabela inbox_events

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

## Observabilidade da mensageria

Cada producer e consumer deve gerar logs e métricas.

Campos esperados em logs:

```text
serviceName
operation
topic
queue
eventType
messageId
eventId
correlationId
status
duration
attempt
error
```

Exemplo:

```text
serviceName=credit-communication-service
operation=consume-message
queue=communication.send.queue
messageId=ad4b9f16-c68b-4893-a191-a34c63d0e022
correlationId=9a6e39fb-d573-4a18-9b3c-1b84cb5a24c2
status=SUCCESS
duration=120ms
```

Parte da padronização de logs será centralizada na lib:

```text
credit-observability-starter
```

---

## Métricas importantes

Métricas planejadas:

```text
messages_published_total
messages_consumed_total
messages_failed_total
messages_retried_total
messages_dlq_total
message_processing_duration_seconds
outbox_pending_total
outbox_published_total
outbox_failed_total
inbox_duplicates_total
kafka_consumer_lag
rabbitmq_queue_backlog
```

---

## Dashboards de mensageria

Dashboards planejados no Grafana:

- mensagens publicadas;
- mensagens consumidas;
- mensagens com erro;
- mensagens em retry;
- mensagens em DLQ;
- tempo médio de consumo;
- backlog de filas;
- lag dos consumers Kafka;
- eventos pendentes na outbox;
- eventos duplicados ignorados pela inbox.

---

## Boas práticas

- Usar Kafka para eventos de domínio.
- Usar RabbitMQ para comandos, tarefas, retry e DLQ.
- Todo evento deve possuir `eventId`.
- Toda mensagem deve possuir `messageId`.
- Toda mensagem deve propagar `correlationId`.
- Consumers devem ser idempotentes.
- Eventos críticos devem ser publicados via Outbox Pattern.
- Eventos críticos consumidos devem usar Inbox Pattern.
- Payloads devem ser versionados.
- Contratos devem ser documentados no `credit-shared-contracts`.
- Payload inválido deve ir para DLQ.
- Erro transitório deve gerar retry.
- Evento duplicado deve ser ignorado com log.
- Não colocar regra de negócio pesada no producer ou consumer.
- Consumer deve chamar porta de entrada da aplicação.
- Producer deve ficar em adapter de saída.
- Mensageria não deve quebrar isolamento entre domínios.

---

## Objetivo da estratégia de mensageria

A estratégia de mensageria deve permitir:

- baixo acoplamento;
- processamento assíncrono;
- rastreabilidade;
- reprocessamento;
- idempotência;
- tolerância a falhas;
- auditoria da jornada;
- comunicação entre microservices;
- simulação realista de produção.
# Mensageria

A Credit Journey Platform utiliza mensageria para desacoplar serviços, permitir processamento assíncrono e aumentar a rastreabilidade da jornada de crédito.

A estratégia usa Kafka e RabbitMQ com responsabilidades diferentes.

## Kafka

Kafka será usado para eventos de domínio.

Eventos de domínio representam fatos que já aconteceram no sistema.

Exemplos:

- CustomerCreated
- CustomerStatusChanged
- CustomerEligibilityApproved
- CustomerEligibilityRejected
- LimitCalculated
- LimitUpdated
- CommunicationSent
- CommunicationFailed

## Quando usar Kafka

Usar Kafka quando:

- O evento representa um fato de negócio.
- Mais de um serviço pode ter interesse no mesmo evento.
- O evento precisa ser reprocessável.
- O evento precisa compor histórico ou auditoria.
- O serviço produtor não precisa saber quem vai consumir.

## Tópicos planejados

| Tópico | Eventos |
|---|---|
| customer.events | CustomerCreated, CustomerUpdated, CustomerStatusChanged |
| rules.events | CustomerEligibilityApproved, CustomerEligibilityRejected |
| limit.events | LimitCalculated, LimitUpdated, LimitBlocked, LimitReleased |
| communication.events | CommunicationSent, CommunicationFailed, CommunicationDiscarded |
| audit.events | Eventos internos de auditoria, se necessário |

## RabbitMQ

RabbitMQ será usado para filas de trabalho, comandos assíncronos, retry e DLQ.

Diferente do Kafka, aqui a ideia é processar tarefas específicas.

## Quando usar RabbitMQ

Usar RabbitMQ quando:

- Existe uma tarefa a ser executada.
- A mensagem precisa de retry controlado.
- Existe necessidade de DLQ.
- O processamento pode falhar por motivo técnico.
- A mensagem representa um comando, não apenas um fato.

## Filas planejadas

| Fila | Responsabilidade |
|---|---|
| communication.send.queue | Fila principal de envio de comunicação |
| communication.send.retry.queue | Fila de retry de comunicação |
| communication.send.dlq | Fila de mensagens mortas de comunicação |
| limit.reprocess.queue | Reprocessamento de limite, se necessário |
| limit.reprocess.dlq | DLQ de reprocessamento de limite |

## Diferença prática

### Kafka

Exemplo:

O cliente foi criado.

Evento:

- CustomerCreated

Consumidores possíveis:

- rules-engine-service
- audit-service

### RabbitMQ

Exemplo:

Envie uma comunicação para o cliente.

Comando:

- SendCommunicationCommand

Consumidor:

- communication-service

## Padrão base dos eventos

Todo evento deve conter metadados mínimos:

- eventId
- eventType
- eventVersion
- occurredAt
- correlationId
- source
- payload

Exemplo conceitual:

{
  "eventId": "uuid",
  "eventType": "CustomerCreated",
  "eventVersion": "1.0",
  "occurredAt": "2026-01-01T10:00:00Z",
  "correlationId": "uuid",
  "source": "credit-customer-service",
  "payload": {}
}

## Padrão base das mensagens RabbitMQ

Toda mensagem de fila deve conter:

- messageId
- correlationId
- type
- createdAt
- attempt
- payload

## Retry

Erros transitórios devem gerar retry.

Exemplos de erro transitório:

- Timeout
- Serviço externo indisponível
- Falha temporária de conexão
- Erro momentâneo de infraestrutura

## DLQ

Mensagens devem ir para DLQ quando:

- excederem o número máximo de tentativas
- possuírem payload inválido
- tiverem erro de negócio não recuperável
- falharem por inconsistência de dados
- não puderem ser processadas com segurança

## Estratégia de erro

| Tipo de erro | Ação |
|---|---|
| Erro transitório | Retry |
| Erro de negócio não recuperável | DLQ |
| Payload inválido | DLQ |
| Duplicidade | Ignorar com log |
| Evento já processado | Ignorar com idempotência |

## Idempotência

Consumers devem ser idempotentes.

Isso evita que uma mesma mensagem processe duas vezes a mesma operação.

Estratégias:

- Registrar eventId processado
- Registrar messageId processado
- Usar tabela inbox_events
- Verificar estado atual antes de alterar dados

## Outbox Pattern

Serviços que publicam eventos críticos devem usar Outbox Pattern.

Objetivo:

Evitar que o sistema salve dados no banco, mas falhe ao publicar o evento.

Fluxo:

1. Salva alteração de domínio.
2. Salva evento na outbox.
3. Processo assíncrono publica o evento.
4. Marca evento como publicado.

## Inbox Pattern

Serviços que consomem eventos críticos devem usar Inbox Pattern.

Objetivo:

Evitar processamento duplicado.

Fluxo:

1. Recebe evento.
2. Verifica se eventId já existe na inbox.
3. Se existir, ignora.
4. Se não existir, processa.
5. Registra evento como processado.

## Observabilidade da mensageria

Cada consumer deve gerar logs e métricas com:

- consumer name
- topic ou queue
- eventType
- messageId
- correlationId
- status
- tempo de processamento
- quantidade de retries
- erro ocorrido

## Métricas importantes

- mensagens consumidas
- mensagens publicadas
- mensagens com erro
- mensagens enviadas para DLQ
- tempo médio de consumo
- backlog de fila
- quantidade de retries
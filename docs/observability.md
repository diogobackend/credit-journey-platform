# Observabilidade

A Credit Journey Platform deve possuir observabilidade desde o início.

O objetivo é permitir investigação de problemas, acompanhamento da saúde dos serviços e rastreabilidade da jornada de crédito.

A observabilidade será baseada em três pilares:

- Logs
- Métricas
- Tracing

## Logs

Os serviços devem gerar logs estruturados.

Formato recomendado:

- JSON

Campos importantes:

- timestamp
- level
- serviceName
- operation
- traceId
- spanId
- correlationId
- customerId quando aplicável
- eventId quando aplicável
- messageId quando aplicável
- status
- errorCode
- errorMessage

## Correlation ID

Toda requisição ou mensagem deve possuir correlationId.

O correlationId permite rastrear a mesma jornada entre vários serviços.

Exemplo de fluxo:

1. customer-service cria cliente.
2. Publica CustomerCreated com correlationId.
3. rules-engine-service consome o evento.
4. Publica CustomerEligibilityApproved com o mesmo correlationId.
5. limit-service calcula limite.
6. communication-service envia notificação.
7. audit-service registra tudo.

## Dados sensíveis

Dados sensíveis não devem ser logados em texto puro.

Exemplos de dados que devem ser mascarados:

- CPF
- documento
- cartão
- senha
- token
- e-mail completo, dependendo do contexto
- telefone completo

## Splunk

Os logs estruturados devem ser compatíveis com ingestão em Splunk.

Neste portfólio, o Splunk pode ser documentado e simulado com logs JSON.

Campos importantes para busca:

- serviceName
- correlationId
- customerId
- eventType
- status
- errorCode

Exemplos de investigação:

- Buscar jornada por correlationId.
- Buscar erro por customerId.
- Buscar mensagens enviadas para DLQ.
- Buscar falhas de comunicação.
- Buscar eventos duplicados.

## Métricas

As métricas serão expostas via Spring Boot Actuator e coletadas pelo Prometheus.

Cada serviço deve expor:

- /actuator/health
- /actuator/metrics
- /actuator/prometheus

## Métricas técnicas

Métricas esperadas:

- quantidade de requests HTTP
- latência por endpoint
- taxa de erro HTTP
- uso de memória
- uso de CPU
- conexões com banco
- status de health check

## Métricas de negócio

Métricas esperadas:

- clientes criados
- clientes aprovados
- clientes recusados
- limites calculados
- comunicações enviadas
- comunicações com falha
- eventos auditados

## Métricas de mensageria

Métricas esperadas:

- mensagens consumidas
- mensagens publicadas
- mensagens com erro
- mensagens em retry
- mensagens em DLQ
- tempo médio de processamento
- backlog de fila

## Grafana

O Grafana será usado para dashboards.

Dashboards planejados:

### Visão geral da plataforma

- saúde dos serviços
- taxa de erro
- latência média
- throughput
- uso de CPU e memória

### Mensageria

- mensagens consumidas por serviço
- mensagens com erro
- filas com backlog
- DLQs
- retries

### Jornada de crédito

- clientes criados
- elegibilidades aprovadas
- elegibilidades recusadas
- limites calculados
- comunicações enviadas

## Tracing

O tracing distribuído será feito com OpenTelemetry.

Objetivo:

- rastrear chamadas entre serviços
- medir tempo por etapa
- identificar gargalos
- conectar logs, métricas e traces

## OpenTelemetry

Cada serviço deve propagar contexto de trace em:

- chamadas HTTP
- eventos Kafka
- mensagens RabbitMQ

Campos esperados:

- traceId
- spanId

## Alertas planejados

Alertas úteis:

- serviço indisponível
- taxa de erro acima do limite
- latência alta
- fila com backlog alto
- mensagens em DLQ
- falha de conexão com banco
- consumer parado
- ausência de consumo em fila crítica

## Boas práticas

- Usar logs estruturados.
- Propagar correlationId.
- Não logar dados sensíveis.
- Criar métricas técnicas e de negócio.
- Criar dashboards simples e úteis.
- Adicionar tracing em chamadas entre serviços.
- Monitorar DLQ.
- Monitorar consumers.
- Garantir actuator habilitado.
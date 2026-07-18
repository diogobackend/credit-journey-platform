# Observabilidade

Este documento descreve a estratégia de observabilidade da **Credit Journey Platform**.

A observabilidade deve permitir investigação de problemas, acompanhamento da saúde dos serviços, rastreabilidade da jornada de crédito e análise operacional da plataforma.

A estratégia será baseada em três pilares:

- Logs;
- Métricas;
- Tracing.

Também haverá padronização técnica por meio da lib compartilhada:

```text
credit-observability-starter
```

---

## Objetivo

A observabilidade da plataforma deve permitir responder perguntas como:

- Qual serviço falhou?
- Qual etapa da jornada de crédito apresentou erro?
- Qual foi o tempo de processamento?
- Qual mensagem foi enviada para DLQ?
- Qual evento foi processado em duplicidade?
- Qual cliente foi impactado?
- Qual correlationId representa a jornada completa?
- Onde está o gargalo entre gateway, orchestrator e microservices?

---

## Pilares da observabilidade

| Pilar | Objetivo |
|---|---|
| Logs | Registrar eventos técnicos e de negócio |
| Métricas | Medir comportamento, volume, latência e erros |
| Tracing | Rastrear chamadas distribuídas entre serviços |

---

## Componentes planejados

| Componente | Responsabilidade |
|---|---|
| Spring Boot Actuator | Expor health, métricas e endpoint Prometheus |
| Prometheus | Coletar métricas dos serviços |
| Grafana | Exibir dashboards |
| OpenTelemetry | Gerar traces distribuídos |
| Jaeger | Visualizar traces |
| Logs JSON | Facilitar busca e ingestão em ferramentas como Splunk |
| credit-observability-starter | Padronizar logs, correlationId, MDC e tracing |

---

## Logs

Os serviços devem gerar logs estruturados.

Formato recomendado:

```text
JSON
```

Campos esperados:

```text
timestamp
level
serviceName
operation
method
traceId
spanId
correlationId
customerId
eventId
messageId
status
duration
errorCode
errorMessage
```

Exemplo conceitual:

```json
{
  "timestamp": "2026-01-01T10:00:00Z",
  "level": "INFO",
  "serviceName": "credit-customer-service",
  "operation": "create-customer",
  "method": "CreateCustomerUseCase.create",
  "traceId": "9f2c8c1d",
  "spanId": "a7b3c9",
  "correlationId": "7c9f4d2e-5c18-4b61-9c2d-111111111111",
  "customerId": "11111111-1111-1111-1111-111111111111",
  "status": "SUCCESS",
  "duration": "120ms"
}
```

---

## Logs automáticos

A plataforma poderá usar annotations para reduzir logs manuais repetidos.

Exemplo:

```kotlin
@LogInfo(logParameters = true, logReturn = true)
fun create(input: CreateCustomerInput): Customer
```

Responsabilidade da lib:

```text
credit-observability-starter
```

Ela poderá fornecer:

- `@LogInfo`;
- `@LogParameter`;
- Aspect para logs automáticos;
- configuração de MDC;
- correlationId filter;
- mascaramento de dados sensíveis;
- padronização dos campos de log.

---

## Correlation ID

Toda requisição ou mensagem deve possuir `correlationId`.

O `correlationId` permite rastrear a mesma jornada entre vários serviços.

Exemplo de fluxo:

```text
1. Cliente chama o credit-api-gateway
2. Gateway cria ou propaga correlationId
3. Orchestrator recebe o mesmo correlationId
4. Customer-service executa operação com o mesmo correlationId
5. Rules-engine-service avalia elegibilidade com o mesmo correlationId
6. Limit-service calcula limite com o mesmo correlationId
7. Communication-service envia comunicação com o mesmo correlationId
8. Audit-service registra a timeline com o mesmo correlationId
```

Header recomendado:

```http
X-Correlation-Id: 7c9f4d2e-5c18-4b61-9c2d-111111111111
```

Regra:

```text
Se a requisição não possuir correlationId, o gateway deve gerar um.
Se já possuir, deve propagar.
```

---

## MDC

O MDC será usado para manter informações comuns no contexto dos logs.

Campos recomendados no MDC:

```text
correlationId
traceId
spanId
serviceName
customerId
eventId
messageId
```

Objetivo:

```text
evitar passar manualmente esses campos em todos os logs
```

---

## Dados sensíveis

Dados sensíveis não devem ser logados em texto puro.

Exemplos de dados que devem ser mascarados:

- CPF;
- documento;
- número de cartão;
- senha;
- token;
- refresh token;
- e-mail completo;
- telefone completo;
- payload com dados pessoais.

Exemplo:

```text
document=123******00
email=j***@email.com
phone=119*****999
```

Regras:

- nunca logar senha;
- nunca logar token completo;
- nunca logar refresh token;
- evitar logar payload completo quando houver dados sensíveis;
- aplicar máscara antes do log.

---

## Splunk

Os logs estruturados devem ser compatíveis com ingestão em Splunk.

Neste portfólio, o Splunk pode ser documentado e simulado com logs JSON.

Campos importantes para busca:

```text
serviceName
correlationId
customerId
eventType
eventId
messageId
status
errorCode
```

Exemplos de investigação:

```text
Buscar jornada por correlationId
Buscar erro por customerId
Buscar mensagens enviadas para DLQ
Buscar falhas de comunicação
Buscar eventos duplicados
Buscar falhas de autenticação
Buscar lentidão no orchestrator
```

Exemplo de busca conceitual:

```text
serviceName="credit-limit-service" correlationId="7c9f4d2e-5c18-4b61-9c2d-111111111111"
```

---

## Métricas

As métricas serão expostas via Spring Boot Actuator e coletadas pelo Prometheus.

Cada serviço deve expor:

```text
/actuator/health
/actuator/metrics
/actuator/prometheus
```

Configuração base esperada:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      probes:
        enabled: true
```

---

## Métricas técnicas

Métricas esperadas:

```text
http_server_requests_seconds_count
http_server_requests_seconds_sum
jvm_memory_used_bytes
jvm_threads_live_threads
process_cpu_usage
system_cpu_usage
hikaricp_connections_active
hikaricp_connections_idle
application_ready_time_seconds
application_started_time_seconds
```

Indicadores importantes:

- quantidade de requests HTTP;
- latência por endpoint;
- taxa de erro HTTP;
- uso de memória;
- uso de CPU;
- conexões com banco;
- status de health check.

---

## Métricas de negócio

Métricas planejadas:

```text
customers_created_total
customers_updated_total
customers_deleted_total
eligibility_approved_total
eligibility_rejected_total
eligibility_manual_analysis_total
limits_calculated_total
limits_updated_total
communications_sent_total
communications_failed_total
credit_journeys_started_total
credit_journeys_completed_total
credit_journeys_failed_total
auth_login_success_total
auth_login_failed_total
```

Objetivo:

```text
medir o comportamento real da jornada de crédito
```

---

## Métricas de mensageria

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

Indicadores importantes:

- mensagens consumidas;
- mensagens publicadas;
- mensagens com erro;
- mensagens em retry;
- mensagens em DLQ;
- tempo médio de processamento;
- backlog de fila;
- lag de consumer Kafka.

---

## Métricas por serviço

### credit-api-gateway

Métricas relevantes:

```text
gateway_requests_total
gateway_request_duration_seconds
gateway_auth_failures_total
gateway_rate_limit_blocked_total
```

---

### credit-auth-service

Métricas relevantes:

```text
auth_login_success_total
auth_login_failed_total
auth_token_generated_total
auth_refresh_token_generated_total
auth_invalid_token_total
```

---

### credit-customer-service

Métricas relevantes:

```text
customers_created_total
customers_updated_total
customers_status_changed_total
customers_deleted_total
customer_events_published_total
```

---

### credit-rules-engine-service

Métricas relevantes:

```text
eligibility_approved_total
eligibility_rejected_total
eligibility_manual_analysis_total
rules_evaluated_total
rules_failed_total
```

---

### credit-limit-service

Métricas relevantes:

```text
limits_calculated_total
limits_updated_total
limits_blocked_total
limits_released_total
limit_calculation_failed_total
```

---

### credit-orchestrator-service

Métricas relevantes:

```text
credit_journeys_started_total
credit_journeys_completed_total
credit_journeys_failed_total
credit_journey_duration_seconds
credit_journey_step_failed_total
```

---

### credit-communication-service

Métricas relevantes:

```text
communications_requested_total
communications_sent_total
communications_failed_total
communications_discarded_total
communication_retries_total
communication_dlq_total
```

---

### credit-audit-service

Métricas relevantes:

```text
audit_events_registered_total
audit_events_failed_total
audit_timeline_created_total
audit_duplicate_events_total
```

---

## Grafana

O Grafana será usado para dashboards.

Dashboards planejados:

- visão geral da plataforma;
- mensageria;
- jornada de crédito;
- autenticação;
- banco de dados;
- JVM e infraestrutura.

---

## Dashboard: Visão geral da plataforma

Deve mostrar:

- saúde dos serviços;
- taxa de erro;
- latência média;
- throughput;
- uso de CPU;
- uso de memória;
- status dos bancos;
- status dos brokers.

---

## Dashboard: Mensageria

Deve mostrar:

- mensagens publicadas por tópico;
- mensagens consumidas por serviço;
- mensagens com erro;
- mensagens em retry;
- mensagens em DLQ;
- filas com backlog;
- lag de consumers Kafka;
- eventos pendentes na outbox;
- eventos duplicados ignorados pela inbox.

---

## Dashboard: Jornada de crédito

Deve mostrar:

- jornadas iniciadas;
- jornadas concluídas;
- jornadas com falha;
- clientes criados;
- elegibilidades aprovadas;
- elegibilidades recusadas;
- limites calculados;
- comunicações enviadas;
- comunicações com falha;
- tempo médio da jornada.

---

## Dashboard: Autenticação

Deve mostrar:

- logins com sucesso;
- logins com falha;
- tokens emitidos;
- tokens inválidos;
- falhas por role/permissão;
- taxa de erro do auth-service.

---

## Tracing

O tracing distribuído será feito com OpenTelemetry.

Objetivo:

- rastrear chamadas entre serviços;
- medir tempo por etapa;
- identificar gargalos;
- conectar logs, métricas e traces;
- visualizar a jornada completa de uma requisição.

---

## OpenTelemetry

Cada serviço deve propagar contexto de trace em:

- chamadas HTTP;
- eventos Kafka;
- mensagens RabbitMQ.

Campos esperados:

```text
traceId
spanId
```

Exemplo de fluxo rastreável:

```text
credit-api-gateway
   -> credit-orchestrator-service
      -> credit-customer-service
      -> credit-rules-engine-service
      -> credit-limit-service
      -> credit-communication-service
      -> credit-audit-service
```

---

## Jaeger

O Jaeger poderá ser usado para visualizar traces localmente.

Porta planejada:

```text
16686
```

Objetivo:

```text
visualizar o tempo gasto em cada serviço durante uma jornada
```

---

## Health checks

Todos os serviços devem expor health check.

Endpoints esperados:

```text
/actuator/health
/actuator/health/readiness
/actuator/health/liveness
```

Uso:

| Endpoint | Finalidade |
|---|---|
| `/actuator/health` | Estado geral |
| `/actuator/health/readiness` | Serviço pronto para receber tráfego |
| `/actuator/health/liveness` | Serviço vivo |

---

## Alertas planejados

Alertas úteis:

- serviço indisponível;
- taxa de erro acima do limite;
- latência alta;
- fila com backlog alto;
- mensagens em DLQ;
- falha de conexão com banco;
- consumer parado;
- ausência de consumo em fila crítica;
- outbox acumulando eventos pendentes;
- falha de autenticação acima do normal;
- jornada de crédito com falha acima do limite;
- uso de memória alto;
- uso de CPU alto.

---

## Estratégia de erro observável

| Cenário | Observabilidade esperada |
|---|---|
| Erro HTTP | Log com status, endpoint, errorCode e correlationId |
| Erro em consumer | Log com eventId/messageId, fila/tópico e correlationId |
| Retry | Métrica de retry e log com attempt |
| DLQ | Métrica de DLQ e log com motivo |
| Evento duplicado | Log informativo e métrica de duplicidade |
| Falha de autenticação | Métrica e log sem expor senha/token |
| Falha de banco | Log com erro técnico e métrica de erro |
| Falha no orchestrator | Log da etapa com erro e correlationId |

---

## Boas práticas

- Usar logs estruturados.
- Usar formato JSON.
- Propagar correlationId.
- Usar MDC.
- Não logar dados sensíveis.
- Mascarar documento, telefone, e-mail e tokens.
- Criar métricas técnicas.
- Criar métricas de negócio.
- Criar métricas de mensageria.
- Criar dashboards simples e úteis.
- Adicionar tracing em chamadas entre serviços.
- Monitorar DLQ.
- Monitorar consumers.
- Monitorar outbox.
- Garantir actuator habilitado.
- Usar health, readiness e liveness.
- Padronizar logs com `credit-observability-starter`.
- Relacionar logs, métricas e traces por correlationId.

---

## Objetivo da observabilidade

A estratégia de observabilidade deve permitir:

- investigar falhas rapidamente;
- rastrear uma jornada de ponta a ponta;
- identificar gargalos;
- monitorar saúde dos serviços;
- acompanhar eventos de negócio;
- detectar falhas em mensageria;
- monitorar autenticação;
- apoiar suporte operacional;
- simular práticas reais de produção.
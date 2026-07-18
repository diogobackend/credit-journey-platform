# Deployment

Este documento descreve a estratégia de execução local e deploy da **Credit Journey Platform**.

A plataforma será preparada para execução local com Docker Compose e, futuramente, para deploy em Kubernetes.

O objetivo é demonstrar uma visão próxima de um ambiente produtivo, mesmo sendo um projeto de portfólio.

---

## Visão geral

A plataforma é composta por microservices independentes, repositórios de apoio e infraestrutura compartilhada.

Principais componentes executáveis:

- `credit-api-gateway`;
- `credit-auth-service`;
- `credit-customer-service`;
- `credit-limit-service`;
- `credit-rules-engine-service`;
- `credit-orchestrator-service`;
- `credit-communication-service`;
- `credit-audit-service`;
- `credit-config-server`.

Componentes de apoio:

- `credit-platform-infra`;
- `credit-shared-contracts`;
- `credit-observability-starter`.

Observação:

```text
credit-observability-starter não é um serviço executável.
É uma lib compartilhada usada pelos microservices.
```

---

## Ambientes planejados

| Ambiente | Finalidade |
|---|---|
| `local` | Desenvolvimento na máquina |
| `dev` | Simulação de ambiente integrado |
| `qa` | Testes e validações |
| `prod` | Referência conceitual de produção |

---

## Execução local

A execução local será centralizada no repositório:

```text
credit-platform-infra
```

Esse repositório conterá o `docker-compose.yml` com a infraestrutura necessária para subir a plataforma localmente.

---

## Componentes locais

| Componente | Porta |
|---|---|
| `credit-api-gateway` | `8080` |
| `credit-customer-service` | `8081` |
| `credit-auth-service` | `8082` |
| `credit-limit-service` | `8083` |
| `credit-rules-engine-service` | `8084` |
| `credit-orchestrator-service` | `8085` |
| `credit-communication-service` | `8086` |
| `credit-audit-service` | `8087` |
| `credit-config-server` | `8888` |
| MySQL | `3306` |
| Kafka | `9092` |
| RabbitMQ | `5672` |
| RabbitMQ Management | `15672` |
| Prometheus | `9090` |
| Grafana | `3000` |
| OpenTelemetry Collector | `4317 / 4318` |
| Jaeger | `16686` |

---

## Bancos locais

A estratégia é `database per service`.

Cada serviço terá seu próprio banco.

| Serviço | Banco |
|---|---|
| `credit-auth-service` | `auth_db` |
| `credit-customer-service` | `customer_db` |
| `credit-limit-service` | `limit_db` |
| `credit-rules-engine-service` | `rules_db` |
| `credit-orchestrator-service` | `orchestrator_db` |
| `credit-communication-service` | `communication_db` |
| `credit-audit-service` | `audit_db` |

---

## Docker

Cada microservice deve possuir seu próprio `Dockerfile`.

Regras:

- usar Java 21;
- usar imagem base leve;
- não embutir configuração sensível na imagem;
- não versionar secrets;
- configurar a aplicação por variável de ambiente ou Config Server;
- expor actuator health;
- suportar graceful shutdown;
- expor métricas para Prometheus;
- manter imagem pequena e reproduzível.

---

## Exemplo conceitual de Dockerfile

```dockerfile
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

COPY build/libs/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## Docker Compose

O Docker Compose será usado para subir localmente:

- MySQL;
- Kafka;
- RabbitMQ;
- Prometheus;
- Grafana;
- OpenTelemetry Collector;
- Jaeger;
- Config Server;
- microservices da plataforma.

---

## Comandos principais

Subir ambiente local:

```bash
docker compose up -d
```

Parar ambiente local:

```bash
docker compose down
```

Parar e remover volumes:

```bash
docker compose down -v
```

Ver logs:

```bash
docker compose logs -f
```

Ver logs de um serviço específico:

```bash
docker compose logs -f credit-customer-service
```

---

## Ordem recomendada de subida local

Ordem conceitual:

```text
1. MySQL
2. Kafka
3. RabbitMQ
4. OpenTelemetry Collector
5. Prometheus
6. Grafana
7. Config Server
8. Auth Service
9. Customer Service
10. Limit Service
11. Rules Engine Service
12. Orchestrator Service
13. Communication Service
14. Audit Service
15. API Gateway
```

O `credit-api-gateway` deve subir depois dos serviços internos principais, pois ele roteia chamadas para eles.

---

## Configuração local

As configurações locais poderão vir de:

- `application-local.yml`;
- variáveis de ambiente;
- Docker Compose;
- Config Server.

Exemplo de profile:

```text
SPRING_PROFILES_ACTIVE=local
```

---

## Variáveis de ambiente comuns

Exemplos:

```text
SPRING_PROFILES_ACTIVE
SERVER_PORT
SPRING_DATASOURCE_URL
SPRING_DATASOURCE_USERNAME
SPRING_DATASOURCE_PASSWORD
SPRING_KAFKA_BOOTSTRAP_SERVERS
SPRING_RABBITMQ_HOST
SPRING_RABBITMQ_PORT
MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE
OTEL_EXPORTER_OTLP_ENDPOINT
```

---

## Configurações sensíveis

Configurações sensíveis não devem ser versionadas no GitHub.

Exemplos:

- senha do banco;
- secrets de JWT;
- tokens;
- credenciais externas;
- senhas de mensageria;
- chaves privadas.

Em ambiente local, podem ser usadas variáveis de ambiente ou arquivos `.env` não versionados.

---

## Arquivo .env

O projeto poderá usar `.env` para facilitar a execução local.

Exemplo:

```env
MYSQL_ROOT_PASSWORD=root
MYSQL_USER=credit_user
MYSQL_PASSWORD=credit_pass

JWT_SECRET=local-development-secret

RABBITMQ_DEFAULT_USER=guest
RABBITMQ_DEFAULT_PASS=guest
```

O arquivo real `.env` não deve ser versionado.

Um exemplo seguro pode ser versionado como:

```text
.env.example
```

---

## Kubernetes

Futuramente, cada serviço terá seus manifests Kubernetes.

Recursos planejados por serviço:

- `Deployment`;
- `Service`;
- `Ingress`;
- `ConfigMap`;
- `Secret`;
- `HorizontalPodAutoscaler`;
- `ServiceMonitor`.

---

## Deployment

O `Deployment` será responsável por controlar os pods da aplicação.

Deve conter:

- imagem do serviço;
- variáveis de ambiente;
- requests e limits;
- probes;
- estratégia de rolling update;
- número inicial de réplicas;
- configuração de graceful shutdown.

---

## Service

O `Service` expõe a aplicação dentro do cluster.

Tipo padrão:

```text
ClusterIP
```

O acesso externo deve ser feito pelo Ingress ou pelo API Gateway.

---

## Ingress

O `Ingress` será responsável por expor endpoints HTTP externamente em ambiente controlado.

Rotas externas planejadas devem apontar preferencialmente para o `credit-api-gateway`.

Exemplos:

```text
/api/v1/auth/**
/api/v1/customers/**
/api/v1/limits/**
/api/v1/credit-journeys/**
/api/v1/audits/**
```

O Ingress não deve rotear diretamente para todos os serviços de negócio se o API Gateway estiver sendo usado como entrada única.

---

## API Gateway no deploy

O `credit-api-gateway` será a entrada principal para chamadas externas.

Responsabilidades no deploy:

- receber tráfego externo;
- validar JWT;
- aplicar CORS;
- aplicar rate limit;
- propagar headers;
- propagar correlationId;
- rotear chamadas para os serviços internos.

Fluxo esperado:

```text
Cliente
   |
   v
Ingress
   |
   v
credit-api-gateway
   |
   v
microservices internos
```

---

## ConfigMap

O `ConfigMap` será usado para configurações não sensíveis.

Exemplos:

- nome da aplicação;
- profile ativo;
- URL do Config Server;
- nível de log;
- parâmetros não sensíveis;
- configuração de endpoints internos.

---

## Secret

O `Secret` será usado para configurações sensíveis.

Exemplos:

- senha do banco;
- usuário do banco;
- JWT secret;
- tokens;
- credenciais externas.

Regras:

- não versionar secrets reais;
- usar placeholders em manifests de exemplo;
- preferir integração com gerenciadores de segredo em ambiente real.

---

## Probes

Cada serviço deve possuir probes.

---

### Readiness Probe

Indica se o serviço está pronto para receber tráfego.

Endpoint esperado:

```text
/actuator/health/readiness
```

Uso:

```text
evitar envio de tráfego para serviço ainda não pronto
```

---

### Liveness Probe

Indica se o serviço está vivo.

Endpoint esperado:

```text
/actuator/health/liveness
```

Uso:

```text
reiniciar o pod quando a aplicação travar
```

---

### Startup Probe

Pode ser usada em serviços com inicialização mais demorada.

Uso:

```text
evitar que o Kubernetes mate o container antes da aplicação terminar de subir
```

---

## Actuator

Todos os serviços devem expor endpoints técnicos via Spring Boot Actuator.

Endpoints esperados:

```text
/actuator/health
/actuator/health/readiness
/actuator/health/liveness
/actuator/metrics
/actuator/prometheus
```

Exposição mínima recomendada:

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

## Recursos

Cada serviço deve definir requests e limits.

Exemplo conceitual:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "512Mi"
  limits:
    cpu: "500m"
    memory: "1Gi"
```

Objetivo:

- evitar consumo descontrolado;
- permitir melhor agendamento no cluster;
- demonstrar preocupação com operação e estabilidade.

---

## Horizontal Pod Autoscaler

Serviços críticos podem usar HPA.

Critérios possíveis:

- CPU;
- memória;
- métricas customizadas;
- backlog de fila;
- taxa de requisições.

Serviços candidatos:

- `credit-api-gateway`;
- `credit-orchestrator-service`;
- `credit-communication-service`;
- `credit-rules-engine-service`.

---

## Estratégia de configuração

A configuração deve vir de:

- Config Server;
- variáveis de ambiente;
- ConfigMap;
- Secret.

Nenhuma senha deve ser versionada no GitHub.

Configuração sensível deve ficar fora do código.

---

## Config Server

O `credit-config-server` será responsável por centralizar configurações dos serviços.

Exemplos de configurações:

- URLs de serviços;
- tópicos Kafka;
- filas RabbitMQ;
- feature toggles;
- níveis de log;
- parâmetros técnicos;
- configurações por ambiente.

---

## Observabilidade no deploy

A plataforma deve ser preparada para observabilidade distribuída.

Componentes planejados:

- Prometheus;
- Grafana;
- OpenTelemetry Collector;
- Jaeger;
- logs estruturados;
- correlationId;
- tracing distribuído.

---

## Prometheus

Responsável por coletar métricas dos serviços.

Fonte principal:

```text
/actuator/prometheus
```

Métricas esperadas:

```text
requests_total
request_duration_seconds
errors_total
messages_consumed_total
messages_failed_total
messages_dlq_total
```

---

## Grafana

Responsável por dashboards.

Dashboards planejados:

- saúde dos serviços;
- latência;
- taxa de erro;
- throughput;
- backlog de filas;
- mensagens em DLQ;
- falhas de autenticação;
- jornadas processadas.

---

## OpenTelemetry

O OpenTelemetry será usado para tracing distribuído.

Objetivo:

```text
rastrear uma requisição passando por gateway, orchestrator e microservices
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

## Logs

Os serviços devem gerar logs estruturados.

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

Parte da padronização será centralizada na lib:

```text
credit-observability-starter
```

---

## Estratégia de build

Cada repositório de serviço deve ter seu próprio pipeline.

Etapas esperadas:

```text
1. Checkout
2. Setup Java 21
3. Build
4. Testes
5. Ktlint
6. JaCoCo
7. Análise estática
8. Build da imagem Docker
9. Push da imagem
10. Deploy
```

---

## Estratégia de versionamento de imagem

Formato sugerido para local:

```text
service-name:latest
```

Formato sugerido para release:

```text
service-name:1.0.0
```

Formato sugerido para rastreabilidade:

```text
service-name:commit-sha
```

Exemplos:

```text
credit-customer-service:latest
credit-customer-service:1.0.0
credit-customer-service:a1b2c3d
```

---

## Estratégia de deploy

Estratégia padrão:

```text
Rolling Update
```

Objetivo:

- evitar indisponibilidade;
- substituir pods gradualmente;
- permitir rollback;
- reduzir risco operacional.

---

## Graceful shutdown

Os serviços devem suportar encerramento controlado.

Objetivo:

- parar de receber novas requisições;
- finalizar requisições em andamento;
- evitar perda de mensagens;
- encerrar conexões corretamente.

Configuração conceitual:

```yaml
server:
  shutdown: graceful

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

---

## Health check por serviço

Todos os serviços devem responder health check.

Exemplo:

```text
GET /actuator/health
```

Resposta esperada:

```json
{
  "status": "UP"
}
```

---

## Deploy dos serviços

Ordem conceitual de deploy:

```text
1. Infraestrutura base
2. Config Server
3. Auth Service
4. Serviços de domínio
5. Orchestrator Service
6. API Gateway
7. Observabilidade
```

Detalhado:

```text
1. MySQL
2. Kafka
3. RabbitMQ
4. Prometheus
5. Grafana
6. OpenTelemetry Collector
7. Jaeger
8. credit-config-server
9. credit-auth-service
10. credit-customer-service
11. credit-limit-service
12. credit-rules-engine-service
13. credit-communication-service
14. credit-audit-service
15. credit-orchestrator-service
16. credit-api-gateway
```

---

## Boas práticas de deploy

- Usar readiness probe.
- Usar liveness probe.
- Usar startup probe quando necessário.
- Usar graceful shutdown.
- Usar rolling update.
- Não versionar secrets.
- Não usar configuração fixa no código.
- Definir requests e limits.
- Separar configuração por ambiente.
- Expor métricas para Prometheus.
- Centralizar logs estruturados.
- Propagar correlationId.
- Usar imagens versionadas.
- Evitar usar apenas `latest` fora do ambiente local.
- Garantir compatibilidade com deploy gradual.
- Não expor serviços internos diretamente sem necessidade.
- Usar API Gateway como entrada principal.
- Manter banco por serviço.
- Executar migrations com controle.

---

## Objetivo da estratégia de deployment

A estratégia de deployment deve permitir:

- execução local simples;
- simulação de ambiente integrado;
- deploy controlado;
- rastreabilidade;
- observabilidade;
- isolamento entre serviços;
- configuração externa;
- segurança de credenciais;
- preparação para Kubernetes;
- aproximação com práticas reais de produção.
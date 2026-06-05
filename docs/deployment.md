# Deployment

A Credit Journey Platform será preparada para execução local com Docker Compose e para deploy em Kubernetes.

O objetivo é demonstrar uma visão próxima de ambiente produtivo, mesmo sendo um projeto de portfólio.

## Ambientes planejados

| Ambiente | Finalidade |
|---|---|
| local | Desenvolvimento na máquina |
| dev | Simulação de ambiente integrado |
| qa | Testes e validações |
| prod | Referência conceitual de produção |

## Execução local

A execução local será centralizada no repositório:

- credit-platform-infra

Esse repositório conterá o docker-compose.yml com a infraestrutura necessária.

## Componentes locais

| Componente | Porta |
|---|---|
| MySQL | 3306 |
| Kafka | 9092 |
| RabbitMQ | 5672 |
| RabbitMQ Management | 15672 |
| Prometheus | 9090 |
| Grafana | 3000 |
| Config Server | 8888 |
| OpenTelemetry Collector | 4317 / 4318 |

## Docker

Cada microservice deve possuir seu próprio Dockerfile.

Regras:

- Usar imagem base leve.
- Usar Java 21.
- Não embutir configuração sensível na imagem.
- Configurações devem vir por variável de ambiente ou Config Server.
- A aplicação deve expor actuator health.
- A aplicação deve suportar graceful shutdown.

## Docker Compose

O Docker Compose será usado para subir:

- bancos MySQL
- Kafka
- RabbitMQ
- Prometheus
- Grafana
- OpenTelemetry Collector
- Config Server
- microservices da plataforma

Comando esperado:

docker compose up -d

Para parar:

docker compose down

Para limpar volumes:

docker compose down -v

## Kubernetes

Cada serviço terá seus manifests Kubernetes.

Recursos planejados por serviço:

- Deployment
- Service
- Ingress
- ConfigMap
- Secret
- HorizontalPodAutoscaler
- ServiceMonitor

## Deployment

Responsável por controlar os pods da aplicação.

Deve conter:

- imagem do serviço
- variáveis de ambiente
- limites de CPU e memória
- probes
- estratégia de rolling update

## Service

Responsável por expor a aplicação dentro do cluster.

Tipo padrão:

- ClusterIP

## Ingress

Responsável por expor endpoints HTTP externamente em ambiente controlado.

Exemplos:

- /customers
- /limits
- /communications
- /audits

## ConfigMap

Usado para configurações não sensíveis.

Exemplos:

- nome da aplicação
- profile ativo
- URL do Config Server
- nível de log
- parâmetros não sensíveis

## Secret

Usado para configurações sensíveis.

Exemplos:

- senha do banco
- usuário do banco
- tokens
- credenciais externas

## Probes

Cada serviço deve possuir probes.

### Readiness Probe

Indica se o serviço está pronto para receber tráfego.

Exemplo de endpoint:

/actuator/health/readiness

### Liveness Probe

Indica se o serviço está vivo.

Exemplo de endpoint:

/actuator/health/liveness

### Startup Probe

Pode ser usada em serviços com inicialização mais demorada.

## Recursos

Cada serviço deve definir requests e limits.

Exemplo conceitual:

- CPU request
- CPU limit
- Memory request
- Memory limit

Isso demonstra preocupação com operação e estabilidade.

## Horizontal Pod Autoscaler

Serviços críticos podem usar HPA.

Critérios possíveis:

- CPU
- memória
- métricas customizadas
- backlog de fila

## Estratégia de configuração

A configuração deve vir de:

- Config Server
- variáveis de ambiente
- ConfigMap
- Secret

Nenhuma senha deve ser versionada no GitHub.

## Estratégia de build

Cada repositório de serviço deve ter seu próprio pipeline.

Etapas esperadas:

1. Checkout
2. Build
3. Testes
4. Análise estática
5. Build da imagem Docker
6. Push da imagem
7. Deploy

## Estratégia de versionamento de imagem

Formato sugerido:

- service-name:latest para local
- service-name:1.0.0 para releases
- service-name:commit-sha para rastreabilidade

## Boas práticas de deploy

- Usar readiness/liveness probes.
- Usar graceful shutdown.
- Usar rolling update.
- Não versionar secrets.
- Não usar configuração fixa no código.
- Definir requests e limits.
- Separar configuração por ambiente.
- Expor métricas para Prometheus.
- Centralizar logs estruturados.
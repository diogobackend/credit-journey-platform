# Banco de Dados

A Credit Journey Platform utiliza a estratégia database per service.

Cada microservice possui seu próprio banco ou schema, evitando acoplamento direto entre domínios.

## Estratégia principal

Cada serviço é dono dos seus próprios dados.

Um serviço não deve acessar diretamente o banco de outro serviço.

A comunicação entre serviços deve ocorrer por:

- API REST
- Eventos Kafka
- Filas RabbitMQ

## Bancos planejados

| Serviço | Banco |
|---|---|
| credit-customer-service | customer_db |
| credit-rules-engine-service | rules_db |
| credit-limit-service | limit_db |
| credit-communication-service | communication_db |
| credit-audit-service | audit_db |

## MySQL

O banco escolhido para o portfólio será MySQL.

Motivos:

- Simples de executar localmente
- Fácil integração com Spring Boot
- Bom suporte com Docker
- Compatível com Flyway
- Suficiente para demonstrar persistência relacional

## Flyway

As alterações de banco serão versionadas com Flyway.

Cada serviço terá sua própria pasta de migrations.

Estrutura esperada:

src/main/resources/db/migration/

Exemplo:

- V1__create_customers_table.sql
- V2__create_customer_status_history_table.sql
- V3__create_outbox_events_table.sql

## Regras para migrations

- Nunca alterar migration já aplicada.
- Criar uma nova migration para cada mudança.
- Usar nomes claros.
- Evitar mudanças destrutivas.
- Separar mudanças estruturais de inserts de carga inicial.
- Garantir compatibilidade com deploy gradual quando possível.

## credit-customer-service

Banco:

- customer_db

Tabelas planejadas:

### customers

Armazena dados principais do cliente.

Campos esperados:

- id
- name
- document
- email
- phone
- income
- status
- created_at
- updated_at

### customer_status_history

Armazena histórico de alteração de status.

Campos esperados:

- id
- customer_id
- previous_status
- current_status
- reason
- changed_at

### outbox_events

Armazena eventos a serem publicados.

Campos esperados:

- id
- event_id
- event_type
- aggregate_id
- payload
- status
- created_at
- published_at

## credit-rules-engine-service

Banco:

- rules_db

Tabelas planejadas:

### credit_policies

Armazena políticas de crédito configuráveis.

Campos esperados:

- id
- policy_code
- description
- active
- created_at
- updated_at

### eligibility_decisions

Armazena decisões de elegibilidade.

Campos esperados:

- id
- customer_id
- decision
- reason
- score
- evaluated_at

### rule_evaluation_history

Armazena detalhes das regras avaliadas.

Campos esperados:

- id
- customer_id
- rule_code
- result
- reason
- evaluated_at

## credit-limit-service

Banco:

- limit_db

Tabelas planejadas:

### credit_limits

Armazena o limite atual do cliente.

Campos esperados:

- id
- customer_id
- total_limit
- available_limit
- used_limit
- status
- created_at
- updated_at

### limit_change_history

Armazena histórico de alterações de limite.

Campos esperados:

- id
- limit_id
- previous_total_limit
- new_total_limit
- change_type
- reason
- changed_at

### outbox_events

Armazena eventos de limite a publicar.

### inbox_events

Armazena eventos já consumidos para idempotência.

## credit-communication-service

Banco:

- communication_db

Tabelas planejadas:

### communication_templates

Armazena templates de comunicação.

Campos esperados:

- id
- template_code
- channel
- subject
- body
- active
- created_at
- updated_at

### communication_requests

Armazena solicitações de comunicação.

Campos esperados:

- id
- customer_id
- template_code
- channel
- status
- correlation_id
- created_at
- updated_at

### communication_delivery_history

Armazena histórico de tentativas de envio.

Campos esperados:

- id
- communication_request_id
- attempt
- status
- error_message
- sent_at

## credit-audit-service

Banco:

- audit_db

Tabelas planejadas:

### audit_events

Armazena eventos auditados.

Campos esperados:

- id
- event_id
- event_type
- source
- customer_id
- correlation_id
- occurred_at
- created_at

### customer_timeline

Armazena visão de timeline do cliente.

Campos esperados:

- id
- customer_id
- event_type
- description
- occurred_at
- correlation_id

### event_payloads

Armazena payload bruto do evento.

Campos esperados:

- id
- event_id
- payload
- created_at

## Boas práticas

- Usar UUID como identificador externo.
- Usar BigDecimal para valores monetários.
- Usar índices em campos de busca frequente.
- Não compartilhar tabelas entre serviços.
- Não usar joins entre bancos de serviços diferentes.
- Não expor Entity JPA fora do adapter de persistência.
- Separar modelo de domínio de modelo de banco.
- Versionar toda mudança com Flyway.
- Usar created_at e updated_at em tabelas transacionais.
- Usar tabelas append-only para auditoria.
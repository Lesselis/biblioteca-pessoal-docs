# ADR 0001: Escolha da Arquitetura de Microsserviços

## Status
Aprovado. [2025-04-10]

## Contexto
Requisitos não-funcionais críticos:
- Independência de deploy por componente
- Tolerância a falhas parcial
- Onboarding rápido de novos recursos

## Decisão
```mermaid
flowchart TD
  A[Frontend SPA] --> B[API Gateway]
  B --> C[Service Discovery]
  C --> D[usuarios]
  C --> E[catalogo]
  C --> F[diel-indexacao]
  C --> G[auditoria]
  D -->|PostgreSQL| H[(PostgreSQL)]
  E -->|MongoDB| I[(MongoDB)]
  G -->|Elasticsearch| J[(Elastic)]
  
  subgraph Infraestrutura
    K[Config Server]
    L[Circuit Breaker]
    M[RabbitMQ]
  end
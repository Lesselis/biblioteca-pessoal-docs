# ADR 0003: Padrões de Comunicação entre Serviços

## Status
Aprovado. [2025-04-10]

## Contexto
Requisitos derivados do ADR 0001:
- Necessidade de baixa latência em chamadas síncronas (usuário → catálogo)
- Alta disponibilidade para processamento assíncrono (indexação DIEL)
- Resilência a falhas entre serviços

## Decisão

### Protocolos por Cenário
| Tipo de Comunicação       | Protocolo | Tecnologia          | Padrão de Contrato          | Timeout  |
|---------------------------|-----------|---------------------|-----------------------------|----------|
| Frontend → API Gateway    | HTTP/2    | REST                | OpenAPI 3.1                 | 2s       |
| Gateway → Microsserviços  | gRPC      | Protocol Buffers v3  | .proto files                | 1s       |
| Eventos entre serviços    | AMQP      | RabbitMQ            | JSON Schema                 | N/A      |

### Exemplo de Chamada Síncrona (gRPC)
```protobuf
// users.proto
service UserService {
  rpc GetUser (UserRequest) returns (UserResponse) {
    option (google.api.http) = {
      get: "/v1/users/{user_id}"
    };
  }
}
```

### Exemplo de Evento Assíncrono
```protobuf
// Evento de Atualização de Catálogo (RabbitMQ)
{
  "event_id": "urn:uuid:...",
  "event_type": "ObraAtualizada",
  "data": {
    "obra_id": "123e4567-e89b-12d3-a456-426614174000",
    "campos_alterados": ["titulo"]
  }
}
```
## Políticas de Resilência
| Cenário                | Estratégia                          | Fallback                   |
|------------------------|-------------------------------------|----------------------------|
| Falha gRPC             | Retry (3x, backoff exponencial)     | Cache local (stale-while-revalidate) |
| Falha RabbitMQ         | DLQ + Reprocessamento manual        | Log em PostgreSQL          |
| Timeout                | Circuit Breaker (Resilience4j)      | Fail Fast                  |

## Consequências
- ✅ **Benefícios**:
  - Performance otimizada (gRPC 5-10x mais rápido que REST)
  - Desacoplamento via eventos (RabbitMQ)

- ⚠️ **Riscos**:
  - Complexidade em debug (necessidade de tracing distribuído)
  - Overhead de serialização Protobuf

- 🔄 **Impactos Operacionais**:
  - Necessidade de sidecar proxy para gRPC-web
  - Monitoramento de filas RabbitMQ
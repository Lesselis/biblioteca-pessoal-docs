# ADR 0004: Estratégia de Autenticação e Autorização

## Status  
Aprovado. [2025-04-10]  

## Contexto  
Requisitos derivados do ADR 0001 e necessidades de segurança:  
- Controle de acesso granular aos microsserviços  
- Baixa latência em verificações de permissão  
- Compatibilidade com arquitetura multi-protocolo (gRPC/HTTP)  

## Decisão  

### Componentes Críticos  
| Componente           | Tecnologia               | Responsabilidade                          |  
|----------------------|--------------------------|-------------------------------------------|  
| Identity Provider    | Keycloak                 | Gerenciamento centralizado de identidades |  
| Token Service        | Node.js                  | Geração/validação de JWT                  |  
| Policy Engine        | Open Policy Agent (OPA)  | Avaliação de políticas                    |  

### Padronização Técnica  
1. **Tokens**:  
   - Formato: JWT (RFC 7519)  
   - Algoritmo: RS256  
   - Claims obrigatórias:  
     ```json  
     {  
       "sub": "uuidv4",  
       "preferred_username": "user@email.com",  
       "roles": ["USER", "CATALOG_EDITOR"],  
       "iss": "urn:library-auth",  
       "exp": 1735689600  
     }  
     ```  

2. **Fluxos**:  
   - Autenticação: OpenID Connect  
   - Autorização: RBAC + ABAC via OPA  

## Políticas de Segurança  
| Cenário                  | Regra                              | Ação                          |  
|--------------------------|------------------------------------|-------------------------------|  
| Token expirado           | `exp < now()`                      | 401 + Header `WWW-Authenticate`|  
| Acesso a recurso privado | `roles ∩ resource_roles ≠ ∅`       | 403                           |  
| Bruteforce               | `>5 tentativas/min`                | Block IP (30min)              |  

## Consequências  
- ✅ **Benefícios**:  
  - SSO para futuras integrações  
  - Controle centralizado de políticas  
- ⚠️ **Riscos**:  
  - SPOF no Keycloak (mitigação: cluster HA)  
  - Complexidade no gerenciamento de políticas  
- 🔄 **Impactos Operacionais**:  
  - Necessidade de rotação periódica de chaves RSA  
  - Auditoria mensal de tokens ativos  

## Alinhamento com ADRs Existentes  
1. **Gateway (ADR 0001)**:  
   - Implementará validação JWT pré-roteamento  
   - Converterá headers HTTP para metadata gRPC  

2. **Comunicação (ADR 0003)**:  
   - Claims propagadas via gRPC interceptors  
   - Eventos de segurança publicados no RabbitMQ  

3. **Persistência (ADR 0002)**:  
   - Logs de auditoria armazenados no Elasticsearch  
   - Chaves RSA armazenadas no Hashicorp Vault  
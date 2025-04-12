# ADR 0002: Estratégia de Persistência de Dados

## Status
Aprovado. [2025-04-10]

## Contexto
Requisitos derivados do ADR 0001:
- **Serviço de Usuários**: Dados transacionais e relacionais (ACID)
- **Serviço de Catálogo/DIEL**: Schemas flexíveis e evolutivos
- **Serviço de Auditoria**: Alta capacidade de escrita/consulta (logs)

## Decisão
| Serviço          | Banco        | Tecnologia de Acesso       | Migrações               | Justificativa Técnica                          |
|------------------|-------------|---------------------------|-------------------------|-----------------------------------------------|
| `usuarios`       | PostgreSQL  | Sequelize (Node.js)       | Sequelize CLI           | Transações complexas e relações rigidamente definidas |
| `catalogo`       | MongoDB     | Mongoose (Node.js)        | Mongoose Migrations     | Schema-less para evolução dinâmica de modelos |
| `auditoria`      | Elasticsearch| Elasticsearch Client     | N/A (index templates)  | Full-text search e análise de logs            |

## Padrões de Implementação
### PostgreSQL (usuários)
```javascript
// Modelo Sequelize
User.init({
  id: { type: DataTypes.UUID, primaryKey: true },
  email: { type: DataTypes.STRING, unique: true }
}, { sequelize });
```

### MongoDB (catálogo)
```javascript
// Schema Mongoose
const obraSchema = new Schema({
  titulo: { type: String, required: true },
  metadados: { type: Mixed } // Campo flexível para DIEL
});
```

## Políticas Operacionais
### Backups:

 - PostgreSQL: Backup diário + WAL archiving

 - MongoDB: Ops Manager com snapshots horários

 - Elasticsearch: Snapshots S3 a cada 6h

### Monitoramento:

 - PostgreSQL: pg_stat_activity + alertas de long-running queries

 - MongoDB: Atlas Performance Advisor

 - Elasticsearch: Cluster Health API

## Consequências
✅ Benefícios:

 - Escolhas otimizadas para cada domínio

 - Isolamento de falhas por banco de dados

⚠️ Riscos:

 - Dificuldade em transações distribuídas (Saga Pattern necessário)

 - Conhecimento multiplataforma requerido

🔄 Impactos Operacionais:

 - Diferentes estratégias de tuning para cada banco

 - Necessidade de scripts de recovery específicos
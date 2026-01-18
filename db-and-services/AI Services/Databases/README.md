# Database Integration Guide

This directory contains documentation for integrating and managing databases within AI services and Choreo applications.

---

## Overview

Databases are essential for AI applications to store:
- Conversation history and context
- User data and preferences
- Vector embeddings for semantic search
- Training data and feedback
- Application state and configurations

Choreo provides managed database solutions with built-in security, scalability, and high availability.

---

## Contents

- **[Choreo-Managed_Databases.md](./Choreo-Managed_Databases.md)** - Complete guide to Choreo's managed database offerings

---

## Supported Database Types

### Relational Databases
- **PostgreSQL** - Advanced open-source relational database
- **MySQL** - Popular open-source relational database
- **MariaDB** - MySQL-compatible database with enhanced features

### NoSQL Databases
- **MongoDB** - Document-oriented NoSQL database
- **Redis** - In-memory data store for caching and sessions

### Vector Databases (for AI/ML)
- **Pinecone** - Vector database for semantic search
- **Weaviate** - AI-native vector database
- **Qdrant** - Vector search engine

---

## Quick Start

### 1. Create a Managed Database

```bash
# Using Choreo CLI
choreo database create \
  --name my-ai-database \
  --type postgresql \
  --version 15 \
  --storage 20GB
```

### 2. Get Connection Details

```bash
# Retrieve connection string
choreo database get-connection my-ai-database
```

### 3. Connect from Your Application

**Python Example:**
```python
import psycopg2
from choreo.secrets import get_secret

# Get connection details from Choreo secrets
conn = psycopg2.connect(
    host=get_secret("DB_HOST"),
    database=get_secret("DB_NAME"),
    user=get_secret("DB_USER"),
    password=get_secret("DB_PASSWORD")
)

cursor = conn.cursor()
cursor.execute("SELECT version();")
print(cursor.fetchone())
```

**Node.js Example:**
```javascript
const { Pool } = require('pg');
const { getSecret } = require('@choreo/secrets');

const pool = new Pool({
  host: getSecret('DB_HOST'),
  database: getSecret('DB_NAME'),
  user: getSecret('DB_USER'),
  password: getSecret('DB_PASSWORD'),
  port: 5432,
});

pool.query('SELECT NOW()', (err, res) => {
  console.log(res.rows);
});
```

---

## Common Use Cases for AI Applications

### 1. Conversation History Storage

Store chat messages and context for AI assistants:

```sql
CREATE TABLE conversations (
    id SERIAL PRIMARY KEY,
    user_id VARCHAR(255) NOT NULL,
    session_id VARCHAR(255) NOT NULL,
    message_role VARCHAR(50) NOT NULL, -- 'user' or 'assistant'
    message_content TEXT NOT NULL,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    tokens_used INTEGER,
    INDEX idx_session (session_id),
    INDEX idx_user (user_id)
);
```

### 2. Vector Embeddings for Semantic Search

Store document embeddings for retrieval-augmented generation (RAG):

```sql
-- PostgreSQL with pgvector extension
CREATE EXTENSION vector;

CREATE TABLE document_embeddings (
    id SERIAL PRIMARY KEY,
    document_id VARCHAR(255) NOT NULL,
    chunk_text TEXT NOT NULL,
    embedding vector(1536), -- OpenAI embedding dimension
    metadata JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create index for similarity search
CREATE INDEX ON document_embeddings 
USING ivfflat (embedding vector_cosine_ops);
```

### 3. User Preferences and Context

```sql
CREATE TABLE user_profiles (
    user_id VARCHAR(255) PRIMARY KEY,
    preferences JSONB,
    conversation_style VARCHAR(100),
    preferred_model VARCHAR(100),
    token_quota INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 4. AI Model Usage Tracking

```sql
CREATE TABLE ai_usage_logs (
    id SERIAL PRIMARY KEY,
    user_id VARCHAR(255),
    model_name VARCHAR(100),
    prompt_tokens INTEGER,
    completion_tokens INTEGER,
    total_cost DECIMAL(10, 6),
    request_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    response_time_ms INTEGER,
    INDEX idx_user_time (user_id, request_timestamp)
);
```

---

## Database Selection Guide

| Database | Best For | AI Use Case |
|----------|----------|-------------|
| **PostgreSQL** | Complex queries, ACID compliance | Structured data, analytics, pgvector for embeddings |
| **MongoDB** | Flexible schema, JSON data | Document storage, conversation logs, metadata |
| **Redis** | Caching, real-time data | Session management, rate limiting, caching AI responses |
| **Pinecone** | Vector similarity search | RAG systems, semantic search, recommendation engines |

---

## Best Practices

### 1. **Connection Pooling**

Always use connection pooling to manage database connections efficiently:

```python
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool

engine = create_engine(
    connection_string,
    poolclass=QueuePool,
    pool_size=10,
    max_overflow=20
)
```

### 2. **Secure Credentials**

- Store database credentials in Choreo's secret management
- Never commit credentials to version control
- Use environment-specific configurations
- Rotate credentials regularly

### 3. **Optimize for AI Workloads**

```sql
-- Add indexes for frequently queried fields
CREATE INDEX idx_session_timestamp ON conversations(session_id, timestamp);

-- Partition large tables by date
CREATE TABLE conversations_2026_01 PARTITION OF conversations
    FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
```

### 4. **Implement Proper Error Handling**

```python
from contextlib import contextmanager
import psycopg2

@contextmanager
def get_db_connection():
    conn = None
    try:
        conn = psycopg2.connect(connection_string)
        yield conn
        conn.commit()
    except Exception as e:
        if conn:
            conn.rollback()
        raise e
    finally:
        if conn:
            conn.close()
```

### 5. **Monitor Database Performance**

- Set up alerts for slow queries
- Monitor connection pool utilization
- Track storage usage and plan for scaling
- Enable query logging for debugging

---

## Integration with AI Services

### RAG (Retrieval-Augmented Generation) Pattern

```python
def answer_with_context(user_question):
    # 1. Generate embedding for user question
    question_embedding = ai_service.embed(user_question)
    
    # 2. Find relevant documents from vector database
    cursor.execute("""
        SELECT chunk_text, 1 - (embedding <=> %s) as similarity
        FROM document_embeddings
        ORDER BY embedding <=> %s
        LIMIT 5
    """, (question_embedding, question_embedding))
    
    context_docs = cursor.fetchall()
    
    # 3. Build prompt with context
    context = "\n".join([doc[0] for doc in context_docs])
    prompt = f"Context:\n{context}\n\nQuestion: {user_question}\nAnswer:"
    
    # 4. Generate answer using AI service
    answer = ai_service.generate(prompt)
    
    # 5. Store conversation in database
    cursor.execute("""
        INSERT INTO conversations (user_id, message_role, message_content)
        VALUES (%s, 'user', %s), (%s, 'assistant', %s)
    """, (user_id, user_question, user_id, answer))
    
    return answer
```

---

## Backup and Recovery

### Automated Backups
```bash
# Configure automated backups
choreo database configure-backup \
  --name my-ai-database \
  --schedule "0 2 * * *" \
  --retention-days 30
```

### Manual Backup
```bash
# Create manual backup
choreo database backup my-ai-database --tag "pre-migration"
```

### Restore from Backup
```bash
# Restore to a specific point in time
choreo database restore my-ai-database \
  --timestamp "2026-01-15T10:00:00Z"
```

---

## Scaling Strategies

### Vertical Scaling
Increase database resources (CPU, memory, storage):
```bash
choreo database scale my-ai-database \
  --cpu 4 \
  --memory 16GB \
  --storage 100GB
```

### Horizontal Scaling
- **Read Replicas**: For read-heavy AI applications
- **Sharding**: Distribute data across multiple databases
- **Caching Layer**: Use Redis for frequently accessed data

---

## Migration Guide

### From External Database to Choreo-Managed

1. **Export data** from your existing database
2. **Create Choreo-managed database** with appropriate configuration
3. **Import data** using migration tools
4. **Update connection strings** in your application
5. **Test thoroughly** before switching production traffic
6. **Monitor performance** post-migration

---

## Troubleshooting

### Connection Issues
- Verify network connectivity and firewall rules
- Check if database is running and accessible
- Validate credentials and connection string
- Review connection pool settings

### Performance Issues
- Analyze slow query logs
- Check for missing indexes
- Monitor database resource utilization
- Consider adding read replicas

### Storage Issues
- Monitor storage usage trends
- Archive old data
- Implement data retention policies
- Plan for storage scaling

---

## Related Documentation

- [AI Services README](../README.md)
- [Choreo-Managed Databases](./Choreo-Managed_Databases.md)
- [Main Project Documentation](../../../README.md)
- [Deployment Tracks](../../../Deployment_Tracks.md)

---

## Support

For database-related issues:
1. Check Choreo platform status
2. Review database logs in Choreo console
3. Consult the troubleshooting guide above
4. Contact platform support if needed

---

**Last Updated**: January 2026  
**Maintained By**: Platform Engineering Team


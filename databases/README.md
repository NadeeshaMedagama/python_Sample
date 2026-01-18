# Choreo Databases

This directory contains comprehensive documentation for Choreo-managed database services, including relational databases, vector databases, and cache solutions.

---

## Overview

Choreo provides fully managed database services powered by **Aiven.io**, offering enterprise-grade data persistence and caching capabilities across all major cloud providers (AWS, Azure, GCP, and DigitalOcean). These managed services eliminate infrastructure complexity while providing production-ready features like automated backups, high availability, and scalability.

---

## Contents

- **[Choreo-Managed_Databases.md](./Choreo-Managed_Databases.md)** - Complete guide to all Choreo-managed database offerings

---

## Available Database Services

### 🐘 PostgreSQL
**Advanced Open-Source Relational Database**

PostgreSQL is a powerful object-relational database system ideal for:
- Complex queries and transactions
- Structured and unstructured data
- Vector similarity search with pgvector extension
- ACID compliance requirements
- Advanced indexing and full-text search

**Key Features:**
- Support for JSON, arrays, and custom data types
- Powerful query optimization
- MVCC (Multi-Version Concurrency Control)
- Extensible with custom functions and extensions

### 🐬 MySQL
**Popular Open-Source Relational Database**

MySQL is a user-friendly, flexible relational database perfect for:
- Web applications
- E-commerce platforms
- Content management systems
- High-read workloads
- Rapid prototyping and development

**Key Features:**
- Fast read operations
- Simple setup and management
- Wide ecosystem support
- Reliable replication

### ⚡ Choreo-Managed Cache (Valkey/Redis Compatible)
**High-Performance In-Memory Data Store**

Fully managed cache service compatible with Redis® OSS, powered by Valkey:
- Session management
- Real-time analytics
- Rate limiting
- Message queuing
- Caching layer for databases

**Key Features:**
- Sub-millisecond latency
- Rich data structures (strings, hashes, lists, sets, sorted sets)
- Pub/Sub messaging
- Persistence options
- Atomic operations

---

## Service Plans

Choreo offers flexible service plans for different use cases:

| Plan | Use Case | Features |
|------|----------|----------|
| **Hobbyist** | Development & Testing | Single node, 7-day free trial available |
| **Startup** | Small Production Apps | Enhanced resources, daily backups |
| **Business** | Production Workloads | High availability, automated failover |
| **Premium** | Enterprise Applications | Multi-region, advanced monitoring |

---

## Quick Start

### 1. Create a Database

**Via Choreo Console:**
```
Choreo Console → Project → Platform Services → Databases → Create Database
```

Select your database type, cloud provider, region, and service plan.

**Via Choreo CLI:**
```bash
# PostgreSQL
choreo database create \
  --name my-postgres-db \
  --type postgresql \
  --version 15 \
  --plan startup \
  --cloud aws \
  --region us-east-1

# MySQL
choreo database create \
  --name my-mysql-db \
  --type mysql \
  --version 8.0 \
  --plan startup \
  --cloud gcp \
  --region us-central1

# Cache
choreo cache create \
  --name my-cache \
  --plan hobbyist \
  --cloud azure \
  --region eastus
```

### 2. Get Connection Details

```bash
# Retrieve connection credentials
choreo database credentials my-postgres-db

# Output:
# Host: my-db-project.aivencloud.com
# Port: 12345
# Database: defaultdb
# Username: avnadmin
# Password: **********
```

### 3. Connect to Your Database

**PostgreSQL Example:**
```python
import psycopg2

conn = psycopg2.connect(
    host="your-host.aivencloud.com",
    port=12345,
    database="defaultdb",
    user="avnadmin",
    password="your-secure-password",
    sslmode="require"
)

cursor = conn.cursor()
cursor.execute("SELECT version();")
print(cursor.fetchone())
```

**MySQL Example:**
```python
import mysql.connector

conn = mysql.connector.connect(
    host="your-host.aivencloud.com",
    port=12345,
    database="defaultdb",
    user="avnadmin",
    password="your-secure-password",
    ssl_ca="ca.pem"
)

cursor = conn.cursor()
cursor.execute("SELECT VERSION();")
print(cursor.fetchone())
```

**Cache Example:**
```python
import redis

cache = redis.Redis(
    host="your-host.aivencloud.com",
    port=12345,
    password="your-secure-password",
    ssl=True,
    decode_responses=True
)

cache.set("key", "value")
print(cache.get("key"))
```

---

## Key Features

### 🔒 Enterprise Security
- **TLS/SSL Encryption**: All connections encrypted in transit
- **At-Rest Encryption**: Data encrypted on disk
- **Network Isolation**: VPC peering support
- **Access Control**: IP allowlisting and authentication
- **Compliance**: SOC 2, ISO 27001, GDPR compliant

### 📊 High Availability
- **99.99% Uptime SLA**: Guaranteed by Aiven
- **Automated Failover**: Seamless recovery from failures
- **Multi-Node Clusters**: For production workloads
- **Geographic Redundancy**: Multi-region support

### 🔄 Automated Management
- **Daily Backups**: Automatic backup retention
- **Point-in-Time Recovery**: Restore to any point in time
- **Automatic Updates**: Security patches and version updates
- **Monitoring**: Built-in performance metrics and alerts

### 📈 Scalability
- **Vertical Scaling**: Upgrade CPU, memory, and storage
- **Horizontal Scaling**: Add read replicas
- **Storage Auto-Scaling**: Automatic storage expansion
- **Connection Pooling**: Built-in PgBouncer for PostgreSQL

---

## Use Cases

### Web Applications
Store user data, session information, and application state with MySQL or PostgreSQL.

### AI/ML Applications
- Store vector embeddings in PostgreSQL with pgvector
- Cache AI model predictions in Redis
- Track model usage and performance metrics

### Real-Time Analytics
- Use Redis for real-time dashboards
- Stream processing with Redis Streams
- Time-series data analysis

### E-Commerce
- Product catalogs in PostgreSQL/MySQL
- Shopping cart sessions in Redis
- Order processing and inventory management

### Content Management
- Store articles and media metadata
- User-generated content
- Full-text search capabilities

---

## Best Practices

### 1. **Security**
```python
# Always use environment variables for credentials
import os
from choreo.secrets import get_secret

DB_HOST = get_secret("DB_HOST")
DB_PASSWORD = get_secret("DB_PASSWORD")

# Never hardcode credentials!
```

### 2. **Connection Management**
```python
# Use connection pooling
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool

engine = create_engine(
    f"postgresql://{user}:{password}@{host}:{port}/{database}",
    poolclass=QueuePool,
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True  # Verify connections before using
)
```

### 3. **Error Handling**
```python
import psycopg2
from psycopg2 import OperationalError

try:
    conn = psycopg2.connect(connection_string)
    # Database operations
except OperationalError as e:
    print(f"Connection failed: {e}")
    # Implement retry logic or fallback
```

### 4. **Performance Optimization**
```sql
-- Create indexes for frequently queried columns
CREATE INDEX idx_user_email ON users(email);

-- Analyze query performance
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'user@example.com';

-- Use prepared statements to prevent SQL injection
```

### 5. **Monitoring**
- Enable slow query logs
- Set up alerts for high CPU/memory usage
- Monitor connection pool utilization
- Track storage growth

---

## Pricing & Billing

### Free Trial
- **7-day free trial** for all database types on Hobbyist plan
- Available to free-tier Choreo users
- No credit card required

### Paid Plans
- Billing included in Choreo subscription
- Pricing varies by service plan and resources
- Pay only for what you use
- See [Choreo Platform Services Billing](https://docs.choreo.dev/references/choreo-platform-services-billing-and-upgrades/) for details

### Cost Optimization Tips
- Start with Hobbyist plan for development
- Scale up only when needed
- Use caching to reduce database load
- Archive old data regularly
- Monitor and optimize slow queries

---

## Technology Partnership

Choreo's managed data services are powered by **Aiven.io**, providing:
- Enterprise-grade infrastructure
- Automated database management
- Expert support and maintenance
- 99.99% monthly uptime SLA

For more information:
- [Aiven SLA](https://aiven.io/sla)
- [Aiven Security & Compliance](https://aiven.io/security-compliance)

---

## Support Model

**Direct Choreo Support:**
- Operational assistance via Choreo support channels
- Guidance on database selection and configuration
- Help with connection and integration issues

**Escalation to Aiven:**
- Complex technical issues
- Infrastructure-level problems
- Coordination with Aiven experts when needed

**How to Get Support:**
1. Check Choreo documentation and troubleshooting guides
2. Review database logs in Choreo console
3. Contact Choreo support team
4. Choreo will escalate to Aiven if necessary

---

## Migration Guide

### From External Database to Choreo-Managed

**Step 1: Export Your Data**
```bash
# PostgreSQL
pg_dump -h old-host -U username -d dbname > backup.sql

# MySQL
mysqldump -h old-host -u username -p dbname > backup.sql
```

**Step 2: Create Choreo Database**
```bash
choreo database create --name new-db --type postgresql --plan startup
```

**Step 3: Import Data**
```bash
# PostgreSQL
psql -h new-host -U avnadmin -d defaultdb < backup.sql

# MySQL
mysql -h new-host -u avnadmin -p defaultdb < backup.sql
```

**Step 4: Update Application**
- Update connection strings
- Test thoroughly in staging environment
- Switch traffic gradually

---

## Troubleshooting

### Connection Issues
**Problem:** Cannot connect to database
- ✅ Verify credentials are correct
- ✅ Check IP allowlist settings
- ✅ Ensure SSL/TLS is enabled
- ✅ Verify database is running (check Choreo console)

### Performance Issues
**Problem:** Slow query performance
- ✅ Check for missing indexes
- ✅ Analyze slow query logs
- ✅ Monitor resource utilization (CPU, memory)
- ✅ Consider upgrading service plan

### Storage Issues
**Problem:** Running out of storage
- ✅ Archive old data
- ✅ Implement data retention policies
- ✅ Upgrade storage capacity
- ✅ Enable automatic storage expansion

### High Availability Issues
**Problem:** Service interruptions
- ✅ Upgrade to multi-node plan
- ✅ Check Aiven service status
- ✅ Review automatic failover settings
- ✅ Consider multi-region deployment

---

## Related Documentation

- [Main Project README](../README.md)
- [DB and Services Overview](../db-and-services/README.md)
- [AI Services Integration](../db-and-services/AI%20Services/README.md)
- [Choreo CLI Documentation](../Choreo_CLI.md)
- [Deployment Tracks](../Deployment_Tracks.md)

---

## Additional Resources

### PostgreSQL
- [PostgreSQL Official Documentation](https://www.postgresql.org/docs/)
- [pgvector for Vector Search](https://github.com/pgvector/pgvector)

### MySQL
- [MySQL Official Documentation](https://dev.mysql.com/doc/)
- [MySQL Performance Tuning](https://dev.mysql.com/doc/refman/8.0/en/optimization.html)

### Redis/Valkey
- [Redis Command Reference](https://redis.io/commands/)
- [Valkey Documentation](https://valkey.io/)

---

## Compliance & Legal

**Trademarks:**
PostgreSQL, MySQL, and Redis® are trademarks and property of their respective owners. All product and service names used in this documentation are for identification purposes only.

**Data Privacy:**
- All managed services are GDPR compliant
- Data residency options available
- Encryption at rest and in transit
- Regular security audits

---

**Last Updated**: January 2026  
**Maintained By**: Platform Engineering Team  
**Powered By**: Aiven.io


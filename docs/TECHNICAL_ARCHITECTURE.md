# EcoCaptain V0 - Technical Architecture

## System Overview

EcoCaptain V0 is built on a modern, scalable microservices architecture designed to support both community-driven and institutional environmental impact tracking. The system emphasizes modularity, security, and performance.

## Architecture Principles

1. **Separation of Concerns**: Clear boundaries between presentation, business logic, and data layers
2. **Scalability**: Horizontal scaling capability for all services
3. **Security First**: Zero-trust architecture with defense in depth
4. **API-First Design**: All functionality exposed through well-documented APIs
5. **Event-Driven**: Asynchronous processing for long-running tasks
6. **Cloud-Native**: Containerized, orchestrated, and cloud-agnostic

## Technology Stack

### Frontend
- **Framework**: React 18 with TypeScript
- **State Management**: Redux Toolkit + RTK Query
- **UI Components**: Material-UI v5
- **Data Visualization**: D3.js for Evidence Graph
- **Forms**: React Hook Form + Yup validation
- **Testing**: Jest, React Testing Library, Cypress

### Backend
- **Runtime**: Node.js 18 LTS
- **API Layer**: 
  - GraphQL: Apollo Server 4
  - REST: Express.js with OpenAPI
- **Authentication**: JWT with refresh tokens
- **Authorization**: Casbin for RBAC/ABAC
- **Job Queue**: Bull with Redis
- **Testing**: Jest, Supertest

### Database
- **Primary**: PostgreSQL 14+ with PostGIS
- **Cache**: Redis 7+
- **Search**: Elasticsearch (future)
- **Time Series**: TimescaleDB (future)

### Infrastructure
- **Containers**: Docker
- **Orchestration**: Kubernetes
- **CI/CD**: GitHub Actions
- **Monitoring**: Prometheus + Grafana
- **Logging**: ELK Stack
- **Cloud**: AWS (primary), Azure (secondary)

## Database Schema

### Core Tables

```sql
-- Organizations
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    type VARCHAR(50) NOT NULL, -- 'community', 'enterprise', 'government'
    tier VARCHAR(50) NOT NULL, -- 'free', 'pro', 'enterprise'
    settings JSONB,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Users
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL,
    organization_id UUID REFERENCES organizations(id),
    settings JSONB,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Projects
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    location GEOGRAPHY(POINT, 4326),
    metadata JSONB,
    status VARCHAR(50),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Impact Claims
CREATE TABLE impact_claims (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id),
    claim_type VARCHAR(100) NOT NULL,
    indicator VARCHAR(255) NOT NULL,
    value DECIMAL(20, 4),
    unit VARCHAR(50),
    method VARCHAR(50), -- 'SROI', 'CBA', 'PIPA'
    calculation_data JSONB,
    assurance_level VARCHAR(50), -- 'self', 'peer', 'verified'
    sdg_targets TEXT[],
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Evidence
CREATE TABLE evidence (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    claim_id UUID REFERENCES impact_claims(id),
    type VARCHAR(50), -- 'document', 'data', 'image', 'video'
    source VARCHAR(255),
    url TEXT,
    metadata JSONB,
    hash VARCHAR(64), -- SHA-256 for integrity
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Assurance Reviews
CREATE TABLE assurance_reviews (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    claim_id UUID REFERENCES impact_claims(id),
    reviewer_id UUID REFERENCES users(id),
    review_type VARCHAR(50), -- 'peer', 'expert', 'third_party'
    status VARCHAR(50), -- 'pending', 'approved', 'rejected', 'conditional'
    comments TEXT,
    score INTEGER,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Audit Trail
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    action VARCHAR(100) NOT NULL,
    entity_type VARCHAR(50),
    entity_id UUID,
    changes JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

## API Design

### GraphQL Schema

```graphql
type Query {
  # User queries
  me: User
  user(id: ID!): User
  users(filter: UserFilter, pagination: Pagination): UserConnection
  
  # Organization queries
  organization(id: ID!): Organization
  organizations(filter: OrgFilter, pagination: Pagination): OrgConnection
  
  # Project queries
  project(id: ID!): Project
  projects(filter: ProjectFilter, pagination: Pagination): ProjectConnection
  
  # Impact queries
  impactClaim(id: ID!): ImpactClaim
  impactClaims(filter: ClaimFilter, pagination: Pagination): ClaimConnection
  
  # Evidence Graph
  evidenceGraph(projectId: ID!): EvidenceGraph
}

type Mutation {
  # User mutations
  updateProfile(input: UpdateProfileInput!): User
  
  # Organization mutations
  createOrganization(input: CreateOrgInput!): Organization
  updateOrganization(id: ID!, input: UpdateOrgInput!): Organization
  
  # Project mutations
  createProject(input: CreateProjectInput!): Project
  updateProject(id: ID!, input: UpdateProjectInput!): Project
  
  # Impact mutations
  createImpactClaim(input: CreateClaimInput!): ImpactClaim
  updateImpactClaim(id: ID!, input: UpdateClaimInput!): ImpactClaim
  submitForReview(claimId: ID!): ImpactClaim
  
  # Evidence mutations
  attachEvidence(claimId: ID!, input: EvidenceInput!): Evidence
  removeEvidence(id: ID!): Boolean
}

type Subscription {
  projectUpdated(projectId: ID!): Project
  claimStatusChanged(projectId: ID!): ImpactClaim
  reviewCompleted(claimId: ID!): AssuranceReview
}
```

### REST Endpoints

```yaml
openapi: 3.0.0
paths:
  /api/v1/auth/login:
    post:
      summary: User login
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                email: { type: string }
                password: { type: string }
      responses:
        200:
          description: Login successful
          content:
            application/json:
              schema:
                type: object
                properties:
                  accessToken: { type: string }
                  refreshToken: { type: string }
                  
  /api/v1/reports/compliance:
    post:
      summary: Generate compliance report
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                projectId: { type: string }
                framework: { type: string, enum: [AASB, ISSB, TNFD] }
                period: { type: object }
      responses:
        200:
          description: Report generated
          content:
            application/json:
              schema:
                type: object
                properties:
                  reportId: { type: string }
                  status: { type: string }
                  downloadUrl: { type: string }
```

## Security Architecture

### Authorization Model (RBAC + ABAC)

```yaml
roles:
  - admin:
      permissions:
        - organizations:*
        - projects:*
        - users:*
        - reports:*
        
  - manager:
      permissions:
        - organizations:read
        - projects:*
        - users:read,update
        - reports:*
        
  - contributor:
      permissions:
        - organizations:read
        - projects:read,update
        - users:read:self
        - reports:read
        
  - viewer:
      permissions:
        - organizations:read
        - projects:read
        - users:read:self
        - reports:read

attributes:
  - organization_id
  - project_owner
  - assurance_level
  - data_classification
```

## Performance Optimization

### Caching Strategy

1. **Application Cache** (Redis)
   - Session data (15 min TTL)
   - User permissions (5 min TTL)
   - Frequently accessed projects (10 min TTL)
   - Report templates (1 hour TTL)

2. **Database Cache** (PostgreSQL)
   - Materialized views for aggregations
   - Partial indexes for common queries
   - Table partitioning for audit logs

3. **CDN Cache** (CloudFront)
   - Static assets (1 year TTL)
   - Public reports (1 hour TTL)
   - API responses (varies by endpoint)

### Query Optimization

```sql
-- Example: Optimized impact claims query
CREATE INDEX idx_claims_project_assurance 
ON impact_claims(project_id, assurance_level) 
WHERE deleted_at IS NULL;

CREATE MATERIALIZED VIEW mv_project_summary AS
SELECT 
    p.id,
    p.name,
    COUNT(DISTINCT ic.id) as claim_count,
    AVG(ic.value) as avg_impact,
    MAX(ic.assurance_level) as max_assurance
FROM projects p
LEFT JOIN impact_claims ic ON p.id = ic.project_id
GROUP BY p.id, p.name;

CREATE INDEX ON mv_project_summary(id);
```

## Deployment Architecture

### Kubernetes Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecocaptain-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ecocaptain-api
  template:
    metadata:
      labels:
        app: ecocaptain-api
    spec:
      containers:
      - name: api
        image: ecocaptain/api:latest
        ports:
        - containerPort: 4000
        env:
        - name: NODE_ENV
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 4000
          initialDelaySeconds: 30
          periodSeconds: 10
```

### CI/CD Pipeline

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run tests
        run: |
          npm ci
          npm run test:ci
          npm run test:e2e

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build and push Docker image
        run: |
          docker build -t ecocaptain/api:${{ github.sha }} .
          docker push ecocaptain/api:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/ecocaptain-api \
            api=ecocaptain/api:${{ github.sha }}
          kubectl rollout status deployment/ecocaptain-api
```

## Monitoring & Observability

### Key Metrics

1. **Application Metrics**
   - Request rate and latency (p50, p95, p99)
   - Error rate by endpoint
   - Active user sessions
   - Database connection pool utilization

2. **Business Metrics**
   - Claims created per day
   - Average assurance progression time
   - Report generation success rate
   - User engagement metrics

3. **Infrastructure Metrics**
   - CPU and memory utilization
   - Disk I/O and network throughput
   - Container restart frequency
   - Database query performance

### Logging Structure

```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "info",
  "service": "impact-service",
  "trace_id": "abc123",
  "user_id": "user456",
  "organization_id": "org789",
  "action": "create_impact_claim",
  "duration_ms": 145,
  "status": "success",
  "metadata": {
    "claim_type": "carbon_reduction",
    "value": 1500,
    "unit": "tCO2e"
  }
}
```

## Disaster Recovery

### Backup Strategy

1. **Database Backups**
   - Continuous replication to standby
   - Daily automated backups (30-day retention)
   - Weekly full backups (1-year retention)
   - Point-in-time recovery capability

2. **File Storage Backups**
   - S3 cross-region replication
   - Versioning enabled
   - Lifecycle policies for archival

3. **Configuration Backups**
   - Infrastructure as Code in Git
   - Secrets in AWS Secrets Manager
   - Regular export of Kubernetes configs

### Recovery Procedures

- **RTO (Recovery Time Objective)**: 4 hours
- **RPO (Recovery Point Objective)**: 1 hour
- **Failover Process**: Automated with manual verification
- **Testing Schedule**: Quarterly DR drills

## Future Considerations

### Scalability Enhancements
- Implement read replicas for database
- Add message queuing (RabbitMQ/Kafka)
- Introduce caching layers (Varnish)
- Consider microservices decomposition

### Feature Additions
- Blockchain integration for immutability
- Machine learning for impact predictions
- Real-time collaboration features
- Mobile native applications

### Technical Debt Management
- Regular dependency updates
- Code quality metrics tracking
- Performance regression testing
- Security vulnerability scanning
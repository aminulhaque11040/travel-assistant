# FastAPI LangGraph Agent Production Ready Template

A production-ready template for building AI agents using FastAPI and LangGraph with comprehensive monitoring, database integration, and containerized deployment.

## Features

- **FastAPI Framework**: High-performance async API framework
- **LangGraph Integration**: Advanced AI agent workflows
- **PostgreSQL Database**: Integrated database service with Docker
- **Authentication & Authorization**: JWT-based security
- **Rate Limiting**: Built-in API rate limiting
- **Monitoring Stack**: Prometheus + Grafana + cAdvisor
- **Production Ready**: Docker containerization with health checks
- **Environment Management**: Separate configs for dev/staging/production
- **Logging**: Structured logging with rotation
- **Security**: Non-root user, secure secrets management

## Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   FastAPI App   │────│   PostgreSQL    │    │   Monitoring    │
│   (Port 8000)   │    │   (Port 5432)   │    │   Stack         │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │  Docker Network │
                    │   (monitoring)  │
                    └─────────────────┘
```

## Prerequisites

- Docker Desktop 4.0+
- Docker Compose v2.0+
- Git

## Quick Start

### 1. Clone the Repository

```bash
git clone <repository-url>
cd fastapi-langgraph-agent-production-ready-template
```

### 2. Environment Configuration

Copy and configure environment files:

```bash
# Copy environment template
cp .env.example .env.development
cp .env.example .env.production

# Edit development environment
nano .env.development
```

**Required Environment Variables:**

```bash
# LLM Settings (Required)
LLM_API_KEY="your-openai-api-key-here"  # Get from https://platform.openai.com/api-keys

# Langfuse Settings (Optional - for observability)
LANGFUSE_PUBLIC_KEY="your-langfuse-public-key"
LANGFUSE_SECRET_KEY="your-langfuse-secret-key"

# JWT Settings (Auto-generated for development)
JWT_SECRET_KEY="dummy-secret-key-for-development-123456"
```

### 3. Start Services

```bash
# Start all services (app + database + monitoring)
docker-compose up --build -d

# Or start in foreground to see logs
docker-compose up --build
```

### 4. Verify Installation

```bash
# Check all services status
docker-compose ps

# Test API health
curl http://localhost:8000/health

# View application logs
docker-compose logs app -f
```

## Service Management

### Start Services

```bash
# Start all services in background
docker-compose up -d

# Start with rebuild
docker-compose up --build -d

# Start specific service
docker-compose up postgres -d
```

### Stop Services

```bash
# Stop all services
docker-compose down

# Stop and remove volumes (WARNING: This deletes database data)
docker-compose down -v

# Stop specific service
docker-compose stop app
```

### View Logs

```bash
# View all logs
docker-compose logs -f

# View specific service logs
docker-compose logs app -f
docker-compose logs postgres -f

# View last 100 lines
docker-compose logs --tail=100 app
```

### Restart Services

```bash
# Restart all services
docker-compose restart

# Restart specific service
docker-compose restart app
```

## Database Management

### PostgreSQL Integration

The template includes a fully integrated PostgreSQL service:

- **Service Name**: `postgres`
- **Database**: `agentdb`
- **User**: `agent`
- **Password**: `1234` (development only)
- **Port**: `5432`

### Database Operations

```bash
# Connect to database from host machine
psql -h localhost -p 5432 -U agent -d agentdb

# Connect to database from inside container
docker-compose exec postgres psql -U agent -d agentdb

# Run database migrations (if using Alembic)
docker-compose exec app alembic upgrade head

# Create database backup
docker-compose exec postgres pg_dump -U agent agentdb > backup.sql

# Restore database backup
docker-compose exec -T postgres psql -U agent agentdb < backup.sql
```

### Database Connection String

```bash
# From within Docker network (containers)
POSTGRES_URL=postgresql://agent:1234@postgres:5432/agentdb

# From host machine
POSTGRES_URL=postgresql://agent:1234@localhost:5432/agentdb
```

## Development

### Local Development Setup

```bash
# Install uv (if not already installed)
pip install uv

# Create virtual environment and install dependencies
uv venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
uv sync  # Install all dependencies from pyproject.toml and uv.lock

# Start only database service
docker-compose up postgres -d

# Run app locally with auto-reload
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

### Environment-Specific Deployment

```bash
# Development
APP_ENV=development docker-compose up --build -d

# Staging
APP_ENV=staging docker-compose up --build -d

# Production
APP_ENV=production docker-compose up --build -d
```

## Monitoring & Observability

### Access Monitoring Services

- **API Documentation**: http://localhost:8000/docs
- **Health Check**: http://localhost:8000/health
- **Grafana Dashboard**: http://localhost:3000 (admin/admin)
- **Prometheus**: http://localhost:9090
- **cAdvisor**: http://localhost:8080

### Key Metrics

- API Response Times
- Request Rates
- Error Rates
- Database Connections
- System Resources (CPU, Memory, Disk)
- Container Health Status

## API Endpoints

### Core Endpoints

```bash
# Health Check
GET /health

# API Documentation
GET /docs
GET /redoc

# Authentication
POST /api/v1/auth/login
POST /api/v1/auth/refresh

# Agent Endpoints
POST /api/v1/chat
POST /api/v1/chat/stream
GET /api/v1/messages
```

### Testing Endpoints

```bash
# Test chat endpoint
curl -X POST "http://localhost:8000/api/v1/chat" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{"message": "Hello, agent!"}'

# Test health endpoint
curl http://localhost:8000/health
```

## Security

### Production Security Checklist

- [ ] Change default PostgreSQL password
- [ ] Use strong JWT secret keys
- [ ] Set proper CORS origins
- [ ] Enable HTTPS in production
- [ ] Use secrets management (Docker secrets, K8s secrets)
- [ ] Regular security updates
- [ ] Monitor security logs

### Environment Variables Security

```bash
# Development - use dummy values
JWT_SECRET_KEY="dummy-secret-key-for-development-123456"

# Production - use strong, random secrets
JWT_SECRET_KEY="$(openssl rand -base64 32)"
```

## Troubleshooting

### Common Issues

**1. Database Connection Failed**

```bash
# Check if PostgreSQL is running
docker-compose ps postgres

# View PostgreSQL logs
docker-compose logs postgres -f

# Test connection
docker-compose exec app ping postgres
```

**2. App Service Unhealthy**

```bash
# Check app logs
docker-compose logs app -f

# Verify health endpoint
curl http://localhost:8000/health

# Check if all environment variables are set
docker-compose exec app env | grep -E "(LLM_API_KEY|JWT_SECRET_KEY)"
```

**3. Port Already in Use**

```bash
# Check what's using the port
lsof -i :8000

# Stop conflicting services
docker stop $(docker ps -q)
```

**4. Environment File Not Found**

```bash
# Verify .env files exist
ls -la .env*

# Check Docker Compose configuration
docker-compose config
```

### Reset Everything

```bash
# Nuclear option - reset everything
docker-compose down -v --remove-orphans
docker system prune -a
docker-compose up --build -d
```

## Production Deployment

### Docker Compose Production

```bash
# Set production environment
export APP_ENV=production

# Deploy with production config
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up --build -d
```

### Kubernetes Deployment

```bash
# Apply Kubernetes manifests
kubectl apply -f k8s/

# Check deployment status
kubectl get pods -l app=fastapi-agent
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

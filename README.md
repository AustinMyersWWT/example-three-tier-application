# example-three-tier-application

A reference implementation of a three-tier web application demonstrating modern full-stack development practices. Built with Next.js, Express, and PostgreSQL, it runs locally with Docker Compose and deploys to Google Cloud Platform using Terraform.

## 🏗️ Architecture

```
Browser → Web (Next.js :3000) → API (Express :3001) → PostgreSQL :5432
```

### Technology Stack

| Layer | Technology | Purpose | Location |
|-------|-----------|---------|----------|
| **Frontend** | Next.js 16.2, React 19, TypeScript | Server-side rendered UI with Server Actions | `src/web/` |
| **API** | Express 5, Node.js 22 | RESTful JSON API | `src/api/` |
| **Database** | PostgreSQL 17 | Data persistence | Docker / Cloud SQL |
| **Migrations** | node-pg-migrate 8 | Schema version control | `src/db/` |
| **Infrastructure** | Terraform 1.5+ | IaC for GCP deployment | `src/infrastructure/` |
| **Styling** | Tailwind CSS 4 | Utility-first styling with dark mode | `src/web/app/` |

### Application Features

The app is a **task manager** (to-do list) that demonstrates:
- ✅ Creating, updating, and listing tasks
- ✅ Toggle task completion status
- ✅ Server-side rendering with Next.js App Router
- ✅ Server Actions for mutations with automatic revalidation
- ✅ Dark mode support
- ✅ Responsive design with Tailwind CSS
- ✅ Database migrations with rollback support
- ✅ Production-ready Docker multi-stage builds
- ✅ Infrastructure as Code with Terraform

## 🚀 Quick Start

### Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (or Docker Engine + Compose plugin)
- No Node.js installation needed—everything runs in containers

### Running Locally

```bash
# Clone the repository
git clone https://github.com/yourusername/example-three-tier-application.git
cd example-three-tier-application

# Start all services
docker compose up --build
```

This orchestrates four services:

1. **postgres** — PostgreSQL 17 database with health checks
2. **migrate** — Applies database schema migrations, then exits
3. **api** — Express REST API (internal only, port 3001)
4. **web** — Next.js frontend (exposed on [http://localhost:3000](http://localhost:3000))

The compose file uses dependency ordering and health checks to ensure services start in the correct sequence.

### Development Workflow

```bash
# View logs
docker compose logs -f

# Stop containers (preserves data)
docker compose down

# Stop and remove all data
docker compose down -v

# Rebuild after code changes
docker compose up --build

# Run a specific service
docker compose up web
```

## 📁 Project Structure

```
.
├── src/
│   ├── api/                     # Express REST API
│   │   ├── index.js             # Route handlers and server setup
│   │   ├── db.js                # PostgreSQL connection pool
│   │   ├── package.json         # API dependencies
│   │   └── Dockerfile           # Production API image
│   │
│   ├── web/                     # Next.js frontend
│   │   ├── app/
│   │   │   ├── page.tsx         # Main task list UI (Server Component)
│   │   │   ├── actions.ts       # Server Actions for data mutations
│   │   │   ├── layout.tsx       # Root layout with fonts
│   │   │   └── globals.css      # Tailwind styles
│   │   ├── public/              # Static assets
│   │   ├── package.json         # Frontend dependencies
│   │   └── Dockerfile           # Multi-stage production build
│   │
│   ├── db/                      # Database migrations
│   │   ├── migrations/
│   │   │   ├── 1718500000000_initial-schema.js   # Creates users table
│   │   │   └── 1718500001000_create-tasks.js     # Creates tasks table
│   │   ├── package.json         # Migration tool dependencies
│   │   └── Dockerfile           # Migration runner image
│   │
│   └── infrastructure/          # Terraform for GCP
│       ├── main.tf              # VPC, Cloud SQL, Cloud Run services
│       ├── migration.tf         # Cloud Run Job for migrations
│       ├── variables.tf         # Input variables
│       ├── outputs.tf           # Stack outputs (URLs, IPs)
│       └── terraform.tfvars.example  # Example configuration
│
├── .github/
│   └── workflows/
│       └── deploy.yml           # CI/CD pipeline for GCP
│
├── docker-compose.yml           # Local development stack
├── agents.md                    # Guide for AI-assisted development
└── README.md                    # This file
```

## 🔌 API Reference

The REST API provides endpoints for task management:

| Method | Path | Description | Request Body | Response |
|--------|------|-------------|--------------|----------|
| `GET` | `/health` | Health check | — | `{ "status": "ok" }` |
| `GET` | `/tasks` | List all tasks | — | `Task[]` |
| `POST` | `/tasks` | Create a task | `{ "title": string }` | `Task` |
| `PATCH` | `/tasks/:id` | Update a task | `{ "completed"?: boolean, "title"?: string }` | `Task` |

### Task Schema

```typescript
{
  id: number;
  title: string;
  completed: boolean;
  created_at: string; // ISO 8601
}
```

### Example API Usage

```bash
# List all tasks
curl http://localhost:3001/tasks

# Create a task
curl -X POST http://localhost:3001/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Learn Docker"}'

# Toggle completion
curl -X PATCH http://localhost:3001/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{"completed": true}'
```

## 🗄️ Database

### Schema

```sql
-- Users table (created but not yet used by the app)
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) NOT NULL UNIQUE,
  created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Tasks table
CREATE TABLE tasks (
  id SERIAL PRIMARY KEY,
  title VARCHAR(500) NOT NULL,
  completed BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### Managing Migrations

```bash
# Run migrations (already happens automatically in docker compose)
docker compose run --rm migrate

# Create a new migration
cd src/db
npm run migrate create add-due-date

# Rollback last migration
npm run migrate:down
```

### Connection Details

In local development:
- **Host:** `postgres` (service name in Docker network)
- **Port:** `5432`
- **Database:** `app`
- **User:** `app`
- **Password:** `app`
- **Connection String:** `postgres://app:app@postgres:5432/app`

## ☁️ Deploying to Google Cloud Platform

The infrastructure code deploys a production-ready stack on GCP with:

- **Cloud Run** for containerized web and API services
- **Cloud SQL** (PostgreSQL) with private IP
- **VPC** with private service access
- **Secret Manager** for database credentials
- **Cloud Run Job** for running migrations
- **Service Account** with least-privilege IAM roles

### Prerequisites

1. [Google Cloud SDK](https://cloud.google.com/sdk/docs/install)
2. [Terraform](https://www.terraform.io/downloads)
3. A GCP project with billing enabled
4. Required APIs enabled:
   ```bash
   gcloud services enable \
     run.googleapis.com \
     sql-component.googleapis.com \
     vpcaccess.googleapis.com \
     secretmanager.googleapis.com \
     servicenetworking.googleapis.com
   ```

### Deployment Steps

```bash
cd src/infrastructure

# Initialize Terraform with GCS backend
terraform init \
  -backend-config="bucket=YOUR_TERRAFORM_STATE_BUCKET" \
  -backend-config="prefix=todo/dev"

# Review planned changes
terraform plan \
  -var="project_id=YOUR_PROJECT_ID" \
  -var="api_image=gcr.io/YOUR_PROJECT_ID/api:latest" \
  -var="web_image=gcr.io/YOUR_PROJECT_ID/web:latest" \
  -var="db_image=gcr.io/YOUR_PROJECT_ID/db:latest"

# Apply infrastructure
terraform apply

# Run database migrations
gcloud run jobs execute todo-dev-migrate --region us-central1

# Get the web URL
terraform output web_url
```

### Infrastructure Components

| Resource | Type | Purpose |
|----------|------|---------|
| VPC Network | `google_compute_network` | Private network for services |
| Cloud SQL | `google_sql_database_instance` | Managed PostgreSQL 17 |
| Cloud Run (Web) | `google_cloud_run_v2_service` | Frontend service |
| Cloud Run (API) | `google_cloud_run_v2_service` | Backend API |
| Cloud Run Job | `google_cloud_run_v2_job` | Migration runner |
| Secret Manager | `google_secret_manager_secret` | Database credentials |
| VPC Connector | `google_vpc_access_connector` | Cloud Run → Cloud SQL |
| Service Account | `google_service_account` | Cloud Run identity |

### Environment Variables

#### API Service
- `DATABASE_URL` — PostgreSQL connection string (from Secret Manager)
- `PORT` — HTTP port (default: 3001)

#### Web Service
- `API_URL` — Internal API URL (e.g., `http://api:3001`)
- `PORT` — HTTP port (default: 3000)

#### Migration Job
- `DATABASE_URL` — PostgreSQL connection string (from Secret Manager)

## 🔄 CI/CD Pipeline

The `.github/workflows/deploy.yml` workflow automates:

1. **Build** — Multi-stage Docker builds for api, web, and db
2. **Push** — Upload images to Google Container Registry
3. **Migrate** — Run database migrations via Cloud Run Job
4. **Deploy** — Update Cloud Run services with Terraform

### Trigger Deployment

```bash
# Automatic on push to main
git push origin main

# Manual deployment to specific environment
gh workflow run deploy.yml -f environment=staging
```

## 🧪 Testing

```bash
# Health check
curl http://localhost:3001/health

# Test task creation
curl -X POST http://localhost:3001/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Test task"}'

# Verify in database
docker compose exec postgres psql -U app -d app -c "SELECT * FROM tasks;"
```

## 🛠️ Development Tips

### Accessing Services

```bash
# View web logs
docker compose logs -f web

# View API logs
docker compose logs -f api

# Access PostgreSQL shell
docker compose exec postgres psql -U app -d app

# Run commands in a service
docker compose exec api sh
```

### Hot Reloading

The local docker-compose setup doesn't include volume mounts for hot reloading. For active development:

```bash
# Develop frontend locally
cd src/web
npm install
API_URL=http://localhost:3001 npm run dev

# Develop API locally
cd src/api
npm install
DATABASE_URL=postgres://app:app@localhost:5432/app npm run dev
```

### Debugging

```bash
# Check service health
docker compose ps

# Inspect a specific service
docker compose logs migrate

# Restart a single service
docker compose restart api

# Execute SQL queries
docker compose exec postgres psql -U app -d app -c "SELECT * FROM tasks;"
```

## 🤖 AI-Assisted Development

This repository includes an `agents.md` guide with:
- Best practices for using Claude Code or GitHub Copilot
- Common prompts for feature development
- Patterns for database migrations
- Infrastructure modifications
- Code review and debugging assistance

See [agents.md](./agents.md) for details.

## 📝 License

MIT License - see [LICENSE](./LICENSE) for details.

## 🤝 Contributing

Contributions are welcome! This is a reference implementation, so focus on:
- Improving documentation
- Fixing bugs
- Adding common patterns (authentication, testing, etc.)
- Enhancing the deployment pipeline

## 🔗 Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [Express.js Guide](https://expressjs.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Docker Compose Reference](https://docs.docker.com/compose/)
- [Terraform GCP Provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs)
- [Cloud Run Documentation](https://cloud.google.com/run/docs)

---

**Built with ❤️ as a learning resource for modern full-stack development**

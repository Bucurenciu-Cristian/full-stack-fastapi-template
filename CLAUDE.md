# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a full-stack FastAPI template with a React frontend. It's designed as a production-ready template for building web applications with authentication, CRUD operations, and deployment configurations.

**Technology Stack:**
- **Backend**: FastAPI with SQLModel, PostgreSQL, Alembic migrations, JWT authentication
- **Frontend**: React with TypeScript, Vite, TanStack Query/Router, Chakra UI
- **Database**: PostgreSQL with SQLModel ORM
- **Deployment**: Docker Compose with Traefik proxy, GitHub Actions CI/CD

## Development Environment Setup

### Quick Start with Docker Compose
```bash
# Start development environment with file watching
docker compose watch

# Access services:
# Frontend: http://localhost:5173
# Backend API: http://localhost:8000
# API Docs: http://localhost:8000/docs
# Adminer: http://localhost:8080
# Traefik UI: http://localhost:8090
```

### Local Development (Individual Services)

**Backend:**
```bash
cd backend
uv sync                          # Install dependencies
source .venv/bin/activate       # Activate virtual environment
fastapi dev app/main.py         # Start dev server
```

**Frontend:**
```bash
cd frontend
npm install                     # Install dependencies
npm run dev                     # Start dev server
```

## Core Development Commands

### Backend (Python/FastAPI)
```bash
# Package management
uv sync                         # Install dependencies
uv add <package>               # Add new dependency

# Development
fastapi dev app/main.py        # Start development server
fastapi run app/main.py        # Start production server

# Database migrations
alembic revision --autogenerate -m "Description"  # Create migration
alembic upgrade head                               # Apply migrations

# Testing and code quality
bash ./scripts/test.sh         # Run tests with coverage
coverage run -m pytest         # Run tests
coverage report --show-missing # Show coverage report
coverage html                  # Generate HTML coverage report
uv run pre-commit run --all-files  # Run pre-commit hooks manually
uv run ruff check              # Lint code
uv run ruff format             # Format code
uv run mypy .                  # Type checking
```

### Frontend (React/TypeScript)
```bash
# Package management
npm install                     # Install dependencies
npm install <package>          # Add new dependency

# Development
npm run dev                     # Start development server
npm run build                   # Build for production
npm run preview                 # Preview production build

# Code quality
npm run lint                    # Lint and format with Biome
npm run generate-client         # Generate API client from OpenAPI

# End-to-end testing
npx playwright test             # Run E2E tests
npx playwright test --ui        # Run tests with UI
```

### Docker Development
```bash
# Main development workflow
docker compose watch            # Start with file watching
docker compose up -d           # Start in detached mode
docker compose down            # Stop services
docker compose down -v         # Stop and remove volumes

# Individual service control
docker compose stop frontend   # Stop specific service
docker compose logs backend    # View service logs
docker compose exec backend bash  # Enter container shell

# Testing in Docker
docker compose exec backend bash scripts/tests-start.sh  # Run backend tests
```

## Project Architecture

### Backend Architecture (FastAPI)
The backend follows a clean architecture pattern:

- **`app/models.py`**: SQLModel models for database tables and API schemas
- **`app/api/routes/`**: API route handlers organized by domain (users, items, auth)
- **`app/core/`**: Core functionality (config, database, security)
- **`app/crud.py`**: Database operations (Create, Read, Update, Delete)
- **`app/api/deps.py`**: Dependency injection for authentication, database sessions

**Key patterns:**
- Uses SQLModel for both database models and API schemas
- JWT authentication with automatic token refresh
- Pydantic validation for all API inputs/outputs
- Alembic for database migrations
- Comprehensive test coverage with pytest

### Frontend Architecture (React)
The frontend uses modern React patterns:

- **`src/routes/`**: File-based routing with TanStack Router
- **`src/components/`**: Reusable UI components organized by feature
- **`src/client/`**: Auto-generated API client from OpenAPI spec
- **`src/hooks/`**: Custom React hooks for authentication and utilities

**Key patterns:**
- TypeScript throughout with strict type checking
- TanStack Query for server state management
- Chakra UI for component library
- Auto-generated API client ensures type safety
- Error boundaries for graceful error handling

### Database Architecture
- **PostgreSQL** with SQLModel ORM
- **UUID primary keys** for all entities
- **Cascade deletes** for related data
- **Alembic migrations** for schema changes
- **Connection pooling** in production

## Authentication & Security

The application uses JWT-based authentication:

- **Registration/Login**: Email/password authentication
- **Token management**: Access tokens with automatic refresh
- **Password reset**: Email-based password recovery
- **Authorization**: Role-based access with superuser privileges
- **Security headers**: CORS, HTTPS redirects in production

## Testing Strategy

### Backend Testing
- **Unit tests**: Individual functions and methods
- **Integration tests**: API endpoints with database
- **Test fixtures**: Reusable test data and mocks
- **Coverage reporting**: HTML reports in `htmlcov/`

Run specific tests:
```bash
pytest app/tests/api/routes/test_users.py  # Test specific file
pytest -k "test_create_user"               # Test by name pattern
pytest -x                                  # Stop on first failure
```

### Frontend Testing
- **E2E tests**: Playwright for user workflows
- **Component tests**: Critical UI components
- **API integration**: Test frontend-backend communication

## Configuration Management

### Environment Variables
Key configuration in `.env` file:
- **`SECRET_KEY`**: JWT token signing (change in production)
- **`FIRST_SUPERUSER`**: Initial admin user email
- **`POSTGRES_*`**: Database connection settings
- **`SMTP_*`**: Email configuration
- **`DOMAIN`**: Application domain for deployment

### Development vs Production
- **Local development**: Uses `docker-compose.override.yml`
- **Production**: Uses `docker-compose.yml` only
- **Environment detection**: `ENVIRONMENT` variable controls features

## Deployment

### Local with Custom Domain
```bash
# Update .env file
DOMAIN=localhost.tiangolo.com

# Start with custom domain routing
docker compose watch
# Access: http://dashboard.localhost.tiangolo.com
```

### Production Deployment
1. **Set up Traefik proxy** on server
2. **Configure environment variables** for production
3. **Deploy with Docker Compose**:
   ```bash
   docker compose -f docker-compose.yml up -d
   ```

### CI/CD with GitHub Actions
- **Staging**: Auto-deploy on push to `master`
- **Production**: Auto-deploy on release creation
- **Testing**: Run full test suite on PRs

## Code Quality Standards

### Backend (Python)
- **Ruff**: Linting and formatting
- **MyPy**: Static type checking
- **Pre-commit hooks**: Automated code quality checks
- **Coverage**: Aim for >90% test coverage

### Frontend (TypeScript)
- **Biome**: Linting and formatting
- **TypeScript strict mode**: Full type safety
- **Import organization**: Automatic import sorting

### Pre-commit Configuration
Pre-commit hooks automatically run:
- File size checks
- YAML/TOML validation
- Python formatting (Ruff)
- End-of-file fixing
- Trailing whitespace removal

## Common Workflows

### Adding a New API Endpoint
1. **Define model** in `app/models.py`
2. **Add CRUD operations** in `app/crud.py`
3. **Create route handler** in `app/api/routes/`
4. **Add tests** in `app/tests/api/routes/`
5. **Regenerate frontend client**: `npm run generate-client`

### Database Schema Changes
1. **Modify models** in `app/models.py`
2. **Create migration**: `alembic revision --autogenerate -m "Description"`
3. **Review migration file** in `app/alembic/versions/`
4. **Apply migration**: `alembic upgrade head`
5. **Test with fresh database**

### Frontend Component Development
1. **Create component** in appropriate `src/components/` subdirectory
2. **Add to routing** in `src/routes/` if needed
3. **Use generated API client** for backend communication
4. **Add error handling** with error boundaries
5. **Test user workflows** with Playwright

## Troubleshooting

### Common Issues
- **Port conflicts**: Check if services are running on 3000, 5173, 8000, 5432
- **Database connection**: Ensure PostgreSQL is running in Docker
- **API client sync**: Regenerate client after backend changes
- **Migration conflicts**: Check alembic version history

### Development Tips
- **API exploration**: Use interactive docs at `http://localhost:8000/docs`
- **Database inspection**: Use Adminer at `http://localhost:8080`
- **Email testing**: MailCatcher available during development
- **Log monitoring**: Use `docker compose logs -f <service>` for real-time logs
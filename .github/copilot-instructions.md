# Project Guidelines

## Overview

This is a Docker-based deployment of Kimai, a professional time tracking application. The setup includes MySQL database and the Kimai web application running in separate containers.

## Code Style

- **Frontend**: Uses pnpm/npm with Symfony Encore (Webpack) for asset building
- **Backend**: PHP 8.1+ with Symfony framework, follows PSR standards
- **Config**: Symfony YAML format in `kimai/config/packages/`
- **Tests**: PHPUnit in `kimai/tests/` directory

Reference key files:
- `kimai/webpack.config.js` for frontend build configuration
- `kimai/config/services.yaml` for Symfony service definitions
- `kimai/phpunit.xml.dist` for testing setup

## Architecture

- **Database Layer**: MySQL 8.0 with Doctrine ORM
- **Application Layer**: Symfony-based Kimai application
- **Frontend**: Twig templates with Webpack-compiled assets
- **API**: RESTful JSON API with stateless authentication
- **Security**: Multi-provider authentication (internal/LDAP), form-based login

Service boundaries:
- Database service: Persistent MySQL data with health checks
- Kimai service: Apache/PHP application server with mounted volumes

## Build and Test

**Prerequisites**: Docker and Docker Compose installed

**Local Development Setup**:
```bash
# Start containers
docker-compose up -d

# Create admin user (first time only)
docker exec timetracking-kimai-1 php /opt/kimai/bin/console kimai:user:create admin admin@example.com ROLE_SUPER_ADMIN admin123

# Access at http://localhost:8001
```

**Frontend Development**:
```bash
cd kimai
pnpm install          # Install dependencies (preferred) or npm install
pnpm run dev          # Development build
pnpm run watch        # Watch mode for development
pnpm run build        # Production build
pnpm run lint         # Lint assets
```

**Backend Development**:
```bash
cd kimai
composer install                           # Install PHP dependencies
docker exec timetracking-kimai-1 php /opt/kimai/bin/console cache:clear  # Clear cache
```

**Testing**:
```bash
cd kimai
./vendor/bin/phpunit                      # Run all tests
./vendor/bin/phpunit tests/Unit/          # Unit tests only
```

## Conventions

**Environment Configuration**:
- Use `.env` files for local development only
- Production should use environment variables or Docker secrets
- Never commit real credentials to version control

**Docker Best Practices**:
- Use health checks for service dependencies
- Mount volumes for persistent data (MySQL, Kimai var/, public/)
- Override environment variables in docker-compose.override.yml for local customization

**Security Requirements**:
- Change all default passwords before production deployment
- Set `TRUSTED_HOSTS` and `TRUSTED_PROXIES` for production
- Use a reverse proxy (Nginx/Caddy) for SSL termination in production
- Store `APP_SECRET` and database credentials securely

**Database Migrations**:
- Kimai handles migrations automatically on container startup
- Manual migration commands available via `docker exec` and `bin/console doctrine:migrations:*`

## Common Gotchas

- Port 8001 (not 8000) is used for the web interface
- First-time setup requires creating admin user manually
- Frontend assets require rebuild after changes (`pnpm run dev`)
- Database credentials in docker-compose.yml are for development only
- Container logs contain startup information including migration status

## File Structure

Key directories:
- `kimai/src/` - Application source code
- `kimai/config/` - Symfony configuration
- `kimai/assets/` - Frontend source files
- `kimai/public/` - Web root (mounted as volume)
- `kimai/var/` - Runtime files, cache, logs (mounted as volume)
- `kimai/migrations/` - Database migration files
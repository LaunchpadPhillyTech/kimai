# Development Workflow

A comprehensive guide for developers working on the Kimai Time Tracking application.

## 🚀 Getting Started

### First-time Setup

1. **Prerequisites**
   ```bash
   # Required software
   docker --version          # >= 20.x
   docker-compose --version  # >= 2.x
   git --version            # >= 2.x
   ```

2. **Clone and Initialize**
   ```bash
   git clone <repository-url> TimeTracking
   cd TimeTracking
   
   # Create environment file
   cp .env.example .env
   # Edit .env with your local settings
   
   # Start services
   docker-compose up -d
   
   # Create admin user
   docker exec timetracking-kimai-1 php /opt/kimai/bin/console kimai:user:create admin admin@example.com ROLE_SUPER_ADMIN admin123
   ```

3. **Verify Setup**
   - Open http://localhost:8001
   - Login with admin/admin123
   - Change default password

## 🛠️ Development Environment

### Frontend Development

The frontend uses Symfony Encore (Webpack) for modern asset building:

```bash
cd kimai

# Install dependencies (prefer pnpm for speed)
pnpm install
# or: npm install

# Development commands
pnpm run dev          # Single build for development
pnpm run watch        # Watch for changes and rebuild
pnpm run build        # Production build
pnpm run lint         # Lint JavaScript/CSS
pnpm run hot          # Hot module replacement (if available)

# Asset locations
# Source: assets/
# Output: public/build/
```

### Backend Development

PHP/Symfony backend development:

```bash
cd kimai

# Install PHP dependencies
composer install

# Useful Symfony commands
docker exec timetracking-kimai-1 php /opt/kimai/bin/console debug:router  # List routes
docker exec timetracking-kimai-1 php /opt/kimai/bin/console debug:container  # List services
docker exec timetracking-kimai-1 php /opt/kimai/bin/console cache:clear  # Clear cache
docker exec timetracking-kimai-1 php /opt/kimai/bin/console doctrine:migrations:status  # Migration status

# Code quality tools
./vendor/bin/phpstan analyse    # Static analysis
./vendor/bin/php-cs-fixer fix   # Code style fixes
```

### Database Management

```bash
# Access database console
docker exec -it timetracking-database-1 mysql -u kimai -p kimai

# Run migrations
docker exec timetracking-kimai-1 php /opt/kimai/bin/console doctrine:migrations:migrate

# Create migration
docker exec timetracking-kimai-1 php /opt/kimai/bin/console doctrine:migrations:generate

# Load fixtures (if available)
docker exec timetracking-kimai-1 php /opt/kimai/bin/console doctrine:fixtures:load
```

## 🧪 Testing

### Running Tests

```bash
cd kimai

# Run all tests
./vendor/bin/phpunit

# Specific test types
./vendor/bin/phpunit tests/Unit/          # Unit tests
./vendor/bin/phpunit tests/Integration/   # Integration tests
./vendor/bin/phpunit tests/Functional/    # Functional tests

# With coverage (requires xdebug)
./vendor/bin/phpunit --coverage-html coverage/

# Test specific file
./vendor/bin/phpunit tests/Unit/Entity/UserTest.php
```

### Writing Tests

Test structure follows Symfony conventions:
```php
// tests/Unit/Service/TimeCalculatorTest.php
<?php
namespace App\Tests\Unit\Service;

use App\Service\TimeCalculator;
use PHPUnit\Framework\TestCase;

class TimeCalculatorTest extends TestCase
{
    public function testCalculateDuration(): void
    {
        $calculator = new TimeCalculator();
        $result = $calculator->calculateDuration('09:00', '17:00');
        $this->assertEquals(8 * 3600, $result); // 8 hours in seconds
    }
}
```

### Frontend Testing

```bash
cd kimai

# JavaScript tests (if configured)
npm run test
npm run test:watch

# End-to-end tests (if configured)
npm run e2e
```

## 🔄 Git Workflow

### Branch Strategy

```bash
# Feature development
git checkout -b feature/user-dashboard
# ... make changes ...
git add .
git commit -m "feat: add user dashboard with time overview"
git push origin feature/user-dashboard

# Bug fixes
git checkout -b bugfix/timezone-calculation
# ... fix bug ...
git add .
git commit -m "fix: correct timezone calculation in reports"
git push origin bugfix/timezone-calculation

# Hotfixes (critical production issues)
git checkout main
git checkout -b hotfix/security-patch
# ... apply fix ...
git commit -m "security: patch XSS vulnerability in user input"
```

### Commit Message Convention

Follow conventional commits:
```
feat: add new feature
fix: bug fix
docs: documentation changes
style: code style changes (formatting, etc.)
refactor: code refactoring
test: adding or updating tests
chore: maintenance tasks
security: security-related changes
```

### Code Review Process

1. Create pull request with:
   - Clear description of changes
   - Screenshots for UI changes
   - Test coverage maintained
   - Documentation updated

2. PR checklist:
   - [ ] Tests pass
   - [ ] Code follows style guidelines
   - [ ] Documentation updated
   - [ ] No secrets or credentials committed
   - [ ] Database migrations (if any) are reversible

## 🏗️ Build and Deployment

### Local Build

```bash
# Frontend assets
cd kimai
pnpm run build

# Docker image (if needed)
docker build -t timetracking:local ./kimai
```

### Staging Deployment

```bash
# Pull latest images
docker-compose pull

# Deploy to staging
docker-compose -f docker-compose.yml -f docker-compose.staging.yml up -d

# Run migrations
docker exec timetracking-kimai-1 php /opt/kimai/bin/console doctrine:migrations:migrate --no-interaction

# Clear cache
docker exec timetracking-kimai-1 php /opt/kimai/bin/console cache:clear --env=prod
```

### Production Deployment

See [SECURITY.md](SECURITY.md) for production security considerations.

```bash
# Use production configuration
docker-compose -f docker-compose.yml -f docker-compose.production.yml up -d

# Verify deployment
docker-compose ps
curl -f http://localhost:8001/health || echo "Health check failed"
```

## 🔧 Development Tools

### Recommended VS Code Extensions

```json
{
  "recommendations": [
    "bmewburn.vscode-intelephense-client",
    "ms-vscode.vscode-typescript-next", 
    "bradlc.vscode-tailwindcss",
    "syler.sass-indented",
    "ms-vscode-remote.remote-containers",
    "ms-azuretools.vscode-docker"
  ]
}
```

### Debugging

#### PHP Debugging with Xdebug

```yaml
# docker-compose.override.yml for debugging
services:
  kimai:
    environment:
      - XDEBUG_MODE=debug
      - XDEBUG_CONFIG=client_host=host.docker.internal
```

#### Database Debugging

```bash
# Enable MySQL query logging
docker exec timetracking-database-1 mysql -u root -p -e "SET GLOBAL general_log = 'ON';"
docker exec timetracking-database-1 mysql -u root -p -e "SET GLOBAL general_log_file = '/var/lib/mysql/query.log';"

# View queries
docker exec timetracking-database-1 tail -f /var/lib/mysql/query.log
```

## 📊 Performance Monitoring

### Application Performance

```bash
# Monitor container resources
docker stats

# Application logs
docker-compose logs -f kimai

# Database performance
docker exec timetracking-database-1 mysql -u root -p -e "SHOW PROCESSLIST;"
```

### Optimization Tips

1. **Database**:
   - Add proper indexes for frequently queried columns
   - Use EXPLAIN to analyze query performance
   - Consider query caching for read-heavy operations

2. **Frontend**:
   - Minimize bundle size with code splitting
   - Optimize images and assets
   - Use browser caching headers

3. **Application**:
   - Enable Symfony cache in production
   - Use Redis/Memcached for session storage
   - Implement database connection pooling

## 🐛 Common Issues and Solutions

### Port Conflicts

```bash
# Find process using port 8001
lsof -i :8001  # macOS/Linux
netstat -ano | findstr :8001  # Windows

# Use different port
KIMAI_PORT=8002 docker-compose up -d
```

### Database Connection Issues

```bash
# Check database status
docker-compose logs database

# Recreate database container
docker-compose down
docker volume rm timetracking_mysql
docker-compose up -d
```

### Asset Build Issues

```bash
# Clear node_modules
cd kimai
rm -rf node_modules package-lock.json
npm install  # or pnpm install

# Clear Symfony cache
docker exec timetracking-kimai-1 php /opt/kimai/bin/console cache:clear
```

### Permission Issues

```bash
# Fix Docker volume permissions
docker exec --user root timetracking-kimai-1 chown -R www-data:www-data /opt/kimai/var
```

## 📚 Resources and Documentation

### Official Documentation

- [Kimai Documentation](https://www.kimai.org/documentation/)
- [Symfony Documentation](https://symfony.com/doc/current/index.html)
- [Docker Compose Reference](https://docs.docker.com/compose/)
- [Webpack Encore](https://symfony.com/doc/current/frontend.html)

### Internal Documentation

- [SECURITY.md](SECURITY.md) - Security guidelines and best practices
- [.github/copilot-instructions.md](.github/copilot-instructions.md) - AI assistant guidelines
- [project-detail.md](project-detail.md) - Detailed project analysis

### API Documentation

Access the API documentation at:
- Development: http://localhost:8001/api/doc
- Interactive API explorer included

### Community and Support

- [Kimai Forum](https://community.kimai.org/)
- [GitHub Issues](https://github.com/kimai/kimai/issues)
- [Discord Chat](https://discord.gg/kimai)

---

**Happy coding! 🚀**

For questions or issues with this development setup, please create an issue or contact the development team.
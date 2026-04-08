# Time Tracking Application (Kimai)

A professional Docker-based deployment of Kimai, the open-source time tracking application. This setup provides a complete development and production-ready environment with MySQL database and Kimai web application.

## Features

- 🕒 Professional time tracking with projects, tasks, and clients
- 👥 Multi-user support with role-based permissions
- 📊 Advanced reporting and data export capabilities
- 🔌 RESTful JSON API for integrations
- 🔒 Secure authentication (database, LDAP, SAML)
- 💰 Invoicing and rate management
- 🏷️ Tagging and categorization
- 📱 Responsive web interface
- 🌍 Multi-language support (30+ languages)
- 🔧 Extensible plugin system

## Quick Start

### Prerequisites

- Docker and Docker Compose installed
- Git (for cloning)
- Port 8001 available

### Installation

1. **Clone and Start**
   ```bash
   git clone <repository-url> TimeTracking
   cd TimeTracking
   docker-compose up -d
   ```

2. **Create Admin User**
   ```bash
   docker exec timetracking-kimai-1 php /opt/kimai/bin/console kimai:user:create admin admin@example.com ROLE_SUPER_ADMIN admin123
   ```

3. **Access Application**
   - Open http://localhost:8001
   - Login with username: `admin`, password: `admin123`
   - **Important**: Change the default password after first login

### Default Credentials

**Web Application Login:**
- Username: `admin`
- Password: `admin123`

**Database (Internal Use Only):**
- MySQL User: `kimai`
- MySQL Password: `kimai123`
- MySQL Root Password: `root123`

⚠️ **Security Note**: These are development credentials. Change all passwords before production deployment.

## Development Setup

### Frontend Development

The application uses Symfony Encore (Webpack) for asset building:

```bash
cd kimai
pnpm install          # Install dependencies (preferred over npm)
pnpm run dev          # Development build
pnpm run watch        # Watch mode for development
pnpm run build        # Production build
pnpm run lint         # Lint frontend code
```

### Backend Development

PHP backend with Symfony framework:

```bash
cd kimai
composer install                                    # Install PHP dependencies
docker exec timetracking-kimai-1 php /opt/kimai/bin/console cache:clear  # Clear cache
```

### Running Tests

```bash
cd kimai
./vendor/bin/phpunit                               # Run all tests
./vendor/bin/phpunit tests/Unit/                   # Unit tests only
./vendor/bin/phpunit tests/Integration/            # Integration tests
```

## Environment Configuration

### Local Development (.env)

For local development, you can create a `.env` file in the project root:

```env
# Database Configuration
MYSQL_ROOT_PASSWORD=your_root_password
MYSQL_DATABASE=kimai
MYSQL_USER=kimai
MYSQL_PASSWORD=your_db_password

# Kimai Configuration  
APP_ENV=dev
APP_SECRET=your_32_char_secret_key_here
DATABASE_URL=mysql://kimai:your_db_password@database/kimai?charset=utf8&serverVersion=8.0.32

# Security
TRUSTED_HOSTS=localhost,127.0.0.1
TRUSTED_PROXIES=

# Email (Optional)
MAILER_FROM=kimai@yourdomain.com
MAILER_URL=null://localhost
```

### Production Configuration

For production deployments:

1. **Use Environment Variables** instead of `.env` files
2. **Set Strong Passwords** for all accounts
3. **Configure HTTPS** with reverse proxy (Nginx/Caddy)
4. **Set Proper TRUSTED_HOSTS** and **TRUSTED_PROXIES**
5. **Use External Database** for better performance and backup
6. **Enable Email Services** for notifications

## Docker Services

- **database**: MySQL 8.0 with persistent storage
- **kimai**: Kimai application with Apache web server

### Useful Docker Commands

```bash
# View service status
docker-compose ps

# View logs
docker-compose logs kimai
docker-compose logs database

# Access container shell
docker exec -it timetracking-kimai-1 bash

# Restart services
docker-compose restart

# Stop everything
docker-compose down

# Update containers
docker-compose pull && docker-compose up -d
```

## Backup and Restore

### Database Backup

```bash
docker exec timetracking-database-1 mysqldump -u root -proot123 kimai > backup.sql
```

### Database Restore

```bash
docker exec -i timetracking-database-1 mysql -u root -proot123 kimai < backup.sql
```

### Application Data

Kimai data is stored in Docker volumes:
- `timetracking_mysql`: Database files
- `timetracking_public`: Uploaded files and assets
- `timetracking_var`: Cache, logs, and runtime files

## Troubleshooting

### Common Issues

1. **Cannot access http://localhost:8001**
   - Check if containers are running: `docker-compose ps`
   - Verify port 8001 is not in use: `netstat -an | findstr :8001`

2. **Login fails with correct credentials**
   - Ensure admin user was created successfully
   - Check container logs: `docker-compose logs kimai`

3. **Database connection errors**
   - Wait for database to fully start (can take 30-60 seconds)
   - Check database health: `docker-compose logs database`

4. **Frontend assets not loading**
   - Rebuild assets: `cd kimai && pnpm run dev`
   - Check for build errors in the console

### Getting Help

- Check container logs: `docker-compose logs`
- View Kimai documentation: https://www.kimai.org/documentation/
- Report issues: Create an issue with container logs and error details

## Security Best Practices

- [ ] Change default admin password after setup
- [ ] Use strong, unique passwords for all accounts
- [ ] Keep Docker images updated
- [ ] Configure firewall to restrict database access
- [ ] Enable HTTPS in production
- [ ] Regular backups of database and uploaded files
- [ ] Monitor application logs for suspicious activity

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly (run all tests)
5. Submit a pull request

## License

This project uses the Kimai application, which is licensed under the MIT License. See the [Kimai repository](https://github.com/kimai/kimai) for details.

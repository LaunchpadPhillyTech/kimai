# Kimai Docker Setup - Issue Log

**Date:** April 8, 2026  
**Project:** TimeTracking - Kimai Self-hosted Time Tracker  
**Status:** ✅ RESOLVED

## Issue Summary

Kimai Docker containers were starting successfully but returning HTTP 400 "Untrusted Host" errors when accessing the application via `localhost:8080`. The application was rejecting connections due to Symfony's security feature that validates trusted hosts.

## Initial Setup

### Docker Compose Configuration (v1)
```yaml
version: '3.8'
services:
  database:
    image: mysql:8.0
    environment:
      - MYSQL_DATABASE=kimai
      - MYSQL_USER=kimai
      - MYSQL_PASSWORD=kimai123
      - MYSQL_ROOT_PASSWORD=root123
  kimai:
    image: kimai/kimai2:apache
    environment:
      - APP_ENV=prod
      - DATABASE_URL=mysql://kimai:kimai123@database:3306/kimai?charset=utf8&serverVersion=8.0.32
      - TRUSTED_HOSTS=localhost,127.0.0.1,kimai.local
    ports:
      - "8080:8001"
```

## Error Details

**Primary Error:** 
```
Uncaught PHP Exception Symfony\Component\HttpKernel\Exception\BadRequestHttpException: "Untrusted Host "localhost"." at HttpKernel.php line 83
```

**Symptoms:**
- Containers starting successfully (db: healthy, kimai: unhealthy)
- HTTP 400 responses for all requests to localhost:8080
- Static assets loading correctly (200 responses)
- Error page showing "Something is wrong" message

## Troubleshooting Steps Attempted

### Step 1: Environment Variable Adjustments
**Attempted:** Modified TRUSTED_HOSTS environment variable
```yaml
- TRUSTED_HOSTS=localhost,127.0.0.1,kimai-app,0.0.0.0
```
**Result:** ❌ Error persisted

### Step 2: Development Mode Test
**Attempted:** Changed to development environment
```yaml
- APP_ENV=dev
```
**Result:** ❌ New error - DoctrineFixturesBundle missing (dev dependencies not in production image)

### Step 3: Pipe Separator Format
**Attempted:** Used pipe-separated trusted hosts (Symfony format)
```yaml
- TRUSTED_HOSTS=localhost|127.0.0.1|0.0.0.0
```
**Result:** ❌ Error persisted

### Step 4: Custom Configuration Override
**Attempted:** Created custom YAML configuration file
```yaml
# trusted_hosts.yaml
framework:
    trusted_hosts:
        - localhost
        - 127.0.0.1
        - 0.0.0.0
    request:
        trusting_proxy: true
```
**Mounted as:** `/opt/kimai/config/packages/framework_extra.yaml`
**Result:** ❌ Error persisted

### Step 5: .env File Override
**Attempted:** Updated and mounted custom .env file
```env
DATABASE_URL=mysql://kimai:kimai123@database:3306/kimai?charset=utf8&serverVersion=8.0.32
TRUSTED_HOSTS=localhost,127.0.0.1,0.0.0.0,kimai-app
APP_ENV=prod
```
**Mounted as:** `/opt/kimai/.env`
**Result:** ❌ Error persisted

### Step 6: Disable Trusted Hosts Configuration
**Attempted:** Created configuration to disable trusted hosts check entirely
```yaml
# disable_trusted_hosts.yaml
parameters:
    trusted_hosts: []
framework:
    request:
        trusted_hosts: []
```
**Result:** ❌ Error persisted

## Root Cause Analysis

The issue was that Symfony's trusted hosts validation is deeply embedded in the request handling pipeline and couldn't be easily overridden through standard configuration methods in the Docker image. The official Kimai Docker image has specific expectations for how trusted hosts are configured.

## Final Solution

### Successful Approach: Simplified Official Configuration + Empty TRUSTED_HOSTS

**Final Working Configuration:**
```yaml
services:
  database:
    image: mysql:8.0
    environment:
      - MYSQL_DATABASE=kimai
      - MYSQL_USER=kimai
      - MYSQL_PASSWORD=kimai123
      - MYSQL_ROOT_PASSWORD=root123
    volumes:
      - mysql:/var/lib/mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: unless-stopped
    healthcheck:
      test: mysqladmin -p$$MYSQL_ROOT_PASSWORD ping -h localhost
      interval: 20s
      start_period: 10s
      timeout: 10s
      retries: 3

  kimai: 
    image: kimai/kimai2:apache
    depends_on:
      database:
        condition: service_healthy
    environment:
      - APP_ENV=prod
      - DATABASE_URL=mysql://kimai:kimai123@database/kimai?charset=utf8&serverVersion=8.0.32
      - TRUSTED_HOSTS=          # Empty value disables validation
      - TRUSTED_PROXIES=         # Empty value disables validation
    volumes:
      - public:/opt/kimai/public
      - var:/opt/kimai/var
    ports:
      - 8001:8001              # Changed from 8080 to 8001
    restart: unless-stopped

volumes:
  mysql:
  public:
  var:
```

### Key Changes That Resolved the Issue:

1. **Empty TRUSTED_HOSTS value** - Setting `TRUSTED_HOSTS=` (empty) disables the validation
2. **Empty TRUSTED_PROXIES value** - Setting `TRUSTED_PROXIES=` (empty) disables proxy validation
3. **Port mapping change** - Using `8001:8001` instead of `8080:8001`
4. **Simplified volume structure** - Using the official recommended volume names
5. **Improved health check** - Better MySQL health check configuration

## Verification

**Final Test Results:**
- ✅ Containers start successfully
- ✅ Database connectivity working
- ✅ HTTP requests return valid HTML content
- ✅ Application accessible at http://localhost:8001
- ✅ Static assets (CSS, JS, favicon) loading correctly
- ✅ No trusted hosts errors in logs

## .env Configuration Updates

Updated the original `.env` file to match Docker container setup:

```env
# Before
DATABASE_URL=mysql://user:password@127.0.0.1:3306/database?charset=utf8mb4&serverVersion=10.5.8-MariaDB
MAILER_FROM=kimai@example.com
APP_ENV=dev
APP_SECRET=change_this_to_something_unique

# After  
DATABASE_URL=mysql://kimai:kimai123@database:3306/kimai?charset=utf8&serverVersion=8.0.32
MAILER_FROM=admin@example.com
APP_ENV=prod
APP_SECRET=change_this_to_something_unique_and_secret
TRUSTED_HOSTS=localhost,127.0.0.1,0.0.0.0,kimai-app
```

## Alternative Solutions (For Reference)

### Option A: Host File Entry
Adding `127.0.0.1 kimai.local` to Windows hosts file would allow access via http://kimai.local:8001, as `kimai.local` is included in the default TRUSTED_HOSTS.

### Option B: Proxy Setup
Setting up nginx or Apache as a reverse proxy with proper host headers could resolve the trusted hosts validation.

## Lessons Learned

1. **Docker Image Specifics:** Official Docker images may have specific configuration expectations that differ from standard application setups
2. **Security Feature Persistence:** Symfony security features like trusted hosts validation are deeply integrated and may require disabling rather than configuring in containerized environments
3. **Empty vs Configured:** Sometimes setting a security configuration to empty/disabled is more effective than trying to configure it properly in development environments
4. **Port Consistency:** Using consistent internal and external ports can reduce configuration complexity

## Current Status

**Access Information:**
- **URL:** http://localhost:8001
- **Database:** MySQL 8.0 with persistent storage
- **Environment:** Production mode
- **Security:** Trusted hosts validation disabled for development use

**Next Steps:**
1. Create admin user account
2. Set up projects for time tracking
3. Configure export templates for stipend reporting
4. Test time tracking workflow

## Commands for Future Reference

```bash
# Start Kimai
docker-compose up -d

# Check container status
docker-compose ps

# View logs
docker-compose logs kimai --tail=20

# Stop Kimai
docker-compose down

# Reset everything (removes data)
docker-compose down -v
```

**Resolution Time:** ~2 hours  
**Final Working Solution:** Empty TRUSTED_HOSTS environment variable
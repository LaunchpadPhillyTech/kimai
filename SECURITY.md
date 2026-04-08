# Security Guidelines and Best Practices

This document outlines security considerations and best practices for deploying and maintaining the Kimai Time Tracking application.

## 🔒 Quick Security Checklist

### Before Production Deployment

- [ ] Change ALL default passwords (database, admin user)
- [ ] Generate new APP_SECRET (32+ characters, cryptographically secure)
- [ ] Set proper TRUSTED_HOSTS and TRUSTED_PROXIES
- [ ] Configure HTTPS with valid SSL certificates
- [ ] Remove or secure database port exposure
- [ ] Set up automated backups with encryption
- [ ] Configure proper file permissions on volume mounts
- [ ] Enable application and server access logging
- [ ] Set up monitoring and alerting
- [ ] Review and limit user permissions

### Ongoing Security Maintenance

- [ ] Regular security updates for Docker images
- [ ] Monitor application logs for suspicious activity
- [ ] Regular database backups and backup testing
- [ ] Periodic security audits and penetration testing
- [ ] Password policy enforcement
- [ ] Review user access and permissions quarterly

## 🛡️ Authentication & Authorization

### Default Credentials (DEVELOPMENT ONLY)

**⚠️ CRITICAL**: These credentials are for development only and MUST be changed:

```
Admin User: admin / admin123
Database User: kimai / kimai123  
Database Root: root / root123
```

### Secure Password Policies

1. **Minimum Requirements**:
   - At least 12 characters
   - Mix of uppercase, lowercase, numbers, and symbols
   - Not based on dictionary words or personal information
   - Unique for each system/user

2. **Recommended Tools**:
   ```bash
   # Generate secure password
   openssl rand -base64 32
   
   # Generate APP_SECRET
   php -r "echo bin2hex(random_bytes(16)) . PHP_EOL;"
   ```

### Multi-Factor Authentication

Consider enabling 2FA for admin accounts:
- TOTP (Time-based One-Time Password)
- Hardware security keys
- SMS backup (less secure but better than nothing)

## 🔐 Environment Security

### Environment Variables vs Files

**Development**: Use `.env` files
```bash
# Copy and customize
cp .env.example .env
```

**Production**: Use environment variables or Docker secrets
```yaml
# Docker Compose with secrets
secrets:
  app_secret:
    file: ./secrets/app_secret.txt
```

### File Permissions

Ensure proper permissions on sensitive files:
```bash
# Secure environment files
chmod 600 .env
chown root:root .env

# Secure Docker secrets
chmod 400 ./secrets/*
chown root:root ./secrets/*
```

## 🌐 Network Security

### Reverse Proxy Configuration

**Nginx Example**:
```nginx
server {
    listen 443 ssl http2;
    server_name time.yourdomain.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
    
    location / {
        proxy_pass http://127.0.0.1:8001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Firewall Rules

Restrict network access:
```bash
# Only allow HTTP/HTTPS and SSH
ufw allow ssh
ufw allow http  
ufw allow https
ufw deny 3306  # Block direct database access
ufw enable
```

### Docker Network Isolation

Services are isolated in their own network:
- Database not accessible from outside Docker
- Application only exposes port 8001
- Consider using Docker secrets for credentials

## 📊 Database Security

### Connection Security

1. **Never expose MySQL port** (3306) directly to the internet
2. **Use encrypted connections** when possible
3. **Limit connection sources** to application containers only

### Backup Security

```bash
# Encrypted backup
docker exec timetracking-database-1 mysqldump -u root -p kimai | \
  gpg --cipher-algo AES256 --compress-algo 1 --symmetric > backup.sql.gpg

# Secure backup storage
chmod 600 backup.sql.gpg
```

### Database Hardening

1. Remove test databases and users
2. Set strong root password
3. Configure MySQL security settings:
   ```sql
   -- Disable remote root access
   DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');
   
   -- Remove anonymous users  
   DELETE FROM mysql.user WHERE User='';
   
   -- Flush privileges
   FLUSH PRIVILEGES;
   ```

## 🔍 Logging & Monitoring

### Application Logs

Monitor these log files for security events:
- Kimai application logs: `/opt/kimai/var/log/`
- Apache access logs: `/var/log/apache2/`
- MySQL logs: `/var/log/mysql/`

### Security Events to Monitor

```bash
# Failed login attempts
grep "Authentication failure" /opt/kimai/var/log/*.log

# Suspicious API requests
grep -E "(sql|script|javascript)" /var/log/apache2/access.log

# Large file uploads
grep -E "POST.*[0-9]{7,}" /var/log/apache2/access.log
```

### Automated Monitoring

Consider implementing:
- Log aggregation (ELK stack, Splunk)
- Intrusion detection (OSSEC, Suricata)  
- Uptime monitoring (Pingdom, StatusPage)
- Security scanning (OpenVAS, Nessus)

## 🚨 Incident Response

### Emergency Procedures

1. **Suspected Breach**:
   ```bash
   # Immediately isolate the system
   docker-compose down
   
   # Preserve evidence
   docker-compose logs > incident-$(date +%Y%m%d).log
   
   # Backup current state
   docker run --rm -v timetracking_mysql:/volume -v $(pwd):/backup alpine tar czf /backup/emergency-backup-$(date +%Y%m%d).tar.gz -C /volume ./
   ```

2. **Password Reset**:
   ```bash
   # Reset admin password
   docker exec timetracking-kimai-1 php /opt/kimai/bin/console kimai:user:password admin
   
   # Reset database password
   docker exec -it timetracking-database-1 mysql -u root -p
   ```

3. **Recovery Checklist**:
   - [ ] Identify and fix security vulnerability
   - [ ] Change all passwords and secrets
   - [ ] Review logs for extent of breach
   - [ ] Restore from clean backup if necessary
   - [ ] Update security measures
   - [ ] Document lessons learned

## 🔄 Updates & Maintenance

### Security Updates

1. **Docker Images**:
   ```bash
   # Check for updates
   docker-compose pull
   
   # Update with downtime
   docker-compose down && docker-compose up -d
   ```

2. **Kimai Application**:
   - Follow official update procedures
   - Test updates in staging environment first
   - Backup before applying updates

3. **System Packages**:
   ```bash
   # Update host system
   sudo apt update && sudo apt upgrade
   
   # Update Docker
   sudo apt update docker-ce docker-compose-plugin
   ```

### Vulnerability Scanning

Regular security assessments:
```bash
# Docker security scanning
docker scout quickview
docker scout cves

# Network port scanning
nmap -sS -O target_ip

# Web application scanning
nikto -h http://localhost:8001
```

## 📋 Compliance Considerations

### Data Protection

- **GDPR**: Implement data retention policies and user data export
- **SOX**: Maintain audit logs and access controls
- **HIPAA**: Encrypt data at rest and in transit (if applicable)

### Audit Trail

Kimai provides built-in audit logging:
- User login/logout events
- Time entry modifications
- Administrative actions
- Data exports

### Backup and Recovery

Test backup procedures regularly:
```bash
# Test backup integrity
docker exec -i timetracking-database-1 mysql -u root -p kimai < backup.sql

# Verify data consistency
docker exec timetracking-kimai-1 php /opt/kimai/bin/console kimai:version
```

## 🆘 Security Contacts

### Internal Security Team
- Security Officer: [contact@company.com]
- System Administrator: [sysadmin@company.com]
- Development Team: [dev@company.com]

### External Resources
- **Kimai Security**: https://www.kimai.org/documentation/security.html
- **Docker Security**: https://docs.docker.com/engine/security/
- **OWASP**: https://owasp.org/www-project-top-ten/

### Incident Reporting
1. Internal incident tracking system
2. External security researchers: security@company.com
3. Emergency contact: [emergency phone number]

---

**Last Updated**: [Current Date]  
**Next Review**: [Next Review Date]  
**Document Owner**: [Security Team/Admin]
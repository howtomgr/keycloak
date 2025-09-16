# keycloak Installation Guide

keycloak is a free and open-source open source identity and access management solution. Keycloak provides single sign-on (SSO) with identity brokering and social login, serving as a powerful FOSS alternative to Auth0, Okta, or Azure AD B2C

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2+ cores minimum
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: HTTPS for authentication flows
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8080 (default keycloak port)
  - Port 8443 for HTTPS
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install keycloak
sudo dnf install -y keycloak

# Enable and start service
sudo systemctl enable --now keycloak

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
keycloak.sh --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install keycloak
sudo apt install -y keycloak

# Enable and start service
sudo systemctl enable --now keycloak

# Configure firewall
sudo ufw allow 8080

# Verify installation
keycloak.sh --version
```

### Arch Linux

```bash
# Install keycloak
sudo pacman -S keycloak

# Enable and start service
sudo systemctl enable --now keycloak

# Verify installation
keycloak.sh --version
```

### Alpine Linux

```bash
# Install keycloak
apk add --no-cache keycloak

# Enable and start service
rc-update add keycloak default
rc-service keycloak start

# Verify installation
keycloak.sh --version
```

### openSUSE/SLES

```bash
# Install keycloak
sudo zypper install -y keycloak

# Enable and start service
sudo systemctl enable --now keycloak

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
keycloak.sh --version
```

### macOS

```bash
# Using Homebrew
brew install keycloak

# Start service
brew services start keycloak

# Verify installation
keycloak.sh --version
```

### FreeBSD

```bash
# Using pkg
pkg install keycloak

# Enable in rc.conf
echo 'keycloak_enable="YES"' >> /etc/rc.conf

# Start service
service keycloak start

# Verify installation
keycloak.sh --version
```

### Windows

```bash
# Using Chocolatey
choco install keycloak

# Or using Scoop
scoop install keycloak

# Verify installation
keycloak.sh --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/keycloak

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
keycloak.sh --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable keycloak

# Start service
sudo systemctl start keycloak

# Stop service
sudo systemctl stop keycloak

# Restart service
sudo systemctl restart keycloak

# Check status
sudo systemctl status keycloak

# View logs
sudo journalctl -u keycloak -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add keycloak default

# Start service
rc-service keycloak start

# Stop service
rc-service keycloak stop

# Restart service
rc-service keycloak restart

# Check status
rc-service keycloak status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'keycloak_enable="YES"' >> /etc/rc.conf

# Start service
service keycloak start

# Stop service
service keycloak stop

# Restart service
service keycloak restart

# Check status
service keycloak status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start keycloak
brew services stop keycloak
brew services restart keycloak

# Check status
brew services list | grep keycloak
```

### Windows Service Manager

```powershell
# Start service
net start keycloak

# Stop service
net stop keycloak

# Using PowerShell
Start-Service keycloak
Stop-Service keycloak
Restart-Service keycloak

# Check status
Get-Service keycloak
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream keycloak_backend {
    server 127.0.0.1:8080;
}

server {
    listen 80;
    server_name keycloak.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name keycloak.example.com;

    ssl_certificate /etc/ssl/certs/keycloak.example.com.crt;
    ssl_certificate_key /etc/ssl/private/keycloak.example.com.key;

    location / {
        proxy_pass http://keycloak_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName keycloak.example.com
    Redirect permanent / https://keycloak.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName keycloak.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/keycloak.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/keycloak.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend keycloak_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/keycloak.pem
    redirect scheme https if !{ ssl_fc }
    default_backend keycloak_backend

backend keycloak_backend
    balance roundrobin
    server keycloak1 127.0.0.1:8080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R keycloak:keycloak /etc/keycloak
sudo chmod 750 /etc/keycloak

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status keycloak

# View logs
sudo journalctl -u keycloak -f

# Monitor resource usage
top -p $(pgrep keycloak)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/keycloak"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/keycloak-backup-$DATE.tar.gz" /etc/keycloak /var/lib/keycloak

echo "Backup completed: $BACKUP_DIR/keycloak-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop keycloak

# Restore from backup
tar -xzf /backup/keycloak/keycloak-backup-*.tar.gz -C /

# Start service
sudo systemctl start keycloak
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u keycloak -n 100
sudo tail -f /var/log/keycloak/keycloak.log

# Check configuration
keycloak.sh --version

# Check permissions
ls -la /etc/keycloak
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8080

# Test connectivity
telnet localhost 8080

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep keycloak)

# Check disk I/O
iotop -p $(pgrep keycloak)

# Check connections
ss -an | grep 8080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  keycloak:
    image: keycloak:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/keycloak
      - ./data:/var/lib/keycloak
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update keycloak

# Debian/Ubuntu
sudo apt update && sudo apt upgrade keycloak

# Arch Linux
sudo pacman -Syu keycloak

# Alpine Linux
apk update && apk upgrade keycloak

# openSUSE
sudo zypper update keycloak

# FreeBSD
pkg update && pkg upgrade keycloak

# Always backup before updates
tar -czf /backup/keycloak-pre-update-$(date +%Y%m%d).tar.gz /etc/keycloak

# Restart after updates
sudo systemctl restart keycloak
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/keycloak

# Clean old logs
find /var/log/keycloak -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/keycloak
```

## Additional Resources

- Official Documentation: https://docs.keycloak.org/
- GitHub Repository: https://github.com/keycloak/keycloak
- Community Forum: https://forum.keycloak.org/
- Best Practices Guide: https://docs.keycloak.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

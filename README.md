# tiki Installation Guide

tiki is a free and open-source wiki groupware CMS. Tiki provides all-in-one wiki, CMS, and groupware solution

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
  - CPU: 1 core minimum
  - RAM: 1GB minimum
  - Storage: 2GB for data
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default tiki port)
  - None
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

# Install tiki
sudo dnf install -y tiki

# Enable and start service
sudo systemctl enable --now tiki

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
tiki --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install tiki
sudo apt install -y tiki

# Enable and start service
sudo systemctl enable --now tiki

# Configure firewall
sudo ufw allow 80

# Verify installation
tiki --version
```

### Arch Linux

```bash
# Install tiki
sudo pacman -S tiki

# Enable and start service
sudo systemctl enable --now tiki

# Verify installation
tiki --version
```

### Alpine Linux

```bash
# Install tiki
apk add --no-cache tiki

# Enable and start service
rc-update add tiki default
rc-service tiki start

# Verify installation
tiki --version
```

### openSUSE/SLES

```bash
# Install tiki
sudo zypper install -y tiki

# Enable and start service
sudo systemctl enable --now tiki

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
tiki --version
```

### macOS

```bash
# Using Homebrew
brew install tiki

# Start service
brew services start tiki

# Verify installation
tiki --version
```

### FreeBSD

```bash
# Using pkg
pkg install tiki

# Enable in rc.conf
echo 'tiki_enable="YES"' >> /etc/rc.conf

# Start service
service tiki start

# Verify installation
tiki --version
```

### Windows

```bash
# Using Chocolatey
choco install tiki

# Or using Scoop
scoop install tiki

# Verify installation
tiki --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/tiki

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
tiki --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable tiki

# Start service
sudo systemctl start tiki

# Stop service
sudo systemctl stop tiki

# Restart service
sudo systemctl restart tiki

# Check status
sudo systemctl status tiki

# View logs
sudo journalctl -u tiki -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add tiki default

# Start service
rc-service tiki start

# Stop service
rc-service tiki stop

# Restart service
rc-service tiki restart

# Check status
rc-service tiki status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'tiki_enable="YES"' >> /etc/rc.conf

# Start service
service tiki start

# Stop service
service tiki stop

# Restart service
service tiki restart

# Check status
service tiki status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start tiki
brew services stop tiki
brew services restart tiki

# Check status
brew services list | grep tiki
```

### Windows Service Manager

```powershell
# Start service
net start tiki

# Stop service
net stop tiki

# Using PowerShell
Start-Service tiki
Stop-Service tiki
Restart-Service tiki

# Check status
Get-Service tiki
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream tiki_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name tiki.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name tiki.example.com;

    ssl_certificate /etc/ssl/certs/tiki.example.com.crt;
    ssl_certificate_key /etc/ssl/private/tiki.example.com.key;

    location / {
        proxy_pass http://tiki_backend;
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
    ServerName tiki.example.com
    Redirect permanent / https://tiki.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName tiki.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/tiki.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/tiki.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend tiki_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/tiki.pem
    redirect scheme https if !{ ssl_fc }
    default_backend tiki_backend

backend tiki_backend
    balance roundrobin
    server tiki1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R tiki:tiki /etc/tiki
sudo chmod 750 /etc/tiki

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
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
sudo systemctl status tiki

# View logs
sudo journalctl -u tiki -f

# Monitor resource usage
top -p $(pgrep tiki)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/tiki"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/tiki-backup-$DATE.tar.gz" /etc/tiki /var/lib/tiki

echo "Backup completed: $BACKUP_DIR/tiki-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop tiki

# Restore from backup
tar -xzf /backup/tiki/tiki-backup-*.tar.gz -C /

# Start service
sudo systemctl start tiki
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u tiki -n 100
sudo tail -f /var/log/tiki/tiki.log

# Check configuration
tiki --version

# Check permissions
ls -la /etc/tiki
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep tiki)

# Check disk I/O
iotop -p $(pgrep tiki)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  tiki:
    image: tiki:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/tiki
      - ./data:/var/lib/tiki
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update tiki

# Debian/Ubuntu
sudo apt update && sudo apt upgrade tiki

# Arch Linux
sudo pacman -Syu tiki

# Alpine Linux
apk update && apk upgrade tiki

# openSUSE
sudo zypper update tiki

# FreeBSD
pkg update && pkg upgrade tiki

# Always backup before updates
tar -czf /backup/tiki-pre-update-$(date +%Y%m%d).tar.gz /etc/tiki

# Restart after updates
sudo systemctl restart tiki
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/tiki

# Clean old logs
find /var/log/tiki -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/tiki
```

## Additional Resources

- Official Documentation: https://docs.tiki.org/
- GitHub Repository: https://github.com/tiki/tiki
- Community Forum: https://forum.tiki.org/
- Best Practices Guide: https://docs.tiki.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

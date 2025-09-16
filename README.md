# gearman Installation Guide

gearman is a free and open-source job server. Gearman provides generic application framework for farming out work

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
  - RAM: 512MB minimum
  - Storage: 1GB for jobs
  - Network: Gearman protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 4730 (default gearman port)
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

# Install gearman
sudo dnf install -y gearman

# Enable and start service
sudo systemctl enable --now gearman

# Configure firewall
sudo firewall-cmd --permanent --add-port=4730/tcp
sudo firewall-cmd --reload

# Verify installation
gearman --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install gearman
sudo apt install -y gearman

# Enable and start service
sudo systemctl enable --now gearman

# Configure firewall
sudo ufw allow 4730

# Verify installation
gearman --version
```

### Arch Linux

```bash
# Install gearman
sudo pacman -S gearman

# Enable and start service
sudo systemctl enable --now gearman

# Verify installation
gearman --version
```

### Alpine Linux

```bash
# Install gearman
apk add --no-cache gearman

# Enable and start service
rc-update add gearman default
rc-service gearman start

# Verify installation
gearman --version
```

### openSUSE/SLES

```bash
# Install gearman
sudo zypper install -y gearman

# Enable and start service
sudo systemctl enable --now gearman

# Configure firewall
sudo firewall-cmd --permanent --add-port=4730/tcp
sudo firewall-cmd --reload

# Verify installation
gearman --version
```

### macOS

```bash
# Using Homebrew
brew install gearman

# Start service
brew services start gearman

# Verify installation
gearman --version
```

### FreeBSD

```bash
# Using pkg
pkg install gearman

# Enable in rc.conf
echo 'gearman_enable="YES"' >> /etc/rc.conf

# Start service
service gearman start

# Verify installation
gearman --version
```

### Windows

```bash
# Using Chocolatey
choco install gearman

# Or using Scoop
scoop install gearman

# Verify installation
gearman --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/gearman

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
gearman --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable gearman

# Start service
sudo systemctl start gearman

# Stop service
sudo systemctl stop gearman

# Restart service
sudo systemctl restart gearman

# Check status
sudo systemctl status gearman

# View logs
sudo journalctl -u gearman -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add gearman default

# Start service
rc-service gearman start

# Stop service
rc-service gearman stop

# Restart service
rc-service gearman restart

# Check status
rc-service gearman status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'gearman_enable="YES"' >> /etc/rc.conf

# Start service
service gearman start

# Stop service
service gearman stop

# Restart service
service gearman restart

# Check status
service gearman status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start gearman
brew services stop gearman
brew services restart gearman

# Check status
brew services list | grep gearman
```

### Windows Service Manager

```powershell
# Start service
net start gearman

# Stop service
net stop gearman

# Using PowerShell
Start-Service gearman
Stop-Service gearman
Restart-Service gearman

# Check status
Get-Service gearman
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream gearman_backend {
    server 127.0.0.1:4730;
}

server {
    listen 80;
    server_name gearman.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name gearman.example.com;

    ssl_certificate /etc/ssl/certs/gearman.example.com.crt;
    ssl_certificate_key /etc/ssl/private/gearman.example.com.key;

    location / {
        proxy_pass http://gearman_backend;
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
    ServerName gearman.example.com
    Redirect permanent / https://gearman.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName gearman.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/gearman.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/gearman.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:4730/
    ProxyPassReverse / http://127.0.0.1:4730/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend gearman_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/gearman.pem
    redirect scheme https if !{ ssl_fc }
    default_backend gearman_backend

backend gearman_backend
    balance roundrobin
    server gearman1 127.0.0.1:4730 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R gearman:gearman /etc/gearman
sudo chmod 750 /etc/gearman

# Configure firewall
sudo firewall-cmd --permanent --add-port=4730/tcp
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
sudo systemctl status gearman

# View logs
sudo journalctl -u gearman -f

# Monitor resource usage
top -p $(pgrep gearman)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/gearman"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/gearman-backup-$DATE.tar.gz" /etc/gearman /var/lib/gearman

echo "Backup completed: $BACKUP_DIR/gearman-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop gearman

# Restore from backup
tar -xzf /backup/gearman/gearman-backup-*.tar.gz -C /

# Start service
sudo systemctl start gearman
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u gearman -n 100
sudo tail -f /var/log/gearman/gearman.log

# Check configuration
gearman --version

# Check permissions
ls -la /etc/gearman
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 4730

# Test connectivity
telnet localhost 4730

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep gearman)

# Check disk I/O
iotop -p $(pgrep gearman)

# Check connections
ss -an | grep 4730
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  gearman:
    image: gearman:latest
    ports:
      - "4730:4730"
    volumes:
      - ./config:/etc/gearman
      - ./data:/var/lib/gearman
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update gearman

# Debian/Ubuntu
sudo apt update && sudo apt upgrade gearman

# Arch Linux
sudo pacman -Syu gearman

# Alpine Linux
apk update && apk upgrade gearman

# openSUSE
sudo zypper update gearman

# FreeBSD
pkg update && pkg upgrade gearman

# Always backup before updates
tar -czf /backup/gearman-pre-update-$(date +%Y%m%d).tar.gz /etc/gearman

# Restart after updates
sudo systemctl restart gearman
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/gearman

# Clean old logs
find /var/log/gearman -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/gearman
```

## Additional Resources

- Official Documentation: https://docs.gearman.org/
- GitHub Repository: https://github.com/gearman/gearman
- Community Forum: https://forum.gearman.org/
- Best Practices Guide: https://docs.gearman.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

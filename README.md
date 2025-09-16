# airsonic Installation Guide

airsonic is a free and open-source free music streaming server. Fork of Subsonic, Airsonic provides personal music streaming, serving as a FOSS alternative to proprietary options

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
  - Storage: 10GB+ for music
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 4040 (default airsonic port)
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

# Install airsonic
sudo dnf install -y airsonic

# Enable and start service
sudo systemctl enable --now airsonic

# Configure firewall
sudo firewall-cmd --permanent --add-port=4040/tcp
sudo firewall-cmd --reload

# Verify installation
airsonic --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install airsonic
sudo apt install -y airsonic

# Enable and start service
sudo systemctl enable --now airsonic

# Configure firewall
sudo ufw allow 4040

# Verify installation
airsonic --version
```

### Arch Linux

```bash
# Install airsonic
sudo pacman -S airsonic

# Enable and start service
sudo systemctl enable --now airsonic

# Verify installation
airsonic --version
```

### Alpine Linux

```bash
# Install airsonic
apk add --no-cache airsonic

# Enable and start service
rc-update add airsonic default
rc-service airsonic start

# Verify installation
airsonic --version
```

### openSUSE/SLES

```bash
# Install airsonic
sudo zypper install -y airsonic

# Enable and start service
sudo systemctl enable --now airsonic

# Configure firewall
sudo firewall-cmd --permanent --add-port=4040/tcp
sudo firewall-cmd --reload

# Verify installation
airsonic --version
```

### macOS

```bash
# Using Homebrew
brew install airsonic

# Start service
brew services start airsonic

# Verify installation
airsonic --version
```

### FreeBSD

```bash
# Using pkg
pkg install airsonic

# Enable in rc.conf
echo 'airsonic_enable="YES"' >> /etc/rc.conf

# Start service
service airsonic start

# Verify installation
airsonic --version
```

### Windows

```bash
# Using Chocolatey
choco install airsonic

# Or using Scoop
scoop install airsonic

# Verify installation
airsonic --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/airsonic

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
airsonic --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable airsonic

# Start service
sudo systemctl start airsonic

# Stop service
sudo systemctl stop airsonic

# Restart service
sudo systemctl restart airsonic

# Check status
sudo systemctl status airsonic

# View logs
sudo journalctl -u airsonic -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add airsonic default

# Start service
rc-service airsonic start

# Stop service
rc-service airsonic stop

# Restart service
rc-service airsonic restart

# Check status
rc-service airsonic status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'airsonic_enable="YES"' >> /etc/rc.conf

# Start service
service airsonic start

# Stop service
service airsonic stop

# Restart service
service airsonic restart

# Check status
service airsonic status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start airsonic
brew services stop airsonic
brew services restart airsonic

# Check status
brew services list | grep airsonic
```

### Windows Service Manager

```powershell
# Start service
net start airsonic

# Stop service
net stop airsonic

# Using PowerShell
Start-Service airsonic
Stop-Service airsonic
Restart-Service airsonic

# Check status
Get-Service airsonic
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream airsonic_backend {
    server 127.0.0.1:4040;
}

server {
    listen 80;
    server_name airsonic.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name airsonic.example.com;

    ssl_certificate /etc/ssl/certs/airsonic.example.com.crt;
    ssl_certificate_key /etc/ssl/private/airsonic.example.com.key;

    location / {
        proxy_pass http://airsonic_backend;
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
    ServerName airsonic.example.com
    Redirect permanent / https://airsonic.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName airsonic.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/airsonic.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/airsonic.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:4040/
    ProxyPassReverse / http://127.0.0.1:4040/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend airsonic_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/airsonic.pem
    redirect scheme https if !{ ssl_fc }
    default_backend airsonic_backend

backend airsonic_backend
    balance roundrobin
    server airsonic1 127.0.0.1:4040 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R airsonic:airsonic /etc/airsonic
sudo chmod 750 /etc/airsonic

# Configure firewall
sudo firewall-cmd --permanent --add-port=4040/tcp
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
sudo systemctl status airsonic

# View logs
sudo journalctl -u airsonic -f

# Monitor resource usage
top -p $(pgrep airsonic)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/airsonic"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/airsonic-backup-$DATE.tar.gz" /etc/airsonic /var/lib/airsonic

echo "Backup completed: $BACKUP_DIR/airsonic-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop airsonic

# Restore from backup
tar -xzf /backup/airsonic/airsonic-backup-*.tar.gz -C /

# Start service
sudo systemctl start airsonic
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u airsonic -n 100
sudo tail -f /var/log/airsonic/airsonic.log

# Check configuration
airsonic --version

# Check permissions
ls -la /etc/airsonic
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 4040

# Test connectivity
telnet localhost 4040

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep airsonic)

# Check disk I/O
iotop -p $(pgrep airsonic)

# Check connections
ss -an | grep 4040
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  airsonic:
    image: airsonic:latest
    ports:
      - "4040:4040"
    volumes:
      - ./config:/etc/airsonic
      - ./data:/var/lib/airsonic
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update airsonic

# Debian/Ubuntu
sudo apt update && sudo apt upgrade airsonic

# Arch Linux
sudo pacman -Syu airsonic

# Alpine Linux
apk update && apk upgrade airsonic

# openSUSE
sudo zypper update airsonic

# FreeBSD
pkg update && pkg upgrade airsonic

# Always backup before updates
tar -czf /backup/airsonic-pre-update-$(date +%Y%m%d).tar.gz /etc/airsonic

# Restart after updates
sudo systemctl restart airsonic
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/airsonic

# Clean old logs
find /var/log/airsonic -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/airsonic
```

## Additional Resources

- Official Documentation: https://docs.airsonic.org/
- GitHub Repository: https://github.com/airsonic/airsonic
- Community Forum: https://forum.airsonic.org/
- Best Practices Guide: https://docs.airsonic.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

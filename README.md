# Dada Mail Installation Guide

Dada Mail is a free and open-source Mailing List. Web-based list management system

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
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 80/443 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80/443 (default dada-mail port)
- **Dependencies**:
  - perl, perl-CGI
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

# Install dada-mail
sudo dnf install -y dada-mail perl, perl-CGI

# Enable and start service
sudo systemctl enable --now httpd

# Configure firewall
sudo firewall-cmd --permanent --add-service=dada-mail
sudo firewall-cmd --reload

# Verify installation
dada-mail --version || systemctl status httpd
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install dada-mail
sudo apt install -y dada-mail perl, perl-CGI

# Enable and start service
sudo systemctl enable --now httpd

# Configure firewall
sudo ufw allow 80/443

# Verify installation
dada-mail --version || systemctl status httpd
```

### Arch Linux

```bash
# Install dada-mail
sudo pacman -S dada-mail

# Enable and start service
sudo systemctl enable --now httpd

# Verify installation
dada-mail --version || systemctl status httpd
```

### Alpine Linux

```bash
# Install dada-mail
apk add --no-cache dada-mail

# Enable and start service
rc-update add httpd default
rc-service httpd start

# Verify installation
dada-mail --version || rc-service httpd status
```

### openSUSE/SLES

```bash
# Install dada-mail
sudo zypper install -y dada-mail perl, perl-CGI

# Enable and start service
sudo systemctl enable --now httpd

# Configure firewall
sudo firewall-cmd --permanent --add-service=dada-mail
sudo firewall-cmd --reload

# Verify installation
dada-mail --version || systemctl status httpd
```

### macOS

```bash
# Using Homebrew
brew install dada-mail

# Start service
brew services start dada-mail

# Verify installation
dada-mail --version
```

### FreeBSD

```bash
# Using pkg
pkg install dada-mail

# Enable in rc.conf
echo 'httpd_enable="YES"' >> /etc/rc.conf

# Start service
service httpd start

# Verify installation
dada-mail --version || service httpd status
```

### Windows

```powershell
# Using Chocolatey
choco install dada-mail

# Or using Scoop
scoop install dada-mail

# Verify installation
dada-mail --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /var/www/dada

# Set up basic configuration
sudo tee /var/www/dada/dada-mail.conf << 'EOF'
# Dada Mail Configuration
batch_sending_amount = 300
EOF

# Test configuration
sudo dada-mail -t || sudo httpd configtest

# Reload service
sudo systemctl reload httpd
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R dada-mail:dada-mail /var/www/dada
sudo chmod 750 /var/www/dada

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable httpd

# Start service
sudo systemctl start httpd

# Stop service
sudo systemctl stop httpd

# Restart service
sudo systemctl restart httpd

# Reload configuration
sudo systemctl reload httpd

# Check status
sudo systemctl status httpd

# View logs
sudo journalctl -u httpd -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add httpd default

# Start service
rc-service httpd start

# Stop service
rc-service httpd stop

# Restart service
rc-service httpd restart

# Check status
rc-service httpd status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'httpd_enable="YES"' >> /etc/rc.conf

# Start service
service httpd start

# Stop service
service httpd stop

# Restart service
service httpd restart

# Check status
service httpd status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start dada-mail
brew services stop dada-mail
brew services restart dada-mail

# Check status
brew services list | grep dada-mail
```

### Windows Service Manager

```powershell
# Start service
net start httpd

# Stop service
net stop httpd

# Using PowerShell
Start-Service httpd
Stop-Service httpd
Restart-Service httpd

# Check status
Get-Service httpd
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /var/www/dada/dada-mail.conf << 'EOF'
batch_sending_amount = 300
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart httpd
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream dada-mail_backend {
    server 127.0.0.1:80/443;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name dada-mail.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name dada-mail.example.com;

    ssl_certificate /etc/ssl/certs/dada-mail.example.com.crt;
    ssl_certificate_key /etc/ssl/private/dada-mail.example.com.key;

    location / {
        proxy_pass http://dada-mail_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName dada-mail.example.com
    Redirect permanent / https://dada-mail.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName dada-mail.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/dada-mail.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/dada-mail.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/443/
    ProxyPassReverse / http://127.0.0.1:80/443/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:80/443/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend dada-mail_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/dada-mail.pem
    redirect scheme https if !{ ssl_fc }
    default_backend dada-mail_backend

backend dada-mail_backend
    balance roundrobin
    option httpchk GET /health
    server dada-mail1 127.0.0.1:80/443 check
    server dada-mail2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R dada-mail:dada-mail /var/www/dada
sudo chmod 750 /var/www/dada

# Configure firewall
sudo firewall-cmd --permanent --add-service=dada-mail
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/dada-mail.conf << 'EOF'
[dada-mail]
enabled = true
port = 80/443
filter = dada-mail
logpath = /var/log/dada/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/dada-mail.key \
    -out /etc/ssl/certs/dada-mail.crt

# Configure SSL in dada-mail
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE dada-mail_db;
CREATE USER dada-mail_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE dada-mail_db TO dada-mail_user;
EOF

# Configure dada-mail to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE dada-mail_db;
CREATE USER 'dada-mail_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON dada-mail_db.* TO 'dada-mail_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Dada Mail specific tuning
batch_sending_amount = 300
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
dada-mail soft nofile 65535
dada-mail hard nofile 65535
dada-mail soft nproc 32768
dada-mail hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'dada-mail'
    static_configs:
      - targets: ['localhost:80/443']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet httpd; then
    echo "Dada Mail is running"
    exit 0
else
    echo "Dada Mail is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/dada-mail << 'EOF'
/var/log/dada/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 dada-mail dada-mail
    postrotate
        systemctl reload httpd > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Dada Mail backup script
BACKUP_DIR="/backup/dada-mail"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop httpd

# Backup configuration
tar -czf "$BACKUP_DIR/dada-mail-config-$DATE.tar.gz" /var/www/dada

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/dada-mail-data-$DATE.tar.gz" /var/lib/dada-mail

# Start service
systemctl start httpd

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop httpd

# Restore configuration
sudo tar -xzf /backup/dada-mail/dada-mail-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/dada-mail/dada-mail-data-*.tar.gz -C /

# Set permissions
sudo chown -R dada-mail:dada-mail /var/www/dada
sudo chown -R dada-mail:dada-mail /var/lib/dada-mail

# Start service
sudo systemctl start httpd
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u httpd -n 100
sudo tail -f /var/log/dada/*.log

# Check configuration
sudo dada-mail -t || sudo httpd configtest

# Check permissions
ls -la /var/www/dada
ls -la /var/lib/dada-mail
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80/443
sudo netstat -tlnp | grep 80/443

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 80/443
nc -zv localhost 80/443
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep httpd)
htop -p $(pgrep httpd)

# Check connections
ss -ant | grep :80/443 | wc -l

# Monitor I/O
iotop -p $(pgrep httpd)
```

### Debug Mode

```bash
# Run in debug mode
sudo dada-mail -d
# or
sudo httpd debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  dada-mail:
    image: dada-mail:latest
    container_name: dada-mail
    ports:
      - "80/443:80/443"
    volumes:
      - ./config:/var/www/dada
      - ./data:/var/lib/dada-mail
    environment:
      - dada-mail_CONFIG=/var/www/dada/dada-mail.conf
    restart: unless-stopped
    networks:
      - dada-mail_net

networks:
  dada-mail_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dada-mail
spec:
  replicas: 3
  selector:
    matchLabels:
      app: dada-mail
  template:
    metadata:
      labels:
        app: dada-mail
    spec:
      containers:
      - name: dada-mail
        image: dada-mail:latest
        ports:
        - containerPort: 80/443
        volumeMounts:
        - name: config
          mountPath: /var/www/dada
      volumes:
      - name: config
        configMap:
          name: dada-mail-config
---
apiVersion: v1
kind: Service
metadata:
  name: dada-mail
spec:
  selector:
    app: dada-mail
  ports:
  - port: 80/443
    targetPort: 80/443
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Dada Mail
  hosts: all
  become: yes
  tasks:
    - name: Install dada-mail
      package:
        name: dada-mail
        state: present
    
    - name: Configure dada-mail
      template:
        src: dada-mail.conf.j2
        dest: /var/www/dada/dada-mail.conf
        owner: dada-mail
        group: dada-mail
        mode: '0640'
      notify: restart dada-mail
    
    - name: Start and enable dada-mail
      systemd:
        name: httpd
        state: started
        enabled: yes
  
  handlers:
    - name: restart dada-mail
      systemd:
        name: httpd
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update dada-mail

# Debian/Ubuntu
sudo apt update && sudo apt upgrade dada-mail

# Arch Linux
sudo pacman -Syu dada-mail

# Alpine Linux
apk update && apk upgrade dada-mail

# openSUSE
sudo zypper update dada-mail

# FreeBSD
pkg update && pkg upgrade dada-mail

# Always backup before updates
tar -czf /backup/dada-mail-pre-update-$(date +%Y%m%d).tar.gz /var/www/dada

# Restart after updates
sudo systemctl restart httpd
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/dada -name "*.log" -mtime +30 -delete

# Verify integrity
sudo dada-mail --verify || sudo httpd check

# Update databases (if applicable)
sudo dada-mail-update-db

# Optimize performance
sudo dada-mail-optimize

# Check for security updates
sudo dada-mail --security-check
```

## Additional Resources

- Official Documentation: https://docs.dada-mail.org/
- GitHub Repository: https://github.com/dada-mail/dada-mail
- Community Forum: https://forum.dada-mail.org/
- Wiki: https://wiki.dada-mail.org/
- Comparison vs Mailman, phpList, Listmonk, Majordomo: https://docs.dada-mail.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

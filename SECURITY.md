### Default Credentials

| Service | Username | Password |
|---------|----------|----------|
| Grafana | `admin` | `admin123` |

**Change these credentials immediately in any non-local deployment.**

### Data Exposure

Metrics may reveal sensitive information about your system:
- Hostname and IP addresses
- Hardware configuration
- Running processes
- Network activity patterns
- Disk and file system details

### Recommendations

1. **Firewall Configuration**
   ```bash
   # Mac - only allow from monitoring server
   # Use macOS firewall or pf rules
   
   # Linux server - restrict Grafana access
   sudo ufw allow from 192.168.1.0/24 to any port 3000
   ```

2. **Change Default Passwords**
   ```yaml
   # docker-compose.yml
   environment:
     - GF_SECURITY_ADMIN_PASSWORD=your-secure-password
   ```

3. **Network Isolation**
   - Run on a private/home network
   - Use VPN for remote access
   - Don't expose ports to the internet

4. **TLS/HTTPS**
   - Use a reverse proxy (nginx, Traefik) with TLS
   - Consider self-signed certificates for internal use

## Reporting a Vulnerability

If you discover a security vulnerability in this project:

### How to Report

1. **Do NOT open a public issue** for security vulnerabilities
2. Use [GitHub Security Advisories](https://github.com/getangar/mac_monitoring/security/advisories/new) to report privately

### What to Include

- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

### Response Timeline

| Action | Timeframe |
|--------|-----------|
| Initial response | Within 72 hours |
| Status update | Within 7 days |
| Fix (if accepted) | Best effort basis |

### What to Expect

- **Accepted:** We'll work on a fix and credit you (unless you prefer anonymity)
- **Declined:** We'll explain why (e.g., out of scope, accepted risk for home use)

## Scope

### In Scope

- Configuration vulnerabilities
- Exposed sensitive data in examples
- Insecure default settings
- Documentation that could lead to insecure deployments

### Out of Scope

- Vulnerabilities in upstream software (Telegraf, Prometheus, Grafana, mactop)
  - Report these to respective projects
- Physical access attacks
- Social engineering
- Issues requiring already-compromised systems

## Security Best Practices for Users

### Mac (Monitored Machine)

```bash
# Restrict Telegraf to specific interface
[[outputs.prometheus_client]]
  listen = "192.168.1.10:9273"  # Instead of ":9273"
```

### Linux Server

```bash
# Bind Prometheus to localhost only
command:
  - '--web.listen-address=127.0.0.1:9090'

# Use Grafana behind reverse proxy with TLS
```

### Network

- Keep monitoring traffic on a separate VLAN if possible
- Use firewall rules to restrict access
- Monitor access logs for unauthorized attempts

---

Thank you for helping keep this project secure! 🔒

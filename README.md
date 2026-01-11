# 📋 **Wazuh Rules Deployment System**

A complete GitOps-based solution for automated Wazuh rule management that combines DaC (Detection-as-Code) principles with a centralized API layer for multi-server deployments.

![Detection-as-Code](https://img.shields.io/badge/Architecture-DaC-blue)
![Multi-Server](https://img.shields.io/badge/Deployment-Multi--Server-green)
![CI/CD](https://img.shields.io/badge/Automation-CI%2FCD-orange)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-red)

## 🏗️ **Architecture Overview**

```mermaid
graph TB
    subgraph "GitHub Repository (DaC)"
        GIT[GitHub Repository<br/>DaC]
        WORKFLOW[GitHub Actions<br/>Workflows]
        CHECKER[check_rule_ids.py<br/>Rule ID Validator]
        APPROVED[Approved Rulesets]
        RULES[rules/ directory]
        DECODERS[decoders/ directory]
    end
    
    subgraph "API Layer"
        API[FastAPI Server<br/>Central API]
        DB[(Deployment<br/>Logs DB)]
        AUTH[JWT/API Key<br/>Auth]
        CACHE[(Rules<br/>Cache)]
    end
    
    subgraph "Wazuh Server Cluster"
        S1[Wazuh Server 1]
        S2[Wazuh Server 2]
        SN[Wazuh Server N]
    end
    
    subgraph "Testing Pipeline"
        CI[CI Pipeline<br/>GitHub Actions]
        TEST[Test Environment]
        VALIDATE[Rule Validation]
    end
    
    GIT --> WORKFLOW
    WORKFLOW --> CI
    CI --> VALIDATE
    VALIDATE --> CHECKER
    
    GIT -- "Git Push" --> API
    API -- "Pull Approved Rules" --> APPROVED
    
    API -- "Log deployments" --> DB
    API -- "Authenticate" --> AUTH
    API -- "Cache rules" --> CACHE
    
    S1 -- "Authenticated API Calls" --> API
    S2 -- "Authenticated API Calls" --> API
    SN -- "Authenticated API Calls" --> API
    
    CI -- "Test deployment" --> TEST
    VALIDATE -- "Check conflicts" --> CHECKER
    CHECKER -- "Validate" --> RULES
    CHECKER -- "Validate" --> DECODERS
```

<!-- 
Place your architecture image here if you have one:
![Architecture Diagram](images/architecture.png) 
-->

## ✨ **Key Features**

### **Detection-as-Code (DaC) Framework**
- ✅ **Version-Controlled Rules** - All detection rules stored as code in Git
- ✅ **Automated Validation** - GitHub Actions validate rule IDs and syntax
- ✅ **Pull Request Workflow** - Review and approval process for all changes
- ✅ **CI/CD Pipeline** - Automated testing and deployment
- ✅ **Rule ID Conflict Prevention** - Automated checking with `check_rule_ids.py`

### **Multi-Server Deployment**
- ✅ **Centralized API** - Single FastAPI server managing multiple Wazuh instances
- ✅ **Secure Authentication** - JWT and API key-based authentication
- ✅ **Synchronized Updates** - Consistent rules across all servers
- ✅ **Deployment Logging** - Complete audit trail of all changes
- ✅ **Health Monitoring** - Real-time status of all Wazuh servers

### **Automation & Integration**
- ✅ **GitHub Actions** - Automated testing and deployment workflows
- ✅ **SSH Integration** - Secure file transfer to Wazuh servers
- ✅ **Rollback Capability** - Automatic backups and version control
- ✅ **Monitoring Dashboard** - Web-based interface for system status
- ✅ **API-First Design** - RESTful API for integration with other tools

## 📁 **Project Structure**

### **DaC Repository (Your Existing Git)**
```
DaC/                                  # Detection-as-Code Repository
├── .github/workflows/               # GitHub Actions CI/CD
│   ├── check_rule_ids.yml          # Rule ID validation workflow
│   ├── integrate_rulesets.yml      # Ruleset deployment workflow
│   └── test_deployment.yml         # Test environment deployment
├── Approved/                        # Approved rules for production
│   ├── rules/                      # Approved Wazuh rules (.xml)
│   └── decoders/                   # Approved Wazuh decoders (.xml)
├── rules/                          # Pending rules (under review)
├── decoders/                       # Pending decoders (under review)
├── check_rule_ids.py               # Rule ID conflict checker
├── rule_approval.json              # Rule approval tracking
├── .gitignore                      # Git ignore rules
└── README.md                       # Project documentation
```

### **API Server Repository**
```
wazuh-api-server/                    # Central API Server
├── api-server/                      # FastAPI application
│   ├── app/
│   │   ├── main.py                 # FastAPI application
│   │   ├── auth.py                 # JWT/API Key authentication
│   │   ├── models.py               # SQLAlchemy models
│   │   ├── schemas.py              # Pydantic schemas
│   │   └── routes/
│   │       ├── rules.py            # Rules management
│   │       ├── servers.py          # Server registration
│   │       └── deployments.py      # Deployment tracking
│   ├── Dockerfile                  # Container configuration
│   ├── requirements.txt            # Python dependencies
│   └── entrypoint.sh               # Startup script
├── deployment-scripts/             # Server deployment utilities
│   ├── register-server             # Server registration
│   ├── sync-rules                  # Rule synchronization
│   ├── backup-rules               # Backup creation
│   └── health-check               # Health monitoring
├── docker-compose.yml              # Container orchestration
├── docker-compose.test.yml         # Test environment
├── .env.example                    # Environment variables
└── README.md                       # API server documentation
```

## 🚀 **Quick Start**

### **Prerequisites**

1. **For DaC Repository:**
   - GitHub account with repository access
   - Wazuh server with SSH access
   - Public IP or NAT configuration for Wazuh server
   - SSH key pair for authentication

2. **For API Server:**
   - Docker and Docker Compose
   - PostgreSQL or SQLite database
   - Python 3.9+ (for local development)

### **1. DaC Repository Setup**

```bash
# Clone the DaC repository
git clone https://github.com/SamsonIdowu/DaC.git
cd DaC

# Configure the Wazuh server as a Git repository
cd /var/ossec/etc
git init
git config --global --add safe.directory /var/ossec/etc
git remote add origin https://github.com/your-org/DaC.git

# Configure Git identity
git config --global user.name "Wazuh Server"
git config --global user.email "wazuh@your-domain.com"

# Initial commit
git checkout -b main
git add rules/ decoders/
git commit -m "Initial commit: Wazuh rules and decoders"
git push -u origin main

# Create dev branch for testing
git checkout -b dev
git push -u origin dev
```

### **2. GitHub Repository Configuration**

1. **Configure GitHub Secrets:**
   - Navigate to: **Settings → Secrets and variables → Actions**
   - Add the following secrets:

   | **Name** | **Secret** |
   |----------|------------|
   | `USERNAME` | SSH username for Wazuh server |
   | `HOST` | Public IP of Wazuh server |
   | `SSH_KEY` | Private SSH key for authentication |
   | `PORT` | SSH port (default: 22) |
   | `API_KEY` | API key for API server communication |

2. **Configure GitHub Actions Workflows:**
   - Edit `.github/workflows/integrate_rulesets.yml` to match your Wazuh paths
   - Edit `.github/workflows/check_rule_ids.yml` for rule validation

### **3. API Server Deployment**

```bash
# Clone API server repository
git clone https://github.com/your-org/wazuh-api-server.git
cd wazuh-api-server

# Configure environment
cp .env.example .env
nano .env  # Edit with your configuration

# Start API server with Docker Compose
docker-compose up -d

# Verify API server is running
curl http://localhost:8000/health
```

### **4. Register Wazuh Servers**

```bash
# On each Wazuh server, register with the API
curl -X POST http://api-server:8000/api/servers/register \
  -H "Content-Type: application/json" \
  -d '{
    "server_name": "wazuh-prod-01",
    "server_group": "production",
    "ip_address": "192.168.1.100",
    "description": "Production Wazuh Server"
  }'

# Save the returned API key for future authentication
```

### **5. Configure Automatic Synchronization**

```bash
# On each Wazuh server, create sync script
cat > /usr/local/bin/sync-wazuh-rules << 'EOF'
#!/bin/bash
API_URL="http://api-server:8000"
API_KEY="your-server-api-key"

# Pull latest rules from API
curl -s -H "X-API-Key: $API_KEY" \
  "$API_URL/api/rules/latest" -o /tmp/latest-rules.tar.gz

# Extract and backup
tar -xzf /tmp/latest-rules.tar.gz -C /var/ossec/etc/
systemctl restart wazuh-manager
EOF

chmod +x /usr/local/bin/sync-wazuh-rules

# Schedule regular sync
(crontab -l 2>/dev/null; echo "*/5 * * * * /usr/local/bin/sync-wazuh-rules") | crontab -
```

## 📝 **Workflow: Detection-as-Code Process**

### **1. Development Workflow**

```mermaid
graph LR
    A[Create/Edit Rule] --> B[Submit Pull Request]
    B --> C[Automated Validation]
    C --> D{Validation Passed?}
    D -->|Yes| E[Manual Review]
    D -->|No| F[Fix Issues]
    F --> B
    E --> G{Approved?}
    G -->|Yes| H[Merge to main]
    G -->|No| I[Request Changes]
    I --> F
    H --> J[Deploy to Production]
```

### **2. Rule Creation Process**

```bash
# 1. Create feature branch
git checkout -b feature/new-webapp-rule

# 2. Create new rule file
cat > rules/webapp_security.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<group name="webapp,security">
  <rule id="100500" level="5">
    <if_sid>31101</if_sid>
    <regex>SQL injection|XSS|csrf</regex>
    <description>Web application attack detected</description>
    <group>web,attack,application</group>
  </rule>
</group>
EOF

# 3. Add and commit
git add rules/webapp_security.xml
git commit -m "Add web application security rule"

# 4. Push and create PR
git push origin feature/new-webapp-rule
# Create Pull Request via GitHub UI
```

### **3. Automated Validation**

The `check_rule_ids.py` script automatically validates:
- **Rule ID Conflicts**: Ensures no duplicate rule IDs
- **XML Syntax**: Validates XML structure and syntax
- **Required Fields**: Checks for mandatory rule attributes
- **ID Range Validation**: Verifies IDs are in allowed ranges

```bash
# Manual validation
python check_rule_ids.py --rules rules/ --decoders decoders/

# Expected output:
# ✅ No rule ID conflicts found
# ✅ All XML files are valid
# ✅ Rule IDs are in allowed ranges
```

## 🔧 **Configuration Details**

### **GitHub Actions Workflows**

#### **1. Rule ID Checker (check_rule_ids.yml)**
```yaml
name: Check Rule ID Conflicts

on:
  pull_request:
    branches: [ main ]
    paths: [ 'rules/**', 'decoders/**' ]

jobs:
  check-ids:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
          
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
          
      - name: Check rule ID conflicts
        run: python check_rule_ids.py
```

#### **2. Ruleset Integration (integrate_rulesets.yml)**
```yaml
name: Integrate Rulesets

on:
  push:
    branches: [ main ]
    paths: [ 'rules/**', 'decoders/**' ]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: SSH into Wazuh server
        uses: appleboy/ssh-action@v0.1.7
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          port: ${{ secrets.PORT }}
          script: |
            cd /var/ossec/etc/
            git pull origin main
            chown -R root:ossec rules/ decoders/
            chmod -R 640 rules/ decoders/
            systemctl restart wazuh-manager
            sleep 5
            systemctl status wazuh-manager
```

### **API Server Configuration**

```bash
# API Server .env configuration
DATABASE_URL=postgresql://user:password@localhost/wazuh_api
SECRET_KEY=your-super-secret-jwt-key
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# Git Repository
GIT_REPO_URL=https://github.com/your-org/DaC.git
GIT_REPO_BRANCH=main
APPROVED_RULES_PATH=Approved/rules
APPROVED_DECODERS_PATH=Approved/decoders

# Server Settings
SYNC_INTERVAL=300
LOG_LEVEL=INFO
HEALTH_CHECK_INTERVAL=60
```

### **Wazuh Server Configuration**

```json
{
  "api_server": "http://api-server:8000",
  "api_key": "your-server-api-key",
  "sync_interval": 300,
  "backup_enabled": true,
  "backup_count": 10,
  "rules_path": "/var/ossec/etc/rules",
  "decoders_path": "/var/ossec/etc/decoders",
  "log_file": "/var/log/wazuh-sync.log"
}
```

## 🔄 **Usage Guide**

### **Creating New Detection Rules**

1. **Rule Development:**
   ```bash
   # Create new rule in pending directory
   cat > rules/new_malware_detection.xml << 'EOF'
   <?xml version="1.0" encoding="UTF-8"?>
   <group name="malware,detection">
     <rule id="100501" level="10">
       <if_sid>100500</if_sid>
       <field name="win.eventdata.image">malware.exe</field>
       <description>Malware execution detected</description>
       <group>malware,execution</group>
     </rule>
   </group>
   EOF
   ```

2. **Submit for Review:**
   ```bash
   git add rules/new_malware_detection.xml
   git commit -m "Add malware detection rule"
   git push origin feature/malware-detection
   # Create Pull Request on GitHub
   ```

3. **Review and Approval:**
   - Automated validation runs via GitHub Actions
   - Security team reviews the rule
   - If approved, merge to main branch
   - Rules automatically deployed to production

### **Modifying Existing Rules**

```bash
# 1. Check out the rule file
git checkout -b fix/false-positive-rule
nano rules/existing_rule.xml

# 2. Modify the rule (example: adjust threshold)
# Change: <field name="count">5</field>
# To: <field name="count">10</field>

# 3. Submit changes
git add rules/existing_rule.xml
git commit -m "Adjust threshold to reduce false positives"
git push origin fix/false-positive-rule
```

### **Emergency Rule Deployment**

```bash
# 1. Create emergency branch
git checkout -b emergency/zero-day-fix

# 2. Add emergency rule
cat > rules/emergency_zero_day.xml << 'EOF'
<rule id="999999" level="15">
  <description>EMERGENCY: Zero-day exploit detected</description>
  <!-- Rule details -->
</rule>
EOF

# 3. Bypass normal process (if needed)
git add rules/emergency_zero_day.xml
git commit -m "EMERGENCY: Zero-day mitigation"
git push origin emergency/zero-day-fix

# 4. Manual deployment if needed
ssh wazuh-server "cd /var/ossec/etc && git pull && systemctl restart wazuh-manager"
```

## 📊 **Monitoring & Operations**

### **Dashboard Access**

```bash
# API Server Dashboard
curl http://api-server:8000/dashboard

# Server Status
curl -H "X-API-Key: YOUR_KEY" http://api-server:8000/api/servers/status

# Deployment History
curl -H "Authorization: Bearer YOUR_TOKEN" \
  http://api-server:8000/api/deployments
```

### **Health Checks**

```bash
# Comprehensive health check
#!/bin/bash
echo "=== Wazuh DaC System Health Check ==="

echo "1. Git Repository Status:"
curl -s https://api.github.com/repos/your-org/DaC | jq '.size'

echo "2. API Server Status:"
curl -s http://api-server:8000/health | jq '.'

echo "3. Wazuh Server Status:"
ssh wazuh-server "systemctl status wazuh-manager"

echo "4. Rule Synchronization:"
curl -s -H "X-API-Key: SERVER_KEY" \
  http://api-server:8000/api/rules/status | jq '.'

echo "5. Recent Deployments:"
curl -s -H "Authorization: Bearer ADMIN_TOKEN" \
  http://api-server:8000/api/deployments?limit=5 | jq '.'
```

### **Log Monitoring**

```bash
# Monitor API server logs
docker logs -f wazuh-api-server

# Monitor Wazuh sync logs
tail -f /var/log/wazuh-sync.log

# Monitor GitHub Actions
# Visit: https://github.com/your-org/DaC/actions

# Monitor rule deployments
curl -s http://api-server:8000/api/deployments/latest | jq '.'
```

## 🚨 **Troubleshooting Guide**

### **Common Issues**

| **Issue** | **Symptoms** | **Solution** |
|-----------|--------------|--------------|
| Rule ID Conflicts | GitHub Actions fails with conflict error | Run `check_rule_ids.py` locally, adjust IDs |
| SSH Connection Failed | GitHub Actions workflow fails to connect | Verify SSH key, firewall rules, and host accessibility |
| API Server Unreachable | Wazuh servers can't sync rules | Check API server status, network connectivity |
| Rule Syntax Errors | Wazuh fails to restart after deployment | Validate XML syntax, check Wazuh logs |
| Permission Denied | Files can't be written to /var/ossec/etc | Verify file ownership and permissions |
| Git Merge Conflicts | Cannot push or pull changes | Resolve conflicts manually, ensure team coordination |

### **Diagnostic Commands**

```bash
# Complete system diagnostics
wget -O - https://raw.githubusercontent.com/your-org/DaC/main/diagnostics.sh | bash

# Check each component
echo "=== System Diagnostics ==="
echo "1. Git Status:" && cd /var/ossec/etc && git status
echo "2. Wazuh Service:" && systemctl status wazuh-manager
echo "3. API Connectivity:" && curl -s http://api-server:8000/health
echo "4. Rule Files:" && ls -la /var/ossec/etc/rules/*.xml | wc -l
echo "5. Recent Logs:" && tail -20 /var/log/wazuh-sync.log

# Validate rule IDs
python check_rule_ids.py --rules /var/ossec/etc/rules --decoders /var/ossec/etc/decoders

# Test API communication
curl -H "X-API-Key: YOUR_KEY" \
  http://api-server:8000/api/rules/latest > /dev/null && echo "API: OK"
```

### **Recovery Procedures**

#### **Rule Rollback**
```bash
# 1. Identify the problematic commit
git log --oneline -10

# 2. Revert to previous version
git revert <commit-hash>

# 3. Force sync to all servers
curl -X POST -H "Authorization: Bearer ADMIN_TOKEN" \
  http://api-server:8000/api/rules/sync-all
```

#### **Server Recovery**
```bash
# 1. Stop Wazuh service
systemctl stop wazuh-manager

# 2. Restore from backup
cp /var/ossec/backup/rules-$(date +%Y%m%d).tar.gz /tmp/
tar -xzf /tmp/rules-*.tar.gz -C /var/ossec/etc/

# 3. Restart service
systemctl start wazuh-manager

# 4. Verify restoration
systemctl status wazuh-manager
```

## 🔒 **Security Best Practices**

### **Access Control**
- **Git Repository**: Role-based access control for rule modification
- **API Server**: JWT tokens with short expiration times
- **Wazuh Servers**: Unique API keys per server
- **SSH Access**: Key-based authentication only, disable password auth

### **Network Security**
```bash
# Recommended firewall configuration
# Allow SSH only from specific IPs
ufw allow from 192.168.1.0/24 to any port 22

# Allow API server communication
ufw allow from 10.0.0.0/8 to any port 8000

# Restrict Wazuh management ports
ufw allow from 192.168.1.0/24 to any port 1514-1515
```

### **Audit Logging**
```bash
# Enable comprehensive logging
# API Server logs
docker logs wazuh-api-server >> /var/log/wazuh-api.log

# Deployment logs
curl -H "Authorization: Bearer ADMIN_TOKEN" \
  http://api-server:8000/api/deployments/export > /var/log/deployments.json

# Git activity logs
cd /var/ossec/etc && git log --oneline --graph --all > /var/log/git-activity.log
```

### **Backup Strategy**
```bash
#!/bin/bash
# Automated backup script
BACKUP_DIR="/var/wazuh-backups"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup
tar -czf "$BACKUP_DIR/rules_$DATE.tar.gz" /var/ossec/etc/rules /var/ossec/etc/decoders

# Keep only last 30 days of backups
find "$BACKUP_DIR" -name "rules_*.tar.gz" -mtime +30 -delete

# Log backup activity
echo "$DATE: Backup created" >> /var/log/wazuh-backup.log
```

## 🔄 **Advanced Automation**

### **Custom GitHub Actions**

```yaml
# .github/workflows/auto-approve.yml
name: Auto-approve Minor Changes

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  auto-approve:
    runs-on: ubuntu-latest
    if: contains(github.event.pull_request.title, '[MINOR]')
    steps:
      - name: Auto-approve minor changes
        uses: hmarr/auto-approve-action@v2
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### **Integration with SIEM Dashboard**

```python
# dashboard_integration.py
import requests
import json

class WazuhDashboard:
    def __init__(self, api_url, api_key):
        self.api_url = api_url
        self.api_key = api_key
    
    def get_rule_statistics(self):
        """Get statistics about deployed rules"""
        response = requests.get(
            f"{self.api_url}/api/rules/stats",
            headers={"X-API-Key": self.api_key}
        )
        return response.json()
    
    def trigger_deployment(self, rule_ids):
        """Trigger deployment of specific rules"""
        response = requests.post(
            f"{self.api_url}/api/rules/deploy",
            headers={"X-API-Key": self.api_key},
            json={"rule_ids": rule_ids}
        )
        return response.json()
```

### **Slack Notifications**

```yaml
# .github/workflows/slack-notify.yml
name: Slack Notification

on:
  push:
    branches: [ main ]

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Send Slack notification
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          fields: repo,message,commit,author,action,eventName,ref,workflow
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

## 🤝 **Team Collaboration**

### **Branch Strategy**
```
main
├── dev
│   ├── feature/new-detection-rule
│   ├── bugfix/false-positive
│   └── enhancement/performance
└── hotfix/emergency-patch
```

### **Code Review Checklist**
- [ ] Rule ID is unique and in correct range
- [ ] XML syntax is valid
- [ ] Rule description is clear and specific
- [ ] Severity level is appropriate
- [ ] Test cases are provided (if applicable)
- [ ] Documentation is updated
- [ ] Backward compatibility maintained

### **Team Roles**
- **Detection Engineers**: Create and modify rules
- **Security Analysts**: Review and approve rules
- **DevOps Engineers**: Maintain infrastructure
- **SOC Analysts**: Test and validate rules

## 📚 **Learning Resources**

### **Wazuh Documentation**
- [Rule Development Guide](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/index.html)
- [Decoder Development](https://documentation.wazuh.com/current/user-manual/ruleset/custom-decoder.html)
- [Best Practices](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-best-practices.html)

### **GitHub Actions**
- [SSH Action Documentation](https://github.com/appleboy/ssh-action)
- [Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Secrets Management](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

### **Detection Engineering**
- [Sigma Rules](https://github.com/SigmaHQ/sigma) - Generic signature format
- [MITRE ATT&CK](https://attack.mitre.org/) - Framework for rule categorization
- [Detection Engineering Maturity Model](https://github.com/ThreatHuntingProject/ThreatHunting)

## 🚀 **Getting Help**

### **Support Channels**
- **GitHub Issues**: Bug reports and feature requests
- **Discord/Slack**: Community support
- **Email**: security-team@your-org.com
- **Documentation**: https://docs.your-org.com/wazuh-dac

### **Escalation Path**
1. Check troubleshooting guide in this README
2. Review GitHub Actions logs
3. Check system logs on Wazuh servers
4. Contact security team lead
5. Emergency hotline (for critical issues)

## 📄 **License**

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 **Acknowledgments**

- [Wazuh](https://wazuh.com/) - Open source security platform
- [GitHub Actions](https://github.com/features/actions) - CI/CD automation
- [FastAPI](https://fastapi.tiangolo.com/) - Modern Python web framework
- [Detection-as-Code Community](https://github.com/SigmaHQ/sigma) - Inspiration and best practices

## 📞 **Contact**

**Project Maintainers:**
- **Abhishek Kumar** - System Architecture & API Development
- **Samson Idowu** - DaC Framework & GitHub Actions
- **Security Team** - Rule Development & Validation

**Support Email:** security-engineering@your-org.com  
**Documentation:** https://docs.your-org.com/wazuh-dac  
**Community:** https://discord.gg/your-community

---

**⭐ If you find this project useful, please give it a star on GitHub!**

---

*Last Updated: December 2024*  
*Version: 3.0.0*  
*Wazuh Compatibility: 4.7.0+*  
*Detection-as-Code Framework: 1.0.0*

---

## 📸 **Image Placement Guide**

If you have custom images or diagrams you want to include in this documentation, here are the recommended locations:

### **For Architecture Diagrams:**
1. **Main architecture image**: Place at the top of the Architecture Overview section
   ```markdown
   ![Architecture Diagram](images/architecture.png)
   ```

2. **Workflow diagrams**: Place in relevant sections
   ```markdown
   ![Workflow Diagram](images/workflow.png)
   ```

3. **Component diagrams**: Place in configuration sections
   ```markdown
   ![Component Diagram](images/components.png)
   ```

### **Recommended Image Directory Structure:**
```
DaC/
├── images/
│   ├── architecture.png
│   ├── workflow.png
│   ├── components.png
│   ├── api-flow.png
│   └── deployment-process.png
├── screenshots/
│   ├── dashboard-01.png
│   ├── github-actions.png
│   └── monitoring.png
└── docs/
    └── diagrams/
        ├── sequence-diagrams/
        └── architecture-diagrams/
```

### **Image Optimization Tips:**
1. **Format**: Use PNG for diagrams, JPEG for screenshots
2. **Size**: Keep images under 500KB
3. **Resolution**: 800-1200px width for readability
4. **Alt Text**: Always include descriptive alt text
5. **Naming**: Use descriptive names (e.g., `api-deployment-flow.png`)

### **Example Image Reference:**
```markdown
<!-- Place your architecture image here if you have one:
![Wazuh Multi-Server Architecture](images/deepseek_mermaid_20251224_0bee61.png) 
-->
```

### **Generating Images:**
You can generate architecture diagrams using:
- **Mermaid.js**: For flowcharts and sequence diagrams (already used above)
- **draw.io**: For complex architecture diagrams
- **PlantUML**: For technical diagrams
- **Lucidchart**: For professional diagrams

### **Image Hosting Options:**
1. **GitHub**: Store images in `/images` directory (recommended)
2. **Cloud Storage**: Use services like AWS S3 or Cloudinary
3. **CDN**: Use a CDN for faster loading
4. **Embed from URL**: Use direct image URLs if hosted elsewhere

### **Accessibility Considerations:**
- Add descriptive alt text for screen readers
- Ensure sufficient color contrast
- Provide text alternatives for complex diagrams
- Include captions for context

Remember to update the image references in the README if you add or rename images in the repository.
```

**Instructions for use:**
1. Copy the entire content above
2. Create a new file named `README.md` in your GitHub repository
3. Paste the copied content
4. Save and commit the file
5. Update any placeholders (like `your-org`, `your-domain.com`) with your actual information
6. Add your images to an `images/` directory in your repository
7. Update the image references in the README to point to your actual image files

Below is the **ready-to-use Markdown (`README.md`) source code** for your GitHub repository.
You can **copy–paste this directly into `README.md`** in your repo and it will render correctly on GitHub (Mermaid diagrams included).

---

````md
# 📋 Wazuh Rules Deployment System

A complete **GitOps-based solution** for automated **Wazuh rule management**, combining **Detection-as-Code (DaC)** principles with a **centralized API layer** for secure, scalable **multi-server deployments**.

![Detection-as-Code](https://img.shields.io/badge/Architecture-DaC-blue)
![Multi-Server](https://img.shields.io/badge/Deployment-Multi--Server-green)
![CI/CD](https://img.shields.io/badge/Automation-CI%2FCD-orange)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-red)

---

## 🏗️ Architecture Overview

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
        DB[(Deployment Logs DB)]
        AUTH[JWT / API Key Auth]
        CACHE[(Rules Cache)]
    end
    
    subgraph "Wazuh Server Cluster"
        S1[Wazuh Server 1]
        S2[Wazuh Server 2]
        SN[Wazuh Server N]
    end
    
    subgraph "Testing Pipeline"
        CI[CI Pipeline]
        TEST[Test Environment]
        VALIDATE[Rule Validation]
    end
    
    GIT --> WORKFLOW
    WORKFLOW --> CI
    CI --> VALIDATE
    VALIDATE --> CHECKER
    
    GIT -- Git Push --> API
    API -- Pull Approved Rules --> APPROVED
    
    API --> DB
    API --> AUTH
    API --> CACHE
    
    S1 --> API
    S2 --> API
    SN --> API
    
    CI --> TEST
    CHECKER --> RULES
    CHECKER --> DECODERS
````

---

## ✨ Key Features

### Detection-as-Code (DaC)

* Version-controlled detection rules
* Automated validation (syntax & ID conflicts)
* Pull-request-based approval workflow
* CI/CD-powered deployments
* Rule ID conflict prevention

### Multi-Server Deployment

* Centralized FastAPI control plane
* Secure JWT & API key authentication
* Synchronized rule deployment
* Deployment history & auditing
* Server health monitoring

### Automation & Integration

* GitHub Actions CI/CD
* Secure SSH deployments
* Rollback & backup support
* Web-based monitoring
* API-first extensibility

---

## 📁 Project Structure

### Detection-as-Code Repository

```
DaC/
├── .github/workflows/
│   ├── check_rule_ids.yml
│   ├── integrate_rulesets.yml
│   └── test_deployment.yml
├── Approved/
│   ├── rules/
│   └── decoders/
├── rules/
├── decoders/
├── check_rule_ids.py
├── rule_approval.json
├── README.md
└── .gitignore
```

### API Server Repository

```
wazuh-api-server/
├── api-server/
│   ├── app/
│   │   ├── main.py
│   │   ├── auth.py
│   │   ├── models.py
│   │   ├── schemas.py
│   │   └── routes/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── entrypoint.sh
├── deployment-scripts/
├── docker-compose.yml
├── docker-compose.test.yml
├── .env.example
└── README.md
```

---

## 🚀 Quick Start

### Prerequisites

* GitHub account
* Wazuh 4.7.0+
* Docker & Docker Compose
* Python 3.9+
* SSH access to Wazuh servers

---

## 🔁 Detection-as-Code Workflow

```mermaid
graph LR
    A[Create Rule] --> B[Pull Request]
    B --> C[CI Validation]
    C --> D{Pass?}
    D -->|Yes| E[Manual Review]
    D -->|No| F[Fix Issues]
    E --> G{Approved?}
    G -->|Yes| H[Merge]
    H --> I[Deploy]
```

---

## 🔧 GitHub Actions – Rule ID Validation

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
      - uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      - run: python check_rule_ids.py
```

---

## 🧠 API Server Configuration

```env
DATABASE_URL=postgresql://user:password@localhost/wazuh_api
SECRET_KEY=super-secret-key
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

GIT_REPO_URL=https://github.com/your-org/DaC.git
GIT_REPO_BRANCH=main
APPROVED_RULES_PATH=Approved/rules
APPROVED_DECODERS_PATH=Approved/decoders
```

---

## 📊 Monitoring & Health

```bash
curl http://api-server:8000/health
curl -H "X-API-Key: KEY" http://api-server:8000/api/servers/status
curl -H "Authorization: Bearer TOKEN" http://api-server:8000/api/deployments
```

---

## 🚨 Troubleshooting

| Issue               | Cause          | Fix                     |
| ------------------- | -------------- | ----------------------- |
| Rule ID conflict    | Duplicate IDs  | Run `check_rule_ids.py` |
| SSH failure         | Key / firewall | Verify access           |
| API unreachable     | Network issue  | Check API status        |
| Wazuh restart fails | XML error      | Validate syntax         |

---

## 🔒 Security Best Practices

* RBAC for Git access
* JWT with short expiry
* Unique API keys per server
* SSH key-only authentication
* Network segmentation & firewalling

---

## 🤝 Team Roles

* Detection Engineers
* Security Analysts
* DevOps Engineers
* SOC Analysts

---

## 📄 License

MIT License – see `LICENSE`

---

## 🙏 Acknowledgments

* Wazuh
* GitHub Actions
* FastAPI
* Detection-as-Code Community

---

## 📞 Contact

**Maintainers**

* Abhishek Kumar – Architecture & API
* Samson Idowu – DaC & CI/CD

**Email:** [security-engineering@your-org.com](mailto:security-engineering@your-org.com)
**Docs:** [https://docs.your-org.com/wazuh-dac](https://docs.your-org.com/wazuh-dac)

---

⭐ *Star the repo if this helps your SOC!*

*Last Updated: December 2024*
*Version: 3.0.0*
*Wazuh: 4.7.0+*

```

---

If you want, I can also:
- 📦 Split this into **multiple docs** (`ARCHITECTURE.md`, `SECURITY.md`)
- 🧪 Add a **testing guide**
- 🖼 Convert Mermaid diagrams to **PNG**
- 🔄 Align this exactly with **Wazuh Rules-as-Code (RaC) blog**

Just tell me 👍
```

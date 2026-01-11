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

# Architecture

```
🌐 INTERNET LAYER
┌─────────────────────────────────────────────────────────────────────────────┐
│                            Cloudflare                                      │
│                     (CDN + DNS + WAF + DDoS)                              │
└─────────────────────────────┬───────────────────────────────────────────────┘
                              │
                              ▼
🔒 EDGE SECURITY LAYER
┌─────────────────────────────────────────────────────────────────────────────┐
│                         HAProxy/Traefik                                    │
│                    (SSL Termination + Load Balancing)                      │
│  ┌─────────────┬─────────────┬─────────────┬─────────────┬─────────────┐   │
│  │   Fail2ban  │  Suricata   │ ModSecurity │    OSSEC    │   ClamAV    │   │
│  │  (IP Ban)   │  (IDS/IPS)  │    (WAF)    │   (HIDS)    │ (AntiVirus) │   │
│  └─────────────┴─────────────┴─────────────┴─────────────┴─────────────┘   │
└─────────────────────────────┬───────────────────────────────────────────────┘
                              │
                              ▼
💻 INFRASTRUCTURE LAYER
┌─────────────────────────────────────────────────────────────────────────────┐
│                            GCP Compute                                     │
│  ┌─────────────┬─────────────┬─────────────┬─────────────┬─────────────┐   │
│  │   VM-PROD   │ VM-STAGING  │ VM-MONITOR  │ VM-SECURITY │  VM-DEVOPS  │   │
│  │             │             │             │             │             │   │
│  │ K3s Cluster │ K3s Cluster │ Monitoring  │ Vault +     │ ArgoCD +    │   │
│  │ + Apps      │ + Apps      │ Stack       │ Security    │ CI/CD       │   │
│  └─────────────┴─────────────┴─────────────┴─────────────┴─────────────┘   │
└─────────────────────────────┬───────────────────────────────────────────────┘
                              │
                              ▼
🔍 MONITORING & OBSERVABILITY LAYER
┌─────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────┬─────────────────────────┬───────────────────┐  │
│  │    METRICS STACK        │     LOGGING STACK       │   TRACING STACK   │  │
│  │  ┌─────────────────┐    │  ┌─────────────────┐    │ ┌───────────────┐ │  │
│  │  │   Prometheus    │    │  │ Elasticsearch   │    │ │    Jaeger     │ │  │
│  │  │   (Metrics)     │◄───┼──┤   (Storage)     │◄───┼─┤  (Tracing)    │ │  │
│  │  └─────────────────┘    │  └─────────────────┘    │ └───────────────┘ │  │
│  │  ┌─────────────────┐    │  ┌─────────────────┐    │ ┌───────────────┐ │  │
│  │  │    Grafana      │    │  │    Logstash     │    │ │     Loki      │ │  │
│  │  │  (Dashboard)    │    │  │   (Process)     │    │ │   (Logs)      │ │  │
│  │  └─────────────────┘    │  └─────────────────┘    │ └───────────────┘ │  │
│  │  ┌─────────────────┐    │  ┌─────────────────┐    │ ┌───────────────┐ │  │
│  │  │ AlertManager    │    │  │     Kibana      │    │ │     Tempo     │ │  │
│  │  │   (Alerts)      │    │  │   (Analyze)     │    │ │  (Tracing)    │ │  │
│  │  └─────────────────┘    │  └─────────────────┘    │ └───────────────┘ │  │
│  └─────────────────────────┴─────────────────────────┴───────────────────┘  │
└─────────────────────────────┬───────────────────────────────────────────────┘
                              │
                              ▼
⚙️ CI/CD & DEVOPS LAYER
┌─────────────────────────────────────────────────────────────────────────────┐
│                              GitHub                                         │
│  ┌─────────────┬─────────────┬─────────────┬─────────────┬─────────────┐   │
│  │ React Repo  │ Flask Repo  │Android Repo │  iOS Repo   │Personal Repo│   │
│  └─────┬───────┴─────┬───────┴─────┬───────┴─────┬───────┴─────┬───────┘   │
└────────┼─────────────┼─────────────┼─────────────┼─────────────┼───────────┘
         │             │             │             │             │
         ▼             ▼             ▼             ▼             ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         GitHub Actions CI/CD                               │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ PIPELINE STAGES:                                                    │   │
│  │ 1. Code Checkout                                                    │   │
│  │ 2. Security Scan (SAST: Semgrep, Bandit, Safety, Snyk)            │   │
│  │ 3. Quality Gate (SonarQube, Hadolint, Checkov, Gitleaks)          │   │
│  │ 4. Build & Test                                                    │   │
│  │ 5. Container Build & Scan (Trivy)                                  │   │
│  │ 6. Push to Harbor Registry                                         │   │
│  │ 7. Deploy via ArgoCD (GitOps)                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────┬───────────────────────────────────────────────┘
                              │
                              ▼
🔧 DEPLOYMENT & ORCHESTRATION LAYER  
┌─────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────┬─────────────────────────┬───────────────────┐  │
│  │      GITOPS STACK       │   INFRASTRUCTURE STACK  │  CONFIGURATION    │  │
│  │  ┌─────────────────┐    │  ┌─────────────────┐    │ ┌───────────────┐ │  │
│  │  │     ArgoCD      │    │  │   Terraform     │    │ │    Ansible    │ │  │
│  │  │   (Deploy)      │◄───┼──┤     (IaC)       │◄───┼─┤   (Config)    │ │  │
│  │  └─────────────────┘    │  └─────────────────┘    │ └───────────────┘ │  │
│  │  ┌─────────────────┐    │  ┌─────────────────┐    │ ┌───────────────┐ │  │
│  │  │     Flux v2     │    │  │      Helm       │    │ │   Portainer   │ │  │
│  │  │   (GitOps)      │    │  │   (Packages)    │    │ │     (UI)      │ │  │
│  │  └─────────────────┘    │  └─────────────────┘    │ └───────────────┘ │  │
│  └─────────────────────────┴─────────────────────────┴───────────────────┘  │
└─────────────────────────────┬───────────────────────────────────────────────┘
                              │
                              ▼
🏃 APPLICATION RUNTIME LAYER
┌─────────────────────────────────────────────────────────────────────────────┐
│                        K3s Kubernetes Clusters                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ PRODUCTION CLUSTER          │          STAGING CLUSTER              │   │
│  │  ┌─────────────────────┐    │    ┌─────────────────────┐            │   │
│  │  │   React Frontend    │    │    │   React Frontend    │            │   │
│  │  │   (React + Nginx)   │    │    │   (React + Nginx)   │            │   │
│  │  └─────────────────────┘    │    └─────────────────────┘            │   │
│  │  ┌─────────────────────┐    │    ┌─────────────────────┐            │   │
│  │  │   Flask Backend     │    │    │   Flask Backend     │            │   │
│  │  │  (Python + APIs)    │    │    │  (Python + APIs)    │            │   │
│  │  └─────────────────────┘    │    └─────────────────────┘            │   │
│  │  ┌─────────────────────┐    │    ┌─────────────────────┐            │   │
│  │  │   Personal Apps     │    │    │   Personal Apps     │            │   │
│  │  │  (Various Services) │    │    │  (Various Services) │            │   │
│  │  └─────────────────────┘    │    └─────────────────────┘            │   │
│  └─────────────────────────────┴─────────────────────────────────────────┘   │
└─────────────────────────────┬───────────────────────────────────────────────┘
                              │
                              ▼
📊 STATUS & ALERTING LAYER
┌─────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────┬─────────────────────────┬───────────────────┐  │
│  │    UPTIME MONITORING    │      STATUS PAGES       │     ALERTING      │  │
│  │  ┌─────────────────┐    │  ┌─────────────────┐    │ ┌───────────────┐ │  │
│  │  │   Uptime Kuma   │    │  │     Cachet      │    │ │   PagerDuty   │ │  │
│  │  │   (Monitor)     │    │  │  (Public Status)│    │ │   (Alerts)    │ │  │
│  │  └─────────────────┘    │  └─────────────────┘    │ └───────────────┘ │  │
│  │  ┌─────────────────┐    │  ┌─────────────────┐    │ ┌───────────────┐ │  │
│  │  │  UptimeRobot    │    │  │    Netdata      │    │ │    Discord    │ │  │
│  │  │  (External)     │    │  │  (Real-time)    │    │ │   (Alerts)    │ │  │
│  │  └─────────────────┘    │  └─────────────────┘    │ └───────────────┘ │  │
│  │  ┌─────────────────┐    │                         │ ┌───────────────┐ │  │
│  │  │     Sentry      │    │                         │ │     Email     │ │  │
│  │  │ (Error Track)   │    │                         │ │   (Alerts)    │ │  │
│  │  └─────────────────┘    │                         │ └───────────────┘ │  │
│  └─────────────────────────┴─────────────────────────┴───────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘

🔄 DATA FLOW DIAGRAM

┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│  CODE   │───▶│SECURITY │───▶│  BUILD  │───▶│ DEPLOY  │───▶│  LIVE   │
│ COMMIT  │    │  SCAN   │    │ & TEST  │    │ GitOps  │    │  APP    │
└─────────┘    └─────────┘    └─────────┘    └─────────┘    └─────────┘
     │              │              │              │              │
     ▼              ▼              ▼              ▼              ▼
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│  LOGS   │    │SECURITY │    │QUALITY  │    │INFRA    │    │METRICS  │
│METRICS  │◄───┤ALERTS   │◄───┤GATES    │◄───┤MONITOR  │◄───┤ LOGS    │
│TRACES   │    │ AUDIT   │    │REPORTS  │    │HEALTH   │    │ALERTS   │
└─────────┘    └─────────┘    └─────────┘    └─────────┘    └─────────┘

🚀 INTEGRATION POINTS

1. **Code → Security**: All commits trigger security scans
2. **Security → Monitoring**: Security events feed into SIEM
3. **Monitoring → Alerting**: Anomalies trigger incident response
4. **CI/CD → Deployment**: GitOps ensures secure deployments
5. **Runtime → Observability**: Apps send telemetry to monitoring
6. **Observability → Security**: Monitoring feeds security analytics
7. **All Systems → Status**: Health checks update public status

💰 **TOTAL COST BREAKDOWN**
- **Security Layer**: $15-25/month (HAProxy Pro optional)
- **Monitoring Layer**: $0-10/month (Sentry Pro optional)  
- **DevOps Layer**: $0-15/month (GitHub Pro optional)
- **Infrastructure**: $30-60/month (5 GCP VMs)
- **TOTAL**: $45-110/month

📈 **KEY BENEFITS**
- ✅ **Zero Trust Security**: 4-layer defense in depth
- ✅ **Full Observability**: Metrics, logs, traces, alerts
- ✅ **Automated DevOps**: GitOps with quality gates
- ✅ **Cost Effective**: Mostly free/open source tools
- ✅ **Enterprise Grade**: Production-ready architecture
- ✅ **Scalable**: Kubernetes-native applications

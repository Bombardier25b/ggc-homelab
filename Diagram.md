flowchart TD
    %% External Internet
    subgraph INTERNET
        I[Internet]
    end

    %% Edge router / firewall
    subgraph ROUTER[Router / Firewall]
        direction TB
        R[OPNsense / VyOS / pfSense]:::router
        R -->|NetFlow / IPFIX| NTOP[ntopng (traffic analytics)]
        R -->|Syslog / Alerts| PROM[Prometheus Exporter]
        R -->|Forwarded DNS| UNB[Unbound DNS]
    end

    %% Load‑balancing layer
    subgraph LB[Load‑Balancing (Skudonet / HAProxy)]
        direction TB
        LB_HA[Skudonet HAProxy]:::lb
        LB_HA -->|TLS termination| PRX[Traefik / Nginx Proxy]
    end

    %% Core Docker hosts
    subgraph HOSTS[Core Docker Hosts]
        direction LR
        VM_INFRA[VM‑infra]:::vm
        VM_APPS1[VM‑apps‑01]:::vm
        VM_APPS2[VM‑apps‑02]:::vm
        VM_SEC[VM‑sec]:::vm
        VM_LOG[VM‑log‑agg]:::vm
    end

    %% Containers on each host
    subgraph INFRA[Infra Containers (VM‑infra)]
        direction TB
        REVERSE[Reverse Proxy (Traefik/Nginx)]:::container
        DNS[Unbound DNS]:::container
        MON[Prometheus]:::container
        GRAF[Grafana]:::container
    end

    subgraph APPS1[App Containers (VM‑apps‑01)]
        direction TB
        DB[PostgreSQL]:::container
        REDIS[Redis]:::container
        WEB[Custom Web/API]:::container
    end

    subgraph APPS2[Batch / CI Containers (VM‑apps‑02)]
        direction TB
        JENK[Jenkins]:::container
        GITR[GitLab Runner]:::container
        BACKUP[Restic Backup Agent]:::container
    end

    subgraph SEC[Security Containers (VM‑sec)]
        direction TB
        WAZU[Wazuh Manager]:::container
        SURIC[Suricata IDS/IPS]:::container
        VAULT[HashiCorp Vault]:::container
    end

    subgraph LOG[Logging & AI (VM‑log‑agg)]
        direction TB
        LOKI[Loki]:::container
        ES[Elasticsearch (OSS)]:::container
        TSDB[TimescaleDB]:::container
        AI[Python Anomaly Detector]:::container
        ALERTM[Alertmanager]:::container
        DISCORD[Discord Bot]:::container
        SIGNAL[Signal Bot]:::container
    end

    %% Connections
    I --> R
    R --> LB_HA
    LB_HA --> REVERSE
    REVERSE --> DB
    REVERSE --> WEB
    REVERSE --> JENK
    REVERSE --> GITR

    %% Monitoring & logging paths
    R --> PROM
    PROM --> MON
    MON --> GRAF
    DB --> LOKI
    WEB --> LOKI
    JENK --> LOKI
    GITR --> LOKI
    WAZU --> LOKI
    SURIC --> LOKI
    LOKI --> ES
    ES --> TSDB
    TSDB --> AI
    AI --> ALERTM
    ALERTM --> DISCORD
    ALERTM --> SIGNAL

    classDef router fill:#ffcc00,stroke:#333,stroke-width:2px;
    classDef lb fill:#66c2a5,stroke:#333,stroke-width:2px;
    classDef vm fill:#8da0cb,stroke:#333,stroke-width:2px;
    classDef container fill:#e78ac3,stroke:#333,stroke-width:1px;

```mermaid
flowchart TB
    subgraph Azure["Azure AKS (Managed)"]
        A1[Control Plane (Azure-managed)]
        A2[Worker Node 0 - B2s VM]
        A3[Worker Node 1 - B2s VM]
        A4[Worker Node 2 - B2s VM]
        A5[Workload: NGINX (2 replicas)]
        A6[Storage: Azure Files (RWX)]
        A7[SOC: Microsoft Sentinel + Defender]
    end

    subgraph Proxmox["Bare-Metal Cluster (Proxmox VE)"]
        P1[PN64 - Master (kubeadm)]
        P2[NUC-1 - Worker]
        P3[NUC-2 - Worker]
        P4[Workload: NGINX (2 replicas)]
        P5[Storage: NFS / local-path PVC]
        P6[Backup: Raspberry Pi 4B (2 GB, NFS target)]
        P7[SOC: ELK on Minisforum Mini PC]
    end

    Azure -->|"Migration Runbook"| Proxmox
```


-------


--------



# 🚀 **Next Enhancements (Overview)**

### Step 1 — Infrastructure & Migration
- [ ] Expand AKS into a **private, policy-enforced** cluster (ACR, networking guardrails, IaC).
- [ ] Add a **migration runbook** to redeploy the same app on a **3-node bare-metal** cluster (Proxmox VE).
- [ ] Use **Raspberry Pi 4B (2 GB)** as an **NFS backup target** for Proxmox `vzdump` VM backups.

### Step 2 — Automation
- [ ] Use **GitHub Actions (OIDC)** for full CI/CD: **build → scan → push (ACR) → deploy → smoke test**.
- [ ] Apply **Azure Policy** during deployment for guardrails.
- [ ] Implement **Workload Identity + Key Vault CSI** for secretless deployments.

### Step 3 — Resilience
- [ ] Add **Velero backups** for AKS and run **chaos tests** (node/pod failures).
- [ ] Validate **Proxmox VM restores** from Raspberry Pi NFS storage.
- [ ] Document **recovery runbooks** (Ingress 5xx, Node NotReady, Policy Deny).

### Step 4 — SOC Layer
- [ ] Forward **AKS + Defender** logs into **Log Analytics + Microsoft Sentinel**.
- [ ] Create **analytic rules** and trigger **Logic Apps playbooks** for automated response.
- [ ] On bare-metal, forward logs via **Fluent Bit → ELK (Minisforum Mini PC)** for a SOC-lite setup.
- [ ] Build **Kibana dashboards** + basic detection rules (privileged pod creation, failed logins).


## ⚙️ Migration Topology (Azure ↔ Bare-Metal)
```text
┌───────────────────────────────────────────────┐   Migrate   ┌───────────────────────────────────────────────┐
│               Azure AKS (managed)             │  ───────▶   │               Bare-Metal K8s Cluster          │
│───────────────────────────────────────────────│             │───────────────────────────────────────────────│
│ Hypervisor:  Azure fabric (managed/hidden)    │             │ Hypervisor:  Proxmox VE                       │
│===============================================│             │===============================================│
│ Control Plane: Managed by Azure               │             │ Control Plane: PN64 (master, kubeadm)         │
│-----------------------------------------------│             │-----------------------------------------------│
│ Worker Node 0: Standard_B2s VM                │             │ Worker Node 1: NUC-1 (kubeadm worker)         │
│-----------------------------------------------│             │-----------------------------------------------│
│ Worker Node 1: Standard_B2s VM                │             │ Worker Node 2: NUC-2 (kubeadm worker)         │
│-----------------------------------------------│             │-----------------------------------------------│
│ • Worker Node 2: Standard_B2s VM              │             │ • Backup/Helper:  Raspberry Pi 4B (2 GB, NFS) │
│ • Workload: NGINX (2 replicas)                │             │ • Workload: NGINX (2 replicas)                │
│ • Storage:  Azure Disk PVC                    │             │ • Storage:  NFS / local-path PVC (RWX)        │
│ • Ingress:  Public LB + Ingress Controller    │             │ • Ingress:  MetalLB (L2) + Ingress Controller │
│ • bservability: Azure Monitor (Log Analytics) │             │ • Observability: Prometheus + Grafana         │
│ • SOC/Security: Microsoft Sentinel (+Defender)│             │ • SOC/Security: ELK on Minisforum Mini PC     │
│ • + Logic Apps playbooks                      │             │ • (Beats/Fluent Bit → Logstash → ES → Kibana) │
└───────────────────────────────────────────────┘             └───────────────────────────────────────────────┘
```
## 📊 Feature Mapping
| Layer / Feature | Azure (AKS)                                          | Bare-Metal                                                                           |
| --------------- | ---------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Hypervisor      | Azure fabric (managed/hidden)                        | **Proxmox VE**                                                                       |
| Control Plane   | **Managed by Azure**                                 | **PN64 master (kubeadm)**                                                            |
| Worker Node 0   | `Standard_B2s` VM                                    | **NUC-1 (kubeadm worker)**                                                           |
| Worker Node 1   | `Standard_B2s` VM                                    | **NUC-2 (kubeadm worker)**                                                           |
| Worker Node 2   | `Standard_B2s` VM                                    | **Raspberry Pi 4B (2 GB) – NFS backup target for Proxmox vzdump**                    |
| Workload        | **NGINX (2 replicas)**                               | **NGINX (2 replicas)**                                                               |
| Storage         | **Azure Files PVC (RWX)**                            | **NFS / local-path PVC (RWX)**                                                       |
| Ingress         | **Public LB + Ingress Controller**                   | **MetalLB (L2) + Ingress Controller**                                                |
| Observability   | **Azure Monitor Container Insights (Log Analytics)** | **Prometheus + Grafana**                                                             |
| SOC / Security  | **Microsoft Sentinel** (+ Defender, Logic Apps SOAR) | **ELK on Minisforum Mini PC** (Beats/Fluent Bit → Logstash → Elasticsearch → Kibana) |




-------


--------



# 🚀 **Next Enhancements (Overview)**

### Step 1 — Infrastructure & Migration
- [ ] Expand AKS into a **private, policy-enforced** cluster (ACR, networking guardrails, IaC).
- [ ] Add a **migration runbook** to redeploy the same app on a **3-node bare-metal** cluster (Proxmox VE).
- [ ] Use **Raspberry Pi 4B (2 GB)** as an **NFS backup target** for Proxmox `vzdump` VM backups.

### Step 2 — Automation
- [ ] Use **GitHub Actions (OIDC)** for full CI/CD: **build → scan → push (ACR) → deploy → smoke test**.
- [ ] Apply **Azure Policy** during deployment for guardrails.
- [ ] Implement **Workload Identity + Key Vault CSI** for secretless deployments.

### Step 3 — Resilience
- [ ] Add **Velero backups** for AKS and run **chaos tests** (node/pod failures).
- [ ] Validate **Proxmox VM restores** from Raspberry Pi NFS storage.
- [ ] Document **recovery runbooks** (Ingress 5xx, Node NotReady, Policy Deny).

### Step 4 — SOC Layer
- [ ] Forward **AKS + Defender** logs into **Log Analytics + Microsoft Sentinel**.
- [ ] Create **analytic rules** and trigger **Logic Apps playbooks** for automated response.
- [ ] On bare-metal, forward logs via **Fluent Bit → ELK (Minisforum Mini PC)** for a SOC-lite setup.
- [ ] Build **Kibana dashboards** + basic detection rules (privileged pod creation, failed logins).


## ⚙️ Migration Topology (Azure ↔ Bare-Metal)
```text
┌───────────────────────────────────────────────┐   Migrate   ┌───────────────────────────────────────────────┐
│               Azure AKS (managed)             │  ───────▶   │               Bare-Metal K8s Cluster          │
│───────────────────────────────────────────────│             │───────────────────────────────────────────────│
│ Hypervisor:  Azure fabric (managed/hidden)    │             │ Hypervisor:  Proxmox VE                       │
│===============================================│             │===============================================│
│ Control Plane: Managed by Azure               │             │ Control Plane: PN64 (master, kubeadm)         │
│-----------------------------------------------│             │-----------------------------------------------│
│ Worker Node 0: Standard_B2s VM                │             │ Worker Node 1: NUC-1 (kubeadm worker)         │
│-----------------------------------------------│             │-----------------------------------------------│
│ Worker Node 1: Standard_B2s VM                │             │ Worker Node 2: NUC-2 (kubeadm worker)         │
│-----------------------------------------------│             │-----------------------------------------------│
│ • Worker Node 2: Standard_B2s VM              │             │ • Backup/Helper:  Raspberry Pi 4B (2 GB, NFS) │
│ • Workload: NGINX (2 replicas)                │             │ • Workload: NGINX (2 replicas)                │
│ • Storage:  Azure Disk PVC                    │             │ • Storage:  NFS / local-path PVC (RWX)        │
│ • Ingress:  Public LB + Ingress Controller    │             │ • Ingress:  MetalLB (L2) + Ingress Controller │
│ • bservability: Azure Monitor (Log Analytics) │             │ • Observability: Prometheus + Grafana         │
│ • SOC/Security: Microsoft Sentinel (+Defender)│             │ • SOC/Security: ELK on Minisforum Mini PC     │
│ • + Logic Apps playbooks                      │             │ • (Beats/Fluent Bit → Logstash → ES → Kibana) │
└───────────────────────────────────────────────┘             └───────────────────────────────────────────────┘
```
## 📊 Feature Mapping
| Layer / Feature | Azure (AKS)                                          | Bare-Metal                                                                           |
| --------------- | ---------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Hypervisor      | Azure fabric (managed/hidden)                        | **Proxmox VE**                                                                       |
| Control Plane   | **Managed by Azure**                                 | **PN64 master (kubeadm)**                                                            |
| Worker Node 0   | `Standard_B2s` VM                                    | **NUC-1 (kubeadm worker)**                                                           |
| Worker Node 1   | `Standard_B2s` VM                                    | **NUC-2 (kubeadm worker)**                                                           |
| Worker Node 2   | `Standard_B2s` VM                                    | **Raspberry Pi 4B (2 GB) – NFS backup target for Proxmox vzdump**                    |
| Workload        | **NGINX (2 replicas)**                               | **NGINX (2 replicas)**                                                               |
| Storage         | **Azure Files PVC (RWX)**                            | **NFS / local-path PVC (RWX)**                                                       |
| Ingress         | **Public LB + Ingress Controller**                   | **MetalLB (L2) + Ingress Controller**                                                |
| Observability   | **Azure Monitor Container Insights (Log Analytics)** | **Prometheus + Grafana**                                                             |
| SOC / Security  | **Microsoft Sentinel** (+ Defender, Logic Apps SOAR) | **ELK on Minisforum Mini PC** (Beats/Fluent Bit → Logstash → Elasticsearch → Kibana) |

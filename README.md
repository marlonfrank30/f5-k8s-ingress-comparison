# 🌐 F5 Kubernetes Ingress & Egress Solutions — Technology Comparison

[![F5](https://img.shields.io/badge/F5-BIG--IP-red)](https://www.f5.com)
[![NGINX](https://img.shields.io/badge/NGINX-Ingress-green)](https://nginx.org)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-Ingress%20%26%20Gateway%20API-blue)](https://kubernetes.io)
[![Gateway API](https://img.shields.io/badge/Gateway%20API-CNCF%20Conformant-purple)](https://gateway-api.sigs.k8s.io)
[![Status](https://img.shields.io/badge/Status-Reference%20Design-success)](https://github.com)
[![License](https://img.shields.io/badge/License-Apache%202.0-lightgrey)](LICENSE)

---

## 🧭 Overview

This repository provides a **comprehensive, senior-level technology comparison** of all major F5 Kubernetes ingress and traffic management solutions. It is designed as a reference architecture guide to help platform engineers, architects, and DevSecOps teams make informed decisions when selecting an ingress solution for Kubernetes environments.

The five solutions compared are:

| Solution | Short Name | Data Plane | API Model |
|---|---|---|---|
| F5 NGINX Ingress Controller | **NIC** | NGINX OSS / Plus | Ingress API + CRDs |
| Community Ingress-NGINX | **ingress-nginx** ⚠️ EOL | NGINX OSS + Lua | Ingress API + Annotations |
| F5 NGINX Gateway Fabric | **NGF** | NGINX OSS / Plus | Gateway API (CNCF) |
| F5 BIG-IP Next for Kubernetes | **BNK** | F5 TMM (+ DPU) | Gateway API + F5 CRDs |
| F5 BIG-IP Container Ingress Services | **CIS** | F5 BIG-IP (ext.) | Ingress + CRDs + AS3 |

> ⚠️ **Important:** Community `kubernetes/ingress-nginx` reached **End of Life on March 2026**. No further security patches will be issued. Migration planning is urgent — see [Migration Guide](docs/migration-guide.md).

---

## 🔥 Why This Comparison Exists

Kubernetes ingress is not a commodity. Choosing the wrong solution creates technical debt, security gaps, and operational overhead. The F5 portfolio spans five distinct products serving very different use cases:

```
Client Traffic
      │
      ▼
┌─────────────────────────────────────────────────────────────┐
│                 F5 Ingress Solution Layer                   │
│                                                             │
│  ┌──────────┐  ┌─────────────┐  ┌──────────┐                │
│  │   NIC    │  │     NGF     │  │   BNK    │                │
│  │ (NGINX+) │  │ (GW API)    │  │ (TMM+DPU)│                │
│  └──────────┘  └─────────────┘  └──────────┘                │
│       │               │               │                     │
│  ┌────────────────────────────────────────┐                 │
│  │         CIS (External BIG-IP ADC)      │                 │
│  └────────────────────────────────────────┘                 │
└─────────────────────────────────────────────────────────────┘
      │
      ▼
 Kubernetes Services / Pods
```

Each product occupies a distinct position in the F5 portfolio:

- **NIC** — Production-proven NGINX-based ingress. Best drop-in replacement for ingress-nginx today.
- **NGF** — Future-forward Gateway API native ingress. Best for greenfield cloud-native platforms.
- **BNK** — High-performance, DPU-accelerated ingress for telco, 5G, and AI factory workloads.
- **CIS** — Enterprise connector for organisations with existing BIG-IP investments extending into Kubernetes.
- **ingress-nginx** — Legacy community project, EOL. Included for migration planning purposes only.

---

## 🏗 Architecture Diagrams

### NIC — NGINX Ingress Controller (In-Cluster)

```
External Client
      │
      ▼
┌─────────────────────────────────────────┐
│        NGINX Ingress Controller Pod     │
│    (nginxinc/kubernetes-ingress)        │
│                                         │
│  VirtualServer / VirtualServerRoute     │
│  TransportServer / Policy CRDs          │
│  ┌─────────────────────────────────┐    │
│  │        NGINX Data Plane         │    │
│  │  OSS (free) │ Plus (commercial) │    │
│  └─────────────────────────────────┘    │
└────────────────┬────────────────────────┘
                 │  Kubernetes Service
                 ▼
         Backend Pods / Services
```

### NGF — NGINX Gateway Fabric (Gateway API Native)

```
External Client
      │
      ▼
┌─────────────────────────────────────────┐
│      NGINX Gateway Fabric Pod           │
│    (nginx/nginx-gateway-fabric)         │
│                                         │
│  GatewayClass → Gateway → HTTPRoute     │
│  GRPCRoute / TLSRoute / TCPRoute        │
│  ┌─────────────────────────────────┐    │
│  │        NGINX Data Plane         │    │
│  │  3-role model: Infra / Ops / Dev│    │
│  └─────────────────────────────────┘    │
└────────────────┬────────────────────────┘
                 │  ReferenceGrant (cross-NS)
                 ▼
         Backend Pods / Services
```

### BNK — BIG-IP Next for Kubernetes (TMM + DPU)

```
External Client
      │
      ▼
┌─────────────────────────────────────────┐
│     BIG-IP Next for Kubernetes Pod      │
│         (F5 TMM Data Plane)             │
│                                         │
│  Gateway API + BNK CRDs:                │
│  CNEInstance / BNKSecPolicy             │
│  F5BigFwPolicy / F5SPKEgress            │
│  ┌──────────────────────────────────┐   │
│  │  Optional: NVIDIA BlueField DPU  │   │
│  │  (hardware offload: SSL, ACL)    │   │
│  └──────────────────────────────────┘   │
└────────────────┬────────────────────────┘
                 │  Kubernetes Service / SR-IOV
                 ▼
         Backend Pods / Services (Telco / AI)
```

### CIS — Container Ingress Services (External BIG-IP)

```
External Client
      │
      ▼
┌────────────────────────────┐
│    F5 BIG-IP (External)    │
│  Hardware Appliance or VE  │
│  TMM │ APM │ ASM │ AFM     │
│  iRules │ LTM │ GTM        │
└────────────┬───────────────┘
             │  REST / AS3
             ▼
┌────────────────────────────┐
│  CIS Controller Pod        │
│  (F5Networks/k8s-bigip-ctlr│
│                            │
│  Watches: Ingress,         │
│  VirtualServer, Routes,    │
│  ConfigMap AS3, ExternalDNS│
└────────────┬───────────────┘
             │  Kubernetes API
             ▼
     Kubernetes Services / Pods
```

---

## 📊 Feature Comparison Matrix

📥 **[Download the full Excel spreadsheet](assets/f5-k8s-ingress-comparison-matrix.xlsx)** — colour-coded with 130+ features across 12 categories.

<details>
<summary><b>📊 Click to expand — Full Feature Comparison Matrix (130+ features)</b></summary>

<div style="max-width:100%; overflow-x:auto;">

<table style="width:100%; table-layout:fixed; font-size:14px;">
<thead>
<tr>
<th style="width:22%">Feature / Category</th>
<th style="width:15%">NGINX IC (NIC)</th>
<th style="width:15%">Ingress-NGINX ⚠️</th>
<th style="width:16%">Gateway Fabric (NGF)</th>
<th style="width:16%">BIG-IP Next K8s (BNK)</th>
<th style="width:16%">BIG-IP CIS</th>
</tr>
</thead>

<tbody>

<tr>
<th colspan="6" align="left">🔷 TARGET USE CASE</th>
</tr>

<tr>
<th colspan="6" align="left">🔷 GENERAL OVERVIEW</th>
</tr>

<tr>
<td><b>Maintained By</b></td>
<td>F5 / NGINX Inc. (dedicated team)</td>
<td>Kubernetes Community (EOL Mar 2026)</td>
<td>F5 / NGINX Inc. (dedicated team)</td>
<td>F5 (dedicated engineering team)</td>
<td>F5 (open-source — included with BIG-IP support)</td>
</tr>

<tr>
<td><b>GitHub Repository</b></td>
<td>nginx/kubernetes-ingress</td>
<td>kubernetes/ingress-nginx</td>
<td>nginx/nginx-gateway-fabric</td>
<td>❌ No (F5 Artifactory)</td>
<td>F5Networks/k8s-bigip-ctlr<br>clouddocs.f5.com/containers/latest</td>
</tr>

<tr>
<td><b>License</b></td>
<td>Apache 2.0 (OSS)<br>+ Commercial (Plus)</td>
<td>Apache 2.0 (OSS)</td>
<td>Apache 2.0 (OSS)<br>+ Commercial (Plus)</td>
<td>💰 Commercial — MyF5 entitlement required</td>
<td>Apache 2.0 (OSS)<br>Support via BIG-IP entitlement</td>
</tr>

<tr>
<td><b>Kubernetes API Used</b></td>
<td>Ingress API + CRDs<br>(VirtualServer, etc.)</td>
<td>Ingress API + Annotations</td>
<td>Gateway API<br>(GatewayClass, Gateway, Routes)</td>
<td>Gateway API<br>+ F5 CRDs</td>
<td>Ingress + OpenShift Routes<br>+ F5 CRDs + AS3</td>
</tr>

<tr>
<td><b>Data Plane</b></td>
<td>NGINX Open Source<br>or NGINX Plus</td>
<td>NGINX Open Source<br>(Lua extensions)</td>
<td>NGINX Open Source<br>or NGINX Plus</td>
<td>F5 TMM data plane<br>DPU-accelerated optional</td>
<td>External BIG-IP (hardware or VE)<br>TMM data plane</td>
</tr>

<tr>
<td><b>Maturity / Status</b></td>
<td>✅ Actively developed<br>Production-grade</td>
<td>⚠️ Feature frozen<br>EOL March 2026</td>
<td>✅ Actively developed<br>Future-forward</td>
<td>✅ Actively developed<br>Future-forward</td>
<td>✅ Actively developed<br>Enterprise production-grade</td>
</tr>

<tr>
<td><b>Market Adoption</b></td>
<td>~40% Kubernetes ingress deployments</td>
<td>~41% historically<br>declining post-EOL</td>
<td>Growing adoption<br>Gateway API conformant</td>
<td>Growing enterprise adoption</td>
<td>Established in enterprises with BIG-IP</td>
</tr>

<tr>
<td><b>Commercial Tier</b></td>
<td>✅ NGINX Plus available</td>
<td>❌ None</td>
<td>✅ NGINX Plus / NGINX One</td>
<td>✅ Enterprise subscription</td>
<td>💰 Included with BIG-IP license</td>
</tr>

<tr>
<td><b>CNCF Conformance</b></td>
<td>N/A (Ingress API)</td>
<td>N/A (Ingress API)</td>
<td>✅ Gateway API conformant</td>
<td>✅ Gateway API v1.2 conformant</td>
<td>⚠️ Not Gateway API data plane</td>
</tr>

<tr>
<th colspan="6" align="left">🔷 CONFIGURATION MODEL</th>
</tr>

<tr>
<td><b>Configuration Approach</b></td>
<td>CRDs + Ingress resources</td>
<td>Ingress + annotations + ConfigMap</td>
<td>Gateway API resources</td>
<td>Gateway API + F5 CRDs</td>
<td>CRDs + AS3 + Ingress + Routes</td>
</tr>

<tr>
<td><b>CRDs</b></td>
<td>VirtualServer<br>TransportServer<br>Policy</td>
<td>⚠️ Minimal</td>
<td>Gateway API CRDs</td>
<td>Extensive F5 CRDs</td>
<td>VirtualServer<br>TransportServer<br>Policy</td>
</tr>

<tr>
<td><b>Annotation Config</b></td>
<td>Supported</td>
<td>Primary configuration method</td>
<td>❌ None</td>
<td>❌ None</td>
<td>Supported</td>
</tr>

<tr>
<td><b>Schema Validation</b></td>
<td>CRD schema validation</td>
<td>⚠️ Limited</td>
<td>Gateway API schema</td>
<td>Gateway API schema</td>
<td>CRD + AS3 JSON schema</td>
</tr>

<tr>
<td><b>GitOps Compatible</b></td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>✅ Yes</td>
</tr>

<tr>
<td><b>Helm Deployment</b></td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>✅ Yes</td>
</tr>

<tr>
<th colspan="6" align="left">🔷 TRAFFIC ROUTING</th>
</tr>

<tr>
<td><b>HTTP/HTTPS Routing</b></td>
<td>Host + path routing</td>
<td>Host + path routing</td>
<td>HTTPRoute</td>
<td>HTTPRoute</td>
<td>VirtualServer routing</td>
</tr>

<tr>
<td><b>Layer 7 Routing</b></td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>✅ Yes</td>
</tr>

<tr>
<td><b>Layer 4 Routing</b></td>
<td>TransportServer CRD</td>
<td>⚠️ Partial</td>
<td>🔄 Roadmap</td>
<td>—</td>
<td>TransportServer CRD</td>
</tr>

<tr>
<td><b>gRPC</b></td>
<td>Native support</td>
<td>⚠️ Requires annotation</td>
<td>GRPCRoute</td>
<td>GRPCRoute</td>
<td>⚠️ HTTP/2 profile</td>
</tr>

<tr>
<td><b>WebSocket</b></td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>—</td>
<td>✅ Yes</td>
</tr>

<tr>
<th colspan="6" align="left">🔷 LOAD BALANCING</th>
</tr>

<tr>
<td><b>Layer 7 Load Balancing</b></td>
<td>NGINX upstream LB</td>
<td>NGINX upstream LB</td>
<td>NGINX upstream LB</td>
<td>TMM high-performance LB</td>
<td>BIG-IP LTM</td>
</tr>

<tr>
<td><b>Round Robin</b></td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>✅ Yes</td>
</tr>

<tr>
<td><b>Least Connections</b></td>
<td>Supported</td>
<td>Supported</td>
<td>Supported</td>
<td>—</td>
<td>Supported</td>
</tr>

<tr>
<td><b>Session Persistence</b></td>
<td>Cookie-based</td>
<td>Cookie annotation</td>
<td>Supported</td>
<td>—</td>
<td>BIG-IP persistence profiles</td>
</tr>

<tr>
<th colspan="6" align="left">🔷 SECURITY</th>
</tr>

<tr>
<td><b>TLS Termination</b></td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>✅ Yes</td>
<td>—</td>
<td>Hardware-accelerated SSL</td>
</tr>

<tr>
<td><b>mTLS</b></td>
<td>Policy CRD</td>
<td>Annotations</td>
<td>BackendTLSPolicy</td>
<td>Supported</td>
<td>Mutual auth profiles</td>
</tr>

<tr>
<td><b>JWT Authentication</b></td>
<td>Supported</td>
<td>⚠️ Limited</td>
<td>🔄 Roadmap</td>
<td>—</td>
<td>BIG-IP APM</td>
</tr>

<tr>
<td><b>WAF</b></td>
<td>NGINX App Protect</td>
<td>⚠️ ModSecurity</td>
<td>F5 WAF for NGINX</td>
<td>—</td>
<td>F5 Advanced WAF</td>
</tr>

<tr>
<th colspan="6" align="left">📖 Legend</th>
</tr>

<tr>
<td><b>✅ Yes</b></td>
<td colspan="5">Feature fully supported</td>
</tr>

<tr>
<td><b>❌ No</b></td>
<td colspan="5">Feature not available</td>
</tr>

<tr>
<td><b>⚠️ Partial</b></td>
<td colspan="5">Feature partially supported or limited</td>
</tr>

<tr>
<td><b>💰 Commercial</b></td>
<td colspan="5">Available only in commercial editions</td>
</tr>

<tr>
<td><b>🔄 Roadmap</b></td>
<td colspan="5">Feature planned but not yet released</td>
</tr>

</tbody>
</table>

</div>
</details>

---

## 📁 Repository Structure

```
f5-k8s-ingress-comparison/
├── README.md                          ← This file
├── assets/
│   └── f5-k8s-ingress-comparison-matrix.xlsx   ← Full 130+ feature matrix
├── docs/
│   ├── product-overview.md            ← Deep-dive on each product
│   ├── decision-guide.md              ← When to use which solution
│   ├── migration-guide.md             ← ingress-nginx EOL migration paths
│   ├── security-comparison.md         ← WAF, mTLS, OIDC, Zero Trust deep-dive
│   └── gateway-api-overview.md        ← Gateway API concepts and adoption
├── config/
│   ├── nic/                           ← NIC sample CRD manifests
│   ├── ngf/                           ← NGF sample Gateway API manifests
│   ├── bnk/                           ← BNK sample CRD manifests
│   └── cis/                           ← CIS sample VirtualServer + AS3 manifests
└── images/                            ← Architecture diagrams
```

---

## 📖 Documentation

| Document | Description |
|---|---|
| [Product Overview](docs/product-overview.md) | Deep-dive on each of the 5 products |
| [Decision Guide](docs/decision-guide.md) | Flowchart and criteria for choosing the right solution |
| [Migration Guide](docs/migration-guide.md) | How to migrate from ingress-nginx (EOL) |
| [Security Comparison](docs/security-comparison.md) | WAF, mTLS, OIDC, Zero Trust, DDoS deep-dive |
| [Gateway API Overview](docs/gateway-api-overview.md) | Gateway API concepts and why it matters |

---

## 🔄 Migration from ingress-nginx (EOL March 2026)

Community `kubernetes/ingress-nginx` reached End of Life on **March 2026**. If you are still running it, you should migrate immediately.

```
ingress-nginx (EOL)
        │
        ├──► F5 NGINX Ingress Controller (NIC)
        │    Best for: Fastest migration, familiar NGINX engine,
        │              existing annotation knowledge transfers.
        │    Tool: ingress2gateway (NGINX provider)
        │
        ├──► F5 NGINX Gateway Fabric (NGF)
        │    Best for: Greenfield, long-term portability,
        │              multi-tenant platform teams.
        │    Tool: ingress2gateway (NGINX provider → NGF target)
        │
        └──► F5 BIG-IP CIS
             Best for: Enterprises with existing BIG-IP;
                       OpenShift environments.
             Tool: Manual migration — CIS uses external BIG-IP model.
```

See the full [Migration Guide](docs/migration-guide.md) for step-by-step instructions.

---

## 🔐 Security Capabilities at a Glance

| Capability | NIC | NGF | BNK | CIS |
|---|---|---|---|---|
| WAF | 💰 NGINX App Protect | 💰 F5 WAF | ✅ F5 BIG-IP Next WAF | ✅ F5 Advanced WAF |
| mTLS | ✅ Policy CRD | ✅ BackendTLSPolicy | ✅ | ✅ C3D |
| JWT | ✅ OSS + Plus | 🔄 Roadmap | ✅ | ✅ APM |
| OIDC / SSO | ✅ v5.3+ | 🔄 Roadmap | ✅ | ✅ APM (MFA, federation) |
| IPS/IDS | ❌ | ❌ | ✅ F5-IPSD | ✅ AFM |
| Stateful Firewall | ❌ | ❌ | ✅ F5BigFwPolicy | ✅ AFM |
| DDoS | ✅ F5 stack | ✅ F5 stack | ✅ Full mitigation | ✅ BIG-IP DoS profiles |
| Zero Trust | ✅ Policy CRD | ✅ Role model | ✅ | ✅ APM identity-aware |

---

## 🤖 AI & Modern Workload Support

BNK introduces capabilities purpose-built for AI factory and LLM inference workloads:

- **f5-analyzer** — Intelligent routing of LLM traffic based on request complexity and backend load
- **MCP Server Proxy** — Reverse proxy for Model Context Protocol (MCP) servers with authentication and data classification
- **NVIDIA BlueField-3 DPU** — 99% CPU reduction vs NGINX for high-throughput AI inference traffic
- **190x energy efficiency** improvement in SoftBank PoC vs software-only ingress

---

## 📡 Telco & 5G Support (BNK)

BNK is the only solution in this comparison with native carrier-grade protocol support:

| Protocol | Support |
|---|---|
| SCTP | ✅ Native |
| GTP (4G/5G User Plane) | ✅ Native |
| Diameter (4G Core) | ✅ Native |
| SIP (VoIP) | ✅ Native |
| NGAP (5G N2 Interface) | ✅ Native |
| HTTP/3 (QUIC) | 🔄 Roadmap |

---

## 🧱 When to Use Which Solution

```
Start here: What is your primary use case?
│
├── I need the fastest migration from ingress-nginx
│   └──► F5 NGINX Ingress Controller (NIC)
│
├── I am building a new platform and want long-term portability
│   └──► F5 NGINX Gateway Fabric (NGF)
│
├── I have existing F5 BIG-IP hardware/VE and want K8s integration
│   └──► F5 BIG-IP Container Ingress Services (CIS)
│
├── I need carrier-grade performance, telco protocols, or AI inference routing
│   └──► F5 BIG-IP Next for Kubernetes (BNK)
│
└── I am still running ingress-nginx
    └──► ⚠️ Plan migration immediately — EOL since March 2026
```

See the full [Decision Guide](docs/decision-guide.md) for detailed scoring criteria.

---

## 📚 References

| Resource | URL |
|---|---|
| NGINX Ingress Controller Docs | https://docs.nginx.com/nginx-ingress-controller |
| NGINX Gateway Fabric Docs | https://docs.nginx.com/nginx-gateway-fabric |
| BIG-IP CIS Docs | https://clouddocs.f5.com/containers/latest |
| BIG-IP Next for Kubernetes | https://clouddocs.f5.com/bigip-next-for-kubernetes/latest |
| Kubernetes Gateway API | https://gateway-api.sigs.k8s.io |
| ingress2gateway Migration Tool | https://github.com/kubernetes-sigs/ingress2gateway |
| NGINX GitHub | https://github.com/nginxinc/kubernetes-ingress |
| NGF GitHub | https://github.com/nginx/nginx-gateway-fabric |
| CIS GitHub | https://github.com/F5Networks/k8s-bigip-ctlr |

---

## 📄 License

This repository is licensed under the [Apache 2.0 License](LICENSE).

---

*Maintained by the F5 community. Not an official F5 publication — for official documentation visit [docs.nginx.com](https://docs.nginx.com) and [clouddocs.f5.com](https://clouddocs.f5.com).*

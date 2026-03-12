# 🌐 F5 Kubernetes Ingress Solutions — Technology Comparison

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
│  ┌──────────┐  ┌─────────────┐  ┌──────────┐              │
│  │   NIC    │  │     NGF     │  │   BNK    │              │
│  │ (NGINX+) │  │ (GW API)   │  │ (TMM+DPU)│              │
│  └──────────┘  └─────────────┘  └──────────┘              │
│       │               │               │                    │
│  ┌────────────────────────────────────────┐               │
│  │         CIS (External BIG-IP ADC)      │               │
│  └────────────────────────────────────────┘               │
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
│  Gateway API + BNK CRDs:               │
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

<table>
<thead>
<tr>
<th>Feature / Category</th>
<th>F5 NGINX Ingress Controller (NIC)</th>
<th>Community Ingress-NGINX ⚠️ EOL Mar 2026</th>
<th>F5 NGINX Gateway Fabric (NGF)</th>
<th>F5 BIG-IP Next for Kubernetes (BNK)</th>
<th>F5 BIG-IP Container Ingress Services (CIS)</th>
</tr>
</thead>
<tbody>
<tr><th colspan="6"><b>🔷 TARGET USE CASE</b></th></tr>
<tr><th colspan="6"><b>🔷  GENERAL OVERVIEW</b></th></tr>
<tr><td><b>Maintained By</b></td><td>F5 / NGINX Inc. (dedicated team)</td><td>Kubernetes Community (EOL Mar 2026)</td><td>F5 / NGINX Inc. (dedicated team)</td><td>F5 (dedicated engineering team)</td><td>F5 (open-source — included with BIG-IP support)</td></tr>
<tr><td><b>GitHub Repository</b></td><td>nginx/kubernetes-ingress</td><td>kubernetes/ingress-nginx</td><td>nginx/nginx-gateway-fabric</td><td>❌ No (F5 Artiifactory)</td><td>F5Networks/k8s-bigip-ctlr (clouddocs.f5.com/containers/latest)</td></tr>
<tr><td><b>License</b></td><td>Apache 2.0 (OSS) + Commercial (Plus)</td><td>Apache 2.0 (OSS)</td><td>Apache 2.0 (OSS) + Commercial (Plus)</td><td>💰 Commercial — F5 / MyF5 entitlement required</td><td>Apache 2.0 (OSS) — support via BIG-IP entitlement</td></tr>
<tr><td><b>Kubernetes API Used</b></td><td>Ingress API + CRDs (VirtualServer, etc.)</td><td>Ingress API + Annotations</td><td>Gateway API (GatewayClass, Gateway, Routes)</td><td>Gateway API (GatewayClass, Gateway, Routes,Virtual Servers, etc)</td><td>Ingress API + OpenShift Routes + F5 CRDs (VirtualServer, TransportServer, Policy, ExternalDNS) + AS3 ConfigMap</td></tr>
<tr><td><b>Data Plane</b></td><td>NGINX Open Source or NGINX Plus</td><td>NGINX Open Source (Lua extensions)</td><td>NGINX Open Source or NGINX Plus</td><td>F5 TMM (Traffic Management Microkernel) — optionally DPU-accelerated (NVIDIA BlueField-3)</td><td>F5 BIG-IP (hardware or VE) — external to the cluster; TMM data plane</td></tr>
<tr><td><b>Maturity / Status</b></td><td>✅ Actively developed — production-grade</td><td>⚠️ Feature frozen — EOL March 2026</td><td>✅ Actively developed — future-forward</td><td>✅ Actively developed — future-forward</td><td>✅ Actively developed — v2.x GA — enterprise production-grade</td></tr>
<tr><td><b>Market Share / Adoption</b></td><td>~40% of Kubernetes Ingress deployments</td><td>Previously ~41% — declining post-EOL</td><td>Growing — CNCF-conformant GA release</td><td>telco/service provider &amp; AI infrastructure; growing enterprise</td><td>Established — prevalent in enterprises with existing BIG-IP investments</td></tr>
<tr><td><b>Commercial Tier Available</b></td><td>✅ Yes — NGINX Plus integration</td><td>❌ No</td><td>✅ Yes — NGINX Plus / NGINX One</td><td>✅ Yes — SPK, CNF and BNK</td><td>💰 Included with BIG-IP license — CIS itself is open source</td></tr>
<tr><td><b>CNCF Conformance</b></td><td>N/A (Ingress API)</td><td>N/A (Ingress API)</td><td>✅ Yes — Gateway API conformant</td><td>✅ Yes — Gateway API v1.2.0 conformant</td><td>⚠️ N/A — control-plane connector; data plane is BIG-IP (not Gateway API)</td></tr>
<tr><th colspan="6"><b>🔷  CONFIGURATION MODEL</b></th></tr>
<tr><td><b>Configuration Approach</b></td><td>CRDs (VirtualServer, Policy, TransportServer) + standard Ingress resources</td><td>Kubernetes Ingress resources + annotations + ConfigMap</td><td>Gateway API resources (GatewayClass, Gateway, HTTPRoute, GRPCRoute, etc.)</td><td>Gateway API CRDs + F5 extension CRDs (CNEInstance, BNKSecPolicy, F5BigFwPolicy, F5SPKEgress, iRules)</td><td>CRDs (VirtualServer, TransportServer, Policy, ExternalDNS) + AS3 ConfigMap + Kubernetes Ingress + OpenShift Routes</td></tr>
<tr><td><b>Custom Resource Definitions (CRDs)</b></td><td>✅ Yes — VirtualServer, VirtualServerRoute, TransportServer, Policy, GlobalConfiguration</td><td>⚠️ Limited — custom ConfigMap keys only</td><td>✅ Yes — Gateway API CRDs (standardized)</td><td>✅ Yes —BIGIP Analyzers, CNE addresslists, iRules, PortLists, DDOS globals, FW pol, RuleLists, Profiles, BNK Gateways, BNK Egress, SnatPools, Static Routes, Vlans, BIGIP TMM, etc</td><td>✅ Yes — VirtualServer, TransportServer, IngressLink, Policy, ExternalDNS, TLSProfile</td></tr>
<tr><td><b>Annotation-based Config</b></td><td>✅ Supported (plus CRDs for advanced use)</td><td>✅ Primary configuration method</td><td>❌ No — annotation-less by design</td><td>❌ No — CRD/CR-driven exclusively</td><td>✅ Yes — Ingress annotations + OpenShift Route annotations</td></tr>
<tr><td><b>Schema Validation</b></td><td>✅ Yes — CRD schema-backed validation</td><td>⚠️ Limited — annotation strings, no schema</td><td>✅ Yes — Gateway API schema validation</td><td>✅ Yes — Gateway API schema validation</td><td>✅ Yes — CRD schema + AS3 JSON schema validation</td></tr>
<tr><td><b>Role-Based Configuration Model</b></td><td>⚠️ Single role (cluster admin focus)</td><td>⚠️ Single role (cluster admin focus)</td><td>✅ Yes — 3 roles: Infra Provider, Cluster Operator, App Developer</td><td>✅ Yes — 3 roles: Infra Provider, Cluster Operator, App Developer (Gateway API model)</td><td>⚠️ Limited — single admin model; NetOps drives BIG-IP config via K8s manifests</td></tr>
<tr><td><b>Multi-Tenancy Support</b></td><td>⚠️ Limited — namespace-level separation</td><td>⚠️ Limited</td><td>✅ Native — role-oriented multi-tenant model</td><td>??</td><td>✅ Yes — BIG-IP partitions per tenant; multi-cluster support</td></tr>
<tr><td><b>GitOps Compatible</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — declarative YAML/CRDs + AS3 declarations</td></tr>
<tr><td><b>Helm Chart Deployment</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — Helm charts + OLM (Operator Lifecycle Manager) on OpenShift</td></tr>
<tr><th colspan="6"><b>🔷  TRAFFIC ROUTING</b></th></tr>
<tr><td><b>HTTP/HTTPS Routing</b></td><td>✅ Yes — host &amp; path-based</td><td>✅ Yes — host &amp; path-based</td><td>✅ Yes — HTTPRoute (host, path, header, query)</td><td>✅ Yes — HTTPRoute (host, path, header, query, irules)</td><td>✅ Yes — host &amp; path routing via VirtualServer CRD and Ingress</td></tr>
<tr><td><b>Layer 7 Routing</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — full BIG-IP LTM L7 capabilities</td></tr>
<tr><td><b>Layer 4 (TCP/UDP) Routing</b></td><td>✅ Yes — via TransportServer CRD</td><td>⚠️ Partial — via ConfigMap annotation</td><td>🔄 Roadmap — L4 support planned (TCPRoute/UDPRoute)</td><td>—</td><td>✅ Yes — TransportServer CRD (TCP, UDP)</td></tr>
<tr><td><b>gRPC Routing</b></td><td>✅ Yes — native gRPC support</td><td>⚠️ Partial — requires TLS + HTTP/2 annotation</td><td>✅ Yes — native GRPCRoute</td><td>✅ Yes — GRPCRoute</td><td>⚠️ Partial — via BIG-IP HTTP/2 profile; no native gRPC CRD</td></tr>
<tr><td><b>WebSocket Support</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>—</td><td>✅ Yes</td></tr>
<tr><td><b>Header-Based Routing</b></td><td>✅ Yes</td><td>⚠️ Partial — via annotations</td><td>✅ Yes — native in HTTPRoute spec</td><td>✅ Yes — HTTPRoute native</td><td>✅ Yes — via iRules + LTM policies on BIG-IP</td></tr>
<tr><td><b>Path-Based Routing</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — VirtualServer CRD</td></tr>
<tr><td><b>Query Parameter Routing</b></td><td>✅ Yes</td><td>⚠️ Partial</td><td>✅ Yes — native in HTTPRoute spec</td><td>✅ Yes</td><td>⚠️ Partial — via iRules</td></tr>
<tr><td><b>Cookie-Based Routing</b></td><td>✅ Yes</td><td>✅ Yes — canary via cookie annotation</td><td>✅ Yes</td><td>—</td><td>✅ Yes — BIG-IP persistence profiles</td></tr>
<tr><td><b>Method-Based Routing</b></td><td>✅ Yes</td><td>⚠️ Partial</td><td>✅ Yes — native</td><td>✅ Yes</td><td>⚠️ Partial — via iRules / LTM policy</td></tr>
<tr><td><b>Cross-Namespace Routing</b></td><td>✅ Yes (v5.3+ — upstream from different NS)</td><td>⚠️ Limited</td><td>✅ Yes — native via ReferenceGrant</td><td>—</td><td>✅ Yes — CIS watches multiple namespaces; multi-cluster service discovery</td></tr>
<tr><td><b>TLS Passthrough</b></td><td>✅ Yes — via TransportServer</td><td>⚠️ Partial</td><td>✅ Yes — TLSRoute passthrough</td><td>✅ Yes — TLSRoute</td><td>✅ Yes — TLSProfile passthrough + BIG-IP SSL passthrough</td></tr>
<tr><th colspan="6"><b>🔷  LOAD BALANCING</b></th></tr>
<tr><td><b>Layer 7 Load Balancing</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — high-performance via TMM</td><td>✅ Yes — full BIG-IP LTM</td></tr>
<tr><td><b>Layer 4 Load Balancing</b></td><td>✅ Yes — TCP/UDP via TransportServer</td><td>⚠️ Partial</td><td>🔄 Roadmap</td><td>—</td><td>✅ Yes — TransportServer + BIG-IP LTM TCP/UDP</td></tr>
<tr><td><b>Round Robin</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td></tr>
<tr><td><b>Least Connections</b></td><td>✅ Yes (Plus: full upstream config)</td><td>✅ Yes</td><td>✅ Yes</td><td>—</td><td>✅ Yes — BIG-IP LTM load balancing methods</td></tr>
<tr><td><b>IP Hash / Sticky Sessions</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — BIG-IP persistence profiles (cookie, source IP, SSL)</td></tr>
<tr><td><b>Session Persistence</b></td><td>✅ Yes (cookie-based + Plus: full)</td><td>✅ Yes — cookie annotation</td><td>✅ Yes</td><td>—</td><td>✅ Yes — full BIG-IP persistence (cookie, source IP, custom)</td></tr>
<tr><td><b>Active Health Checks</b></td><td>💰 NGINX Plus only</td><td>⚠️ Passive only (OSS)</td><td>💰 NGINX Plus only</td><td>✅ Yes — TMM-native active health checks</td><td>✅ Yes — BIG-IP native active health monitors (HTTP, TCP, ICMP, custom)</td></tr>
<tr><td><b>Passive Health Checks</b></td><td>✅ Yes (OSS)</td><td>✅ Yes</td><td>✅ Yes (OSS)</td><td>—</td><td>✅ Yes</td></tr>
<tr><td><b>Dynamic Reconfiguration</b></td><td>✅ Yes — no NGINX reload required (Plus)</td><td>⚠️ Requires NGINX reload</td><td>✅ Yes (Plus)</td><td>✅ Yes — TMM hot reconfiguration, no reload</td><td>✅ Yes — AS3 declarative push; no BIG-IP reload required</td></tr>
<tr><td><b>Upstream Zone / State Sharing</b></td><td>💰 NGINX Plus — key-value store, zones</td><td>❌ No</td><td>💰 NGINX Plus</td><td>—</td><td>✅ Yes — BIG-IP HA config-sync; shared state across BIG-IP pairs</td></tr>
<tr><th colspan="6"><b>🔷  TRAFFIC MANAGEMENT &amp; DEPLOYMENT PATTERNS</b></th></tr>
<tr><td><b>Traffic Splitting / Weighting</b></td><td>✅ Yes — VirtualServer spec</td><td>✅ Yes — canary annotation</td><td>✅ Yes — native HTTPRoute weight</td><td>—</td><td>✅ Yes — VirtualServer CRD pools with weights</td></tr>
<tr><td><b>Canary Deployments</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — pool weighting</td></tr>
<tr><td><b>Blue-Green Deployments</b></td><td>✅ Yes</td><td>⚠️ Partial — via annotations</td><td>✅ Yes — native</td><td>—</td><td>✅ Yes — OpenShift Routes A/B + CIS CRD pools</td></tr>
<tr><td><b>A/B Testing</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td></tr>
<tr><td><b>Rate Limiting</b></td><td>✅ Yes — Policy CRD</td><td>✅ Yes — annotation</td><td>✅ Yes — RateLimit policy</td><td>—</td><td>✅ Yes — Policy CRD + BIG-IP rate shaping</td></tr>
<tr><td><b>Circuit Breaking</b></td><td>✅ Yes</td><td>⚠️ Partial</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — BIG-IP connection limits + health monitors</td></tr>
<tr><td><b>Request Buffering</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>—</td><td>✅ Yes — BIG-IP HTTP profiles</td></tr>
<tr><td><b>Request/Response Header Manipulation</b></td><td>✅ Yes</td><td>✅ Yes — annotations</td><td>✅ Yes — native HTTPRoute filters</td><td>✅ Yes — HTTPRoute filters</td><td>✅ Yes — iRules + BIG-IP HTTP profile header insertion</td></tr>
<tr><td><b>URL Rewriting / Redirects</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — native HTTPRoute filters</td><td>—</td><td>✅ Yes — VirtualServer rewrite + iRules</td></tr>
<tr><td><b>Request Mirroring</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — native</td><td>✅ Yes</td><td>✅ Yes — BIG-IP mirroring</td></tr>
<tr><td><b>Direct Response Injection</b></td><td>✅ Yes</td><td>⚠️ Partial</td><td>✅ Yes — native</td><td>—</td><td>✅ Yes — iRules</td></tr>
<tr><td><b>Caching</b></td><td>✅ Yes (v5.3+ — configurable cache policy)</td><td>⚠️ Limited</td><td>🔄 Roadmap</td><td>🔄 Roadmap</td><td>✅ Yes — BIG-IP RAM Cache (Web Acceleration profile)</td></tr>
<tr><td><b>Connection Limiting</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>—</td><td>✅ Yes — BIG-IP virtual server connection limits</td></tr>
<tr><th colspan="6"><b>🔷  SECURITY</b></th></tr>
<tr><td><b>TLS/SSL Termination</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>—</td><td>✅ Yes — hardware-accelerated SSL on BIG-IP</td></tr>
<tr><td><b>mTLS (Mutual TLS)</b></td><td>✅ Yes — Policy CRD (client &amp; backend)</td><td>✅ Yes — annotations</td><td>✅ Yes — BackendTLSPolicy</td><td>✅ Yes</td><td>✅ Yes — C3D (Client Certificate Constrained Delegation) + mutual auth profiles</td></tr>
<tr><td><b>JWT Authentication</b></td><td>✅ Yes (OSS + Plus)</td><td>⚠️ Limited — third-party auth only</td><td>🔄 Roadmap</td><td>—</td><td>✅ Yes — BIG-IP APM + iRules</td></tr>
<tr><td><b>OIDC Authentication</b></td><td>✅ Yes — OIDC + v5.3+ enhancements</td><td>⚠️ Via external auth-url annotation</td><td>🔄 Roadmap</td><td>✅ Yes</td><td>✅ Yes — BIG-IP APM OIDC/OAuth federation, SSO, MFA</td></tr>
<tr><td><b>Basic Authentication</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>—</td><td>✅ Yes</td></tr>
<tr><td><b>RBAC</b></td><td>✅ Yes — Kubernetes native RBAC</td><td>✅ Yes — Kubernetes native RBAC</td><td>✅ Yes — enhanced, role-based model</td><td>✅ Yes — Kubernetes native + Gateway API role model</td><td>✅ Yes — Kubernetes RBAC + BIG-IP partition-level RBAC</td></tr>
<tr><td><b>WAF (Web Application Firewall)</b></td><td>💰 NGINX App Protect (Plus) — OWASP Top 10, gRPC schema, OpenAPI</td><td>⚠️ ModSecurity (limited)</td><td>💰 F5 WAF for NGINX (Plus/NGINX One)</td><td>—</td><td>✅ Yes — F5 Advanced WAF (OWASP Top 10, bot defense, API security, L3/L7 firewall via Policy CRD)</td></tr>
<tr><td><b>DDoS Protection</b></td><td>✅ Yes — combined with F5 security stack</td><td>⚠️ Basic</td><td>✅ Yes — combined with F5 security stack</td><td>✅ Yes — full DDoS mitigation, edge firewall</td><td>✅ Yes — F5 Advanced WAF + BIG-IP DoS protection profiles</td></tr>
<tr><td><b>IP Allowlist/Denylist</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>—</td><td>✅ Yes — Policy CRD + BIG-IP iRules + network firewall</td></tr>
<tr><td><b>CORS Policy</b></td><td>✅ Yes</td><td>✅ Yes — annotation</td><td>✅ Yes — native HTTPRoute filter</td><td>✅ Yes</td><td>✅ Yes — iRules + BIG-IP HTTP profiles</td></tr>
<tr><td><b>Secret Management (TLS)</b></td><td>✅ Yes — Kubernetes Secrets</td><td>✅ Yes — Kubernetes Secrets</td><td>✅ Yes — Kubernetes Secrets</td><td>—</td><td>✅ Yes — Kubernetes Secrets; BIG-IP certificate management</td></tr>
<tr><td><b>End-to-End Encryption</b></td><td>✅ Yes — TLS passthrough &amp; termination</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — hardware-offloaded encryption on DPU</td><td>✅ Yes — SSL bridging, SSL passthrough, re-encryption</td></tr>
<tr><td><b>Zero Trust / Shift Left Security</b></td><td>✅ Yes — Policy CRD enforcement</td><td>⚠️ Limited</td><td>✅ Yes — role-based policy model</td><td>✅ Yes — role-based policy model</td><td>✅ Yes — BIG-IP APM zero-trust; identity-aware ingress</td></tr>
<tr><td><b>OpenID Connect Enhancements</b></td><td>✅ Yes (v5.3+ token timeouts, front-channel logout)</td><td>⚠️ Basic</td><td>🔄 Roadmap</td><td>✅ Yes</td><td>✅ Yes — BIG-IP APM full OIDC (federation, MFA, token validation)</td></tr>
<tr><th colspan="6"><b>🔷  OBSERVABILITY &amp; MONITORING</b></th></tr>
<tr><td><b>Prometheus Metrics</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — CIS exports AS3 HTTP client metrics; BIG-IP Telemetry Streaming</td></tr>
<tr><td><b>Grafana Integration</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — via BIG-IP Telemetry Streaming + Prometheus</td></tr>
<tr><td><b>OpenTelemetry / Tracing</b></td><td>✅ Yes — OTel, Jaeger, Zipkin</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>⚠️ Partial — via BIG-IP Analytics; not native OTel</td></tr>
<tr><td><b>Access Logging</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — BIG-IP request logging profiles; High-Speed Logging (HSL)</td></tr>
<tr><td><b>Structured JSON Logging</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — BIG-IP Telemetry Streaming JSON output</td></tr>
<tr><td><b>Real-Time Metrics Dashboard</b></td><td>💰 NGINX Plus — NGINX One Console</td><td>❌ Not built-in ✅ Yes through Grafana and Prometheus integrations</td><td>💰 NGINX Plus — NGINX One Console</td><td>❌ Not built-in ✅ Yes through Grafana and Prometheus integrations</td><td>✅ Yes — BIG-IP Analytics + Telemetry Streaming dashboard</td></tr>
<tr><td><b>Health Check Endpoint</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — CIS /ready endpoint + BIG-IP health monitors</td></tr>
<tr><td><b>NGINX One Console</b></td><td>💰 Available with NGINX One subscription</td><td>❌ No</td><td>💰 Available with NGINX One subscription</td><td>❌ No</td><td>❌ No — uses BIG-IP management (TMUI, TMOS CLI, F5 ADSP)</td></tr>
<tr><th colspan="6"><b>🔷  EXTENSIBILITY</b></th></tr>
<tr><td><b>Custom Extensions Model</b></td><td>✅ CRDs + Policy resources + NGINX snippets</td><td>⚠️ Annotations + Lua modules (fragile)</td><td>✅ Policy Attachments — annotation-less model</td><td>—</td><td>✅ Yes — iRules (TCL) + AS3 ConfigMap + Policy CRD + BIG-IP profiles</td></tr>
<tr><td><b>NGINX Configuration Snippets</b></td><td>✅ Yes</td><td>✅ Yes — nginx.ingress.kubernetes.io/configuration-snippet</td><td>❌ No — by design (cleaner model)</td><td>❌ No — not NGINX-based</td><td>❌ No — not NGINX-based</td></tr>
<tr><td><b>Lua Scripting</b></td><td>❌ No (uses native NGINX modules)</td><td>✅ Yes — Lua-based extensions</td><td>❌ No</td><td>—</td><td>❌ No — uses iRules (TCL-based programmable data plane)</td></tr>
<tr><td><b>Custom Policies / Filters</b></td><td>✅ Yes — Policy CRD</td><td>⚠️ Via annotations only</td><td>✅ Yes — Policy attachments (extensionRef)</td><td>✅ Yes — BNKSecPolicy, BNKNetPolicy, F5BigFwPolicy</td><td>✅ Yes — Policy CRD (WAF, firewall, persistence, iRules, profiles)</td></tr>
<tr><td><b>Plugin Ecosystem</b></td><td>⚠️ Limited — NGINX modules</td><td>⚠️ Limited — Lua plugins</td><td>✅ Growing — Gateway API extension points</td><td>—</td><td>✅ Yes — full BIG-IP module ecosystem (APM, ASM/WAF, DNS, AFM, GTM)</td></tr>
<tr><th colspan="6"><b>🔷  PLATFORM &amp; INTEGRATIONS</b></th></tr>
<tr><td><b>Kubernetes Versions</b></td><td>✅ All modern versions</td><td>✅ All modern versions (until EOL)</td><td>✅ All modern versions</td><td>—</td><td>✅ All modern versions — see CIS compatibility matrix</td></tr>
<tr><td><b>Red Hat OpenShift</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — certified</td><td>✅ Yes — Red Hat certified; NextGen Routes + OLM support</td></tr>
<tr><td><b>Service Mesh Integration (Istio/Linkerd)</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — north-south + east-west</td><td>—</td><td>⚠️ Partial — north-south LB; Istio ambient mesh (HBONE) not natively supported. Aspen Mesh</td></tr>
<tr><td><b>F5 BIG-IP / IngressLink</b></td><td>✅ Yes — IngressLink CRD integration</td><td>⚠️ Via CIS only</td><td>🔄 Roadmap</td><td>✅ Yes — same TMM engine; part of F5 ADSP ecosystem</td><td>✅ Yes — CIS IS the BIG-IP connector; IngressLink for NIC+BIG-IP 2-tier</td></tr>
<tr><td><b>F5 Distributed Cloud</b></td><td>✅ Yes</td><td>❌ No</td><td>🔄 Roadmap</td><td>❌ No</td><td>✅ Yes — BIG-IP Distributed Cloud integration</td></tr>
<tr><td><b>Cert-Manager</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td></tr>
<tr><td><b>ExternalDNS</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — ExternalDNS CRD + F5 IPAM Controller (FIC) for IP management</td></tr>
<tr><td><b>AWS / Azure / GCP</b></td><td>✅ Yes — cloud-agnostic</td><td>✅ Yes</td><td>✅ Yes — hybrid/multicloud by design</td><td>✅ Yes — private cloud, colocated DC, and hybrid</td><td>✅ Yes — BIG-IP VE on public cloud; hybrid multi-cloud deployments</td></tr>
<tr><td><b>ingress2gateway Migration Tool</b></td><td>✅ Yes — NGINX provider included</td><td>✅ Yes — source</td><td>✅ Yes — target destination</td><td>❌ No</td><td>❌ No — CIS uses Ingress/CRD model, not Gateway API; no migration tool</td></tr>
<tr><th colspan="6"><b>🔷  OPERATIONAL &amp; ENTERPRISE</b></th></tr>
<tr><td><b>High Availability</b></td><td>✅ Yes — multi-replica deployment</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — BIG-IP HA pairs (Active/Standby); Primary/Secondary CIS instances</td></tr>
<tr><td><b>Horizontal Scaling</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — scales from core to far-edge</td><td>⚠️ Partial — CIS scales; BIG-IP scales via VE licensing or hardware</td></tr>
<tr><td><b>State Sharing (rate limits, etc.)</b></td><td>💰 NGINX Plus — key-value + zone sharing</td><td>❌ No</td><td>💰 NGINX Plus</td><td>✅ Yes</td><td>✅ Yes — BIG-IP HA config-sync; connection mirroring</td></tr>
<tr><td><b>Commercial Support (SLA)</b></td><td>💰 NGINX Plus / NGINX One</td><td>❌ No</td><td>💰 NGINX Plus / NGINX One</td><td>💰 Yes — F5 enterprise support with SLA</td><td>💰 Yes — included with F5 BIG-IP support entitlement</td></tr>
<tr><td><b>Security Patch Cadence</b></td><td>✅ Active — dedicated F5 team</td><td>⚠️ Critical only until EOL — then none</td><td>✅ Active — dedicated F5 team</td><td>✅ Active — dedicated F5 team</td><td>✅ Active — F5 security advisories + quarterly CIS releases</td></tr>
<tr><td><b>Upgrade Path from ingress-nginx</b></td><td>✅ Best-supported — migration guide + tool</td><td>N/A</td><td>✅ Via ingress2gateway tool</td><td>⚠️ Significant migration — different architecture &amp; CRDs</td><td>⚠️ Significant migration — different architecture (external BIG-IP)</td></tr>
<tr><td><b>Vendor Lock-in</b></td><td>⚠️ Moderate — NGINX-specific CRDs</td><td>⚠️ Moderate — annotation-based</td><td>✅ Low — standardized Gateway API</td><td>✅ Yes</td><td>⚠️ High — requires F5 BIG-IP hardware/VE; iRules are proprietary</td></tr>
<tr><td><b>Configuration Portability</b></td><td>⚠️ NGINX-specific</td><td>⚠️ NGINX-specific</td><td>✅ High — portable across Gateway API impls</td><td>⚠️ Limited — Gateway API routes portable; F5 CRDs are proprietary</td><td>⚠️ Low — F5-specific CRDs + iRules; AS3 JSON is F5-only</td></tr>
<tr><td><b>Long-Term Roadmap</b></td><td>✅ Active — Ingress API + Gateway API path</td><td>❌ EOL — no roadmap</td><td>✅ Future-proof — Gateway API is K8s standard</td><td>—</td><td>✅ Active — multi-cluster, NextGen Routes, Gateway API integration planned</td></tr>
<tr><th colspan="6"><b>🔷  PROTOCOL SUPPORT</b></th></tr>
<tr><td><b>HTTP/1.1</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td></tr>
<tr><td><b>HTTP/2</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — BIG-IP HTTP/2 full proxy</td></tr>
<tr><td><b>HTTP/3 (QUIC)</b></td><td>🧪 Experimental</td><td>🧪 Experimental</td><td>🔄 Roadmap</td><td>🔄 Roadmap</td><td>🔄 Roadmap — BIG-IP QUIC support in progress</td></tr>
<tr><td><b>HTTPS</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td></tr>
<tr><td><b>TCP</b></td><td>✅ Yes — TransportServer</td><td>⚠️ Via ConfigMap</td><td>🔄 Roadmap — TCPRoute</td><td>—</td><td>✅ Yes — TransportServer + BIG-IP LTM</td></tr>
<tr><td><b>UDP</b></td><td>✅ Yes — TransportServer</td><td>⚠️ Via ConfigMap</td><td>🔄 Roadmap — UDPRoute</td><td>✅ Yes — L4Route (UDP)</td><td>✅ Yes — TransportServer + BIG-IP LTM</td></tr>
<tr><td><b>gRPC</b></td><td>✅ Yes — native</td><td>⚠️ Partial — requires annotations</td><td>✅ Yes — GRPCRoute</td><td>—</td><td>⚠️ Partial — via HTTP/2 profile; no native gRPC CRD</td></tr>
<tr><td><b>WebSocket</b></td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes</td><td>✅ Yes — BIG-IP WebSocket profile</td></tr>
<tr><td><b>TLS Passthrough</b></td><td>✅ Yes</td><td>⚠️ Via annotation</td><td>✅ Yes — TLSRoute</td><td>—</td><td>✅ Yes — TLSProfile passthrough + BIG-IP SSL passthrough</td></tr>
<tr><th colspan="6"><b>🔷  BNK-EXCLUSIVE CAPABILITIES (not available in NGINX-based solutions)</b></th></tr>
<tr><td><b>Telco Protocols (SCTP, Diameter, SIP, GTP, NGAP)</b></td><td>❌ No</td><td>❌ No</td><td>❌ No</td><td>✅ Yes — native TMM support for 4G/5G telco protocols</td><td>⚠️ Partial — BIG-IP supports SCTP; limited 5G native support vs BNK</td></tr>
<tr><td><b>NVIDIA BlueField DPU Acceleration</b></td><td>❌ No</td><td>❌ No</td><td>❌ No</td><td>✅ Yes — hardware offload: crypto, network processing, security</td><td>❌ No — BIG-IP uses FPGA/ASIC hardware offload (not BlueField DPU)</td></tr>
<tr><td><b>Egress Traffic Management (SNAT pools)</b></td><td>⚠️ Limited</td><td>⚠️ Limited</td><td>⚠️ Limited</td><td>✅ Yes — F5SPKEgress + F5SPKSnatpool CRDs</td><td>✅ Yes — BIG-IP SNAT pools natively; exposed via CIS CRDs</td></tr>
<tr><td><b>Intrusion Prevention System (IPS)</b></td><td>❌ No</td><td>❌ No</td><td>❌ No</td><td>✅ Yes — F5-IPSD (SNORT-compatible deep packet inspection)</td><td>✅ Yes — BIG-IP AFM (Advanced Firewall Manager) IPS/IDS</td></tr>
<tr><td><b>Topology Hiding (internal cluster obscuring)</b></td><td>❌ No</td><td>❌ No</td><td>❌ No</td><td>✅ Yes — ingress topology hiding</td><td>✅ Yes — BIG-IP SNAT + topology hiding via VIP abstraction</td></tr>
<tr><td><b>Per-Subscriber Traffic Visibility / Billing</b></td><td>❌ No</td><td>❌ No</td><td>❌ No</td><td>✅ Yes — compliance tracking and billing telemetry</td><td>⚠️ Partial — BIG-IP Analytics; not Kubernetes-native subscriber model</td></tr>
<tr><td><b>AI / LLM Traffic Routing (f5-analyzer)</b></td><td>❌ No</td><td>❌ No</td><td>❌ No</td><td>✅ Yes — f5-analyzer routes LLM traffic based on complexity/load</td><td>❌ No — not available in CIS</td></tr>
<tr><td><b>MCP Server Proxy (Agentic AI)</b></td><td>❌ No</td><td>❌ No</td><td>❌ No</td><td>✅ Yes — reverse proxy for MCP servers with auth &amp; data classification</td><td>❌ No — not available in CIS</td></tr>
<tr><td><b>Firewall Policy (Stateful ACL)</b></td><td>❌ No</td><td>❌ No</td><td>❌ No</td><td>✅ Yes — F5BigFwPolicy — granular stateful ACL</td><td>✅ Yes — BIG-IP AFM stateful firewall + Policy CRD L3/L7 firewall</td></tr>
<tr><td><b>iRules (Programmable Data Plane)</b></td><td>❌ No</td><td>❌ No</td><td>❌ No</td><td>✅ Yes — F5 iRules (TCL-based programmable data path)</td><td>✅ Yes — full iRules support via BIG-IP; referenced in Policy/VS CRDs</td></tr>
<tr><td><b>CPU Offload / Energy Efficiency</b></td><td>❌ No</td><td>❌ No</td><td>❌ No</td><td>✅ Yes — 99% lower CPU utilization vs NGINX; 190x energy efficiency (BlueField)</td><td>✅ Yes — BIG-IP FPGA/ASIC hardware offload (SSL, compression, etc.)</td></tr>
<tr><th colspan="6"><b>📖 Legend</b></th></tr>
<tr><td><b>✅ Yes</b></td><td colspan="5">Feature fully supported</td></tr>
<tr><td><b>❌ No</b></td><td colspan="5">Feature not available</td></tr>
<tr><td><b>⚠️ Partial / Limited</b></td><td colspan="5">Feature partially supported or has caveats</td></tr>
<tr><td><b>💰 Commercial (Plus)</b></td><td colspan="5">Available in NGINX Plus or NGINX One commercial tier only</td></tr>
<tr><td><b>🔄 Roadmap</b></td><td colspan="5">Feature planned but not yet available</td></tr>
<tr><td><b>🧪 Experimental</b></td><td colspan="5">Feature available but not production-ready</td></tr>
<tr><td><b>—  N/A</b></td><td colspan="5">Not applicable</td></tr>
</tbody>
</table>

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

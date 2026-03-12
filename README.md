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
<summary><strong>📊 Click to expand — Full Feature Comparison Matrix (130+ features)</strong></summary>

<br>

<div style="overflow-x:auto;">
<table style="border-collapse:collapse;width:100%;font-family:Arial,sans-serif;font-size:13px;">
  <thead>
    <tr>
      <th style="background:#1f3864;color:#fff;padding:10px 12px;text-align:left;min-width:220px;border:1px solid #bdbdbd;">Feature / Category</th>
      <th style="background:#2e75b6;color:#fff;padding:10px 12px;text-align:left;min-width:180px;border:1px solid #bdbdbd;">F5 NGINX Ingress Controller<br><small>(NIC)</small></th>
      <th style="background:#555;color:#fff;padding:10px 12px;text-align:left;min-width:180px;border:1px solid #bdbdbd;">Community Ingress-NGINX<br><small>⚠️ EOL Mar 2026</small></th>
      <th style="background:#1a6b3c;color:#fff;padding:10px 12px;text-align:left;min-width:180px;border:1px solid #bdbdbd;">F5 NGINX Gateway Fabric<br><small>(NGF)</small></th>
      <th style="background:#7b2c2c;color:#fff;padding:10px 12px;text-align:left;min-width:180px;border:1px solid #bdbdbd;">F5 BIG-IP Next for K8s<br><small>(BNK)</small></th>
      <th style="background:#1a5276;color:#fff;padding:10px 12px;text-align:left;min-width:180px;border:1px solid #bdbdbd;">F5 BIG-IP CIS<br><small>(External ADC)</small></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td colspan="6" style="background:#1f3864;color:#fff;font-weight:bold;padding:8px 12px;border:1px solid #bdbdbd;">🔷 TARGET USE CASE</td>
    </tr>
    <tr>
      <td colspan="6" style="background:#1f3864;color:#fff;font-weight:bold;padding:8px 12px;border:1px solid #bdbdbd;">🔷  GENERAL OVERVIEW</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Maintained By</td>

      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">F5 / NGINX Inc. (dedicated team)</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">Kubernetes Community (EOL Mar 2026)</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">F5 / NGINX Inc. (dedicated team)</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">F5 (dedicated engineering team)</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">F5 (open-source — included with BIG-IP support)</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">GitHub Repository</td>

      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">nginx/kubernetes-ingress</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">kubernetes/ingress-nginx</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">nginx/nginx-gateway-fabric</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No (F5 Artiifactory)</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">F5Networks/k8s-bigip-ctlr (clouddocs.f5.com/containers/latest)</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">License</td>

      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">Apache 2.0 (OSS) + Commercial (Plus)</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">Apache 2.0 (OSS)</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">Apache 2.0 (OSS) + Commercial (Plus)</td>
      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 Commercial — F5 / MyF5 entitlement required</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">Apache 2.0 (OSS) — support via BIG-IP entitlement</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Kubernetes API Used</td>

      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">Ingress API + CRDs (VirtualServer, etc.)</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">Ingress API + Annotations</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">Gateway API (GatewayClass, Gateway, Routes)</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">Gateway API (GatewayClass, Gateway, Routes,Virtual Servers, etc)</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">Ingress API + OpenShift Routes + F5 CRDs (VirtualServer, TransportServer, Policy, ExternalDNS) + AS3 ConfigMap</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Data Plane</td>

      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">NGINX Open Source or NGINX Plus</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">NGINX Open Source (Lua extensions)</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">NGINX Open Source or NGINX Plus</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">F5 TMM (Traffic Management Microkernel) — optionally DPU-accelerated (NVIDIA BlueField-3)</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">F5 BIG-IP (hardware or VE) — external to the cluster; TMM data plane</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Maturity / Status</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Actively developed — production-grade</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Feature frozen — EOL March 2026</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Actively developed — future-forward</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Actively developed — future-forward</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Actively developed — v2.x GA — enterprise production-grade</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Market Share / Adoption</td>

      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">~40% of Kubernetes Ingress deployments</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">Previously ~41% — declining post-EOL</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">Growing — CNCF-conformant GA release</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">telco/service provider &amp; AI infrastructure; growing enterprise</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">Established — prevalent in enterprises with existing BIG-IP investments</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Commercial Tier Available</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — NGINX Plus integration</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — NGINX Plus / NGINX One</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — SPK, CNF and BNK</td>
      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 Included with BIG-IP license — CIS itself is open source</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">CNCF Conformance</td>

      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">N/A (Ingress API)</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">N/A (Ingress API)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Gateway API conformant</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Gateway API v1.2.0 conformant</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ N/A — control-plane connector; data plane is BIG-IP (not Gateway API)</td>
    </tr>
    <tr>
      <td colspan="6" style="background:#1f3864;color:#fff;font-weight:bold;padding:8px 12px;border:1px solid #bdbdbd;">🔷  CONFIGURATION MODEL</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Configuration Approach</td>

      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">CRDs (VirtualServer, Policy, TransportServer) + standard Ingress resources</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">Kubernetes Ingress resources + annotations + ConfigMap</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">Gateway API resources (GatewayClass, Gateway, HTTPRoute, GRPCRoute, etc.)</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">Gateway API CRDs + F5 extension CRDs (CNEInstance, BNKSecPolicy, F5BigFwPolicy, F5SPKEgress, iRules)</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">CRDs (VirtualServer, TransportServer, Policy, ExternalDNS) + AS3 ConfigMap + Kubernetes Ingress + OpenShift Routes</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Custom Resource Definitions (CRDs)</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — VirtualServer, VirtualServerRoute, TransportServer, Policy, GlobalConfiguration</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Limited — custom ConfigMap keys only</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Gateway API CRDs (standardized)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes —BIGIP Analyzers, CNE addresslists, iRules, PortLists, DDOS globals, FW pol, RuleLists, Profiles, BNK Gateways, BNK Egress, SnatPools, Static Routes, Vlans, BIGIP TMM, etc</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — VirtualServer, TransportServer, IngressLink, Policy, ExternalDNS, TLSProfile</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Annotation-based Config</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Supported (plus CRDs for advanced use)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Primary configuration method</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No — annotation-less by design</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No — CRD/CR-driven exclusively</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Ingress annotations + OpenShift Route annotations</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Schema Validation</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — CRD schema-backed validation</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Limited — annotation strings, no schema</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Gateway API schema validation</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Gateway API schema validation</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — CRD schema + AS3 JSON schema validation</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Role-Based Configuration Model</td>

      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Single role (cluster admin focus)</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Single role (cluster admin focus)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — 3 roles: Infra Provider, Cluster Operator, App Developer</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — 3 roles: Infra Provider, Cluster Operator, App Developer (Gateway API model)</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Limited — single admin model; NetOps drives BIG-IP config via K8s manifests</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Multi-Tenancy Support</td>

      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Limited — namespace-level separation</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Limited</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Native — role-oriented multi-tenant model</td>
      <td style="background:#ffffff;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">??</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP partitions per tenant; multi-cluster support</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">GitOps Compatible</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — declarative YAML/CRDs + AS3 declarations</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Helm Chart Deployment</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Helm charts + OLM (Operator Lifecycle Manager) on OpenShift</td>
    </tr>
    <tr>
      <td colspan="6" style="background:#1f3864;color:#fff;font-weight:bold;padding:8px 12px;border:1px solid #bdbdbd;">🔷  TRAFFIC ROUTING</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">HTTP/HTTPS Routing</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — host &amp; path-based</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — host &amp; path-based</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — HTTPRoute (host, path, header, query)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — HTTPRoute (host, path, header, query, irules)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — host &amp; path routing via VirtualServer CRD and Ingress</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Layer 7 Routing</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — full BIG-IP LTM L7 capabilities</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Layer 4 (TCP/UDP) Routing</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — via TransportServer CRD</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial — via ConfigMap annotation</td>
      <td style="background:#e3f2fd;color:#0d47a1;padding:7px 12px;border:1px solid #e0e0e0;">🔄 Roadmap — L4 support planned (TCPRoute/UDPRoute)</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — TransportServer CRD (TCP, UDP)</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">gRPC Routing</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — native gRPC support</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial — requires TLS + HTTP/2 annotation</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — native GRPCRoute</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — GRPCRoute</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial — via BIG-IP HTTP/2 profile; no native gRPC CRD</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">WebSocket Support</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Header-Based Routing</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial — via annotations</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — native in HTTPRoute spec</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — HTTPRoute native</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — via iRules + LTM policies on BIG-IP</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Path-Based Routing</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — VirtualServer CRD</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Query Parameter Routing</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — native in HTTPRoute spec</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial — via iRules</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Cookie-Based Routing</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — canary via cookie annotation</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP persistence profiles</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Method-Based Routing</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — native</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial — via iRules / LTM policy</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Cross-Namespace Routing</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes (v5.3+ — upstream from different NS)</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Limited</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — native via ReferenceGrant</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — CIS watches multiple namespaces; multi-cluster service discovery</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">TLS Passthrough</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — via TransportServer</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — TLSRoute passthrough</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — TLSRoute</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — TLSProfile passthrough + BIG-IP SSL passthrough</td>
    </tr>
    <tr>
      <td colspan="6" style="background:#1f3864;color:#fff;font-weight:bold;padding:8px 12px;border:1px solid #bdbdbd;">🔷  LOAD BALANCING</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Layer 7 Load Balancing</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — high-performance via TMM</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — full BIG-IP LTM</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Layer 4 Load Balancing</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — TCP/UDP via TransportServer</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial</td>
      <td style="background:#e3f2fd;color:#0d47a1;padding:7px 12px;border:1px solid #e0e0e0;">🔄 Roadmap</td>
      <td style="background:#fafafa;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — TransportServer + BIG-IP LTM TCP/UDP</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Round Robin</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Least Connections</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes (Plus: full upstream config)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#fafafa;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP LTM load balancing methods</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">IP Hash / Sticky Sessions</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP persistence profiles (cookie, source IP, SSL)</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Session Persistence</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes (cookie-based + Plus: full)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — cookie annotation</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#fafafa;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — full BIG-IP persistence (cookie, source IP, custom)</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Active Health Checks</td>

      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 NGINX Plus only</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Passive only (OSS)</td>
      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 NGINX Plus only</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — TMM-native active health checks</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP native active health monitors (HTTP, TCP, ICMP, custom)</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Passive Health Checks</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes (OSS)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes (OSS)</td>
      <td style="background:#fafafa;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Dynamic Reconfiguration</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — no NGINX reload required (Plus)</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Requires NGINX reload</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes (Plus)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — TMM hot reconfiguration, no reload</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — AS3 declarative push; no BIG-IP reload required</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Upstream Zone / State Sharing</td>

      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 NGINX Plus — key-value store, zones</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 NGINX Plus</td>
      <td style="background:#fafafa;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP HA config-sync; shared state across BIG-IP pairs</td>
    </tr>
    <tr>
      <td colspan="6" style="background:#1f3864;color:#fff;font-weight:bold;padding:8px 12px;border:1px solid #bdbdbd;">🔷  TRAFFIC MANAGEMENT &amp; DEPLOYMENT PATTERNS</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Traffic Splitting / Weighting</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — VirtualServer spec</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — canary annotation</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — native HTTPRoute weight</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — VirtualServer CRD pools with weights</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Canary Deployments</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — pool weighting</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Blue-Green Deployments</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial — via annotations</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — native</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — OpenShift Routes A/B + CIS CRD pools</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">A/B Testing</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Rate Limiting</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Policy CRD</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — annotation</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — RateLimit policy</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Policy CRD + BIG-IP rate shaping</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Circuit Breaking</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP connection limits + health monitors</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Request Buffering</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP HTTP profiles</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Request/Response Header Manipulation</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — annotations</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — native HTTPRoute filters</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — HTTPRoute filters</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — iRules + BIG-IP HTTP profile header insertion</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">URL Rewriting / Redirects</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — native HTTPRoute filters</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — VirtualServer rewrite + iRules</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Request Mirroring</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — native</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP mirroring</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Direct Response Injection</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — native</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — iRules</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Caching</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes (v5.3+ — configurable cache policy)</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Limited</td>
      <td style="background:#e3f2fd;color:#0d47a1;padding:7px 12px;border:1px solid #e0e0e0;">🔄 Roadmap</td>
      <td style="background:#e3f2fd;color:#0d47a1;padding:7px 12px;border:1px solid #e0e0e0;">🔄 Roadmap</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP RAM Cache (Web Acceleration profile)</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Connection Limiting</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP virtual server connection limits</td>
    </tr>
    <tr>
      <td colspan="6" style="background:#1f3864;color:#fff;font-weight:bold;padding:8px 12px;border:1px solid #bdbdbd;">🔷  SECURITY</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">TLS/SSL Termination</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — hardware-accelerated SSL on BIG-IP</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">mTLS (Mutual TLS)</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Policy CRD (client &amp; backend)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — annotations</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BackendTLSPolicy</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — C3D (Client Certificate Constrained Delegation) + mutual auth profiles</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">JWT Authentication</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes (OSS + Plus)</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Limited — third-party auth only</td>
      <td style="background:#e3f2fd;color:#0d47a1;padding:7px 12px;border:1px solid #e0e0e0;">🔄 Roadmap</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP APM + iRules</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">OIDC Authentication</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — OIDC + v5.3+ enhancements</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Via external auth-url annotation</td>
      <td style="background:#e3f2fd;color:#0d47a1;padding:7px 12px;border:1px solid #e0e0e0;">🔄 Roadmap</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP APM OIDC/OAuth federation, SSO, MFA</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Basic Authentication</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">RBAC</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Kubernetes native RBAC</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Kubernetes native RBAC</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — enhanced, role-based model</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Kubernetes native + Gateway API role model</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Kubernetes RBAC + BIG-IP partition-level RBAC</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">WAF (Web Application Firewall)</td>

      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 NGINX App Protect (Plus) — OWASP Top 10, gRPC schema, OpenAPI</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ ModSecurity (limited)</td>
      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 F5 WAF for NGINX (Plus/NGINX One)</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — F5 Advanced WAF (OWASP Top 10, bot defense, API security, L3/L7 firewall via Policy CRD)</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">DDoS Protection</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — combined with F5 security stack</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Basic</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — combined with F5 security stack</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — full DDoS mitigation, edge firewall</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — F5 Advanced WAF + BIG-IP DoS protection profiles</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">IP Allowlist/Denylist</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Policy CRD + BIG-IP iRules + network firewall</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">CORS Policy</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — annotation</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — native HTTPRoute filter</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — iRules + BIG-IP HTTP profiles</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Secret Management (TLS)</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Kubernetes Secrets</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Kubernetes Secrets</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Kubernetes Secrets</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Kubernetes Secrets; BIG-IP certificate management</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">End-to-End Encryption</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — TLS passthrough &amp; termination</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — hardware-offloaded encryption on DPU</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — SSL bridging, SSL passthrough, re-encryption</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Zero Trust / Shift Left Security</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Policy CRD enforcement</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Limited</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — role-based policy model</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — role-based policy model</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP APM zero-trust; identity-aware ingress</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">OpenID Connect Enhancements</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes (v5.3+ token timeouts, front-channel logout)</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Basic</td>
      <td style="background:#e3f2fd;color:#0d47a1;padding:7px 12px;border:1px solid #e0e0e0;">🔄 Roadmap</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP APM full OIDC (federation, MFA, token validation)</td>
    </tr>
    <tr>
      <td colspan="6" style="background:#1f3864;color:#fff;font-weight:bold;padding:8px 12px;border:1px solid #bdbdbd;">🔷  OBSERVABILITY &amp; MONITORING</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Prometheus Metrics</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — CIS exports AS3 HTTP client metrics; BIG-IP Telemetry Streaming</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Grafana Integration</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — via BIG-IP Telemetry Streaming + Prometheus</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">OpenTelemetry / Tracing</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — OTel, Jaeger, Zipkin</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial — via BIG-IP Analytics; not native OTel</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Access Logging</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP request logging profiles; High-Speed Logging (HSL)</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Structured JSON Logging</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP Telemetry Streaming JSON output</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Real-Time Metrics Dashboard</td>

      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 NGINX Plus — NGINX One Console</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ Not built-in ✅ Yes through Grafana and Prometheus integrations</td>
      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 NGINX Plus — NGINX One Console</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ Not built-in ✅ Yes through Grafana and Prometheus integrations</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP Analytics + Telemetry Streaming dashboard</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Health Check Endpoint</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — CIS /ready endpoint + BIG-IP health monitors</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">NGINX One Console</td>

      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 Available with NGINX One subscription</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 Available with NGINX One subscription</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No — uses BIG-IP management (TMUI, TMOS CLI, F5 ADSP)</td>
    </tr>
    <tr>
      <td colspan="6" style="background:#1f3864;color:#fff;font-weight:bold;padding:8px 12px;border:1px solid #bdbdbd;">🔷  EXTENSIBILITY</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Custom Extensions Model</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ CRDs + Policy resources + NGINX snippets</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Annotations + Lua modules (fragile)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Policy Attachments — annotation-less model</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — iRules (TCL) + AS3 ConfigMap + Policy CRD + BIG-IP profiles</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">NGINX Configuration Snippets</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — nginx.ingress.kubernetes.io/configuration-snippet</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No — by design (cleaner model)</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No — not NGINX-based</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No — not NGINX-based</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Lua Scripting</td>

      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No (uses native NGINX modules)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Lua-based extensions</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No — uses iRules (TCL-based programmable data plane)</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Custom Policies / Filters</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Policy CRD</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Via annotations only</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Policy attachments (extensionRef)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BNKSecPolicy, BNKNetPolicy, F5BigFwPolicy</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Policy CRD (WAF, firewall, persistence, iRules, profiles)</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Plugin Ecosystem</td>

      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Limited — NGINX modules</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Limited — Lua plugins</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Growing — Gateway API extension points</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — full BIG-IP module ecosystem (APM, ASM/WAF, DNS, AFM, GTM)</td>
    </tr>
    <tr>
      <td colspan="6" style="background:#1f3864;color:#fff;font-weight:bold;padding:8px 12px;border:1px solid #bdbdbd;">🔷  PLATFORM &amp; INTEGRATIONS</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Kubernetes Versions</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ All modern versions</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ All modern versions (until EOL)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ All modern versions</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ All modern versions — see CIS compatibility matrix</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Red Hat OpenShift</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — certified</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — Red Hat certified; NextGen Routes + OLM support</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Service Mesh Integration (Istio/Linkerd)</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — north-south + east-west</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial — north-south LB; Istio ambient mesh (HBONE) not natively supported. Aspen Mesh</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">F5 BIG-IP / IngressLink</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — IngressLink CRD integration</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Via CIS only</td>
      <td style="background:#e3f2fd;color:#0d47a1;padding:7px 12px;border:1px solid #e0e0e0;">🔄 Roadmap</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — same TMM engine; part of F5 ADSP ecosystem</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — CIS IS the BIG-IP connector; IngressLink for NIC+BIG-IP 2-tier</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">F5 Distributed Cloud</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#e3f2fd;color:#0d47a1;padding:7px 12px;border:1px solid #e0e0e0;">🔄 Roadmap</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP Distributed Cloud integration</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Cert-Manager</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">ExternalDNS</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — ExternalDNS CRD + F5 IPAM Controller (FIC) for IP management</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">AWS / Azure / GCP</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — cloud-agnostic</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — hybrid/multicloud by design</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — private cloud, colocated DC, and hybrid</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP VE on public cloud; hybrid multi-cloud deployments</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">ingress2gateway Migration Tool</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — NGINX provider included</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — source</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — target destination</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No — CIS uses Ingress/CRD model, not Gateway API; no migration tool</td>
    </tr>
    <tr>
      <td colspan="6" style="background:#1f3864;color:#fff;font-weight:bold;padding:8px 12px;border:1px solid #bdbdbd;">🔷  OPERATIONAL &amp; ENTERPRISE</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">High Availability</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — multi-replica deployment</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP HA pairs (Active/Standby); Primary/Secondary CIS instances</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Horizontal Scaling</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — scales from core to far-edge</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial — CIS scales; BIG-IP scales via VE licensing or hardware</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">State Sharing (rate limits, etc.)</td>

      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 NGINX Plus — key-value + zone sharing</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 NGINX Plus</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP HA config-sync; connection mirroring</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Commercial Support (SLA)</td>

      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 NGINX Plus / NGINX One</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 NGINX Plus / NGINX One</td>
      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 Yes — F5 enterprise support with SLA</td>
      <td style="background:#f3e5f5;color:#4a148c;padding:7px 12px;border:1px solid #e0e0e0;">💰 Yes — included with F5 BIG-IP support entitlement</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Security Patch Cadence</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Active — dedicated F5 team</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Critical only until EOL — then none</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Active — dedicated F5 team</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Active — dedicated F5 team</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Active — F5 security advisories + quarterly CIS releases</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Upgrade Path from ingress-nginx</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Best-supported — migration guide + tool</td>
      <td style="background:#f5f5f5;color:#212121;padding:7px 12px;border:1px solid #e0e0e0;">N/A</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Via ingress2gateway tool</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Significant migration — different architecture &amp; CRDs</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Significant migration — different architecture (external BIG-IP)</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Vendor Lock-in</td>

      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Moderate — NGINX-specific CRDs</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Moderate — annotation-based</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Low — standardized Gateway API</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ High — requires F5 BIG-IP hardware/VE; iRules are proprietary</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Configuration Portability</td>

      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ NGINX-specific</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ NGINX-specific</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ High — portable across Gateway API impls</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Limited — Gateway API routes portable; F5 CRDs are proprietary</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Low — F5-specific CRDs + iRules; AS3 JSON is F5-only</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Long-Term Roadmap</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Active — Ingress API + Gateway API path</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ EOL — no roadmap</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Future-proof — Gateway API is K8s standard</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Active — multi-cluster, NextGen Routes, Gateway API integration planned</td>
    </tr>
    <tr>
      <td colspan="6" style="background:#1f3864;color:#fff;font-weight:bold;padding:8px 12px;border:1px solid #bdbdbd;">🔷  PROTOCOL SUPPORT</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">HTTP/1.1</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">HTTP/2</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP HTTP/2 full proxy</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">HTTP/3 (QUIC)</td>

      <td style="background:#fff3e0;color:#bf360c;padding:7px 12px;border:1px solid #e0e0e0;">🧪 Experimental</td>
      <td style="background:#fff3e0;color:#bf360c;padding:7px 12px;border:1px solid #e0e0e0;">🧪 Experimental</td>
      <td style="background:#e3f2fd;color:#0d47a1;padding:7px 12px;border:1px solid #e0e0e0;">🔄 Roadmap</td>
      <td style="background:#e3f2fd;color:#0d47a1;padding:7px 12px;border:1px solid #e0e0e0;">🔄 Roadmap</td>
      <td style="background:#e3f2fd;color:#0d47a1;padding:7px 12px;border:1px solid #e0e0e0;">🔄 Roadmap — BIG-IP QUIC support in progress</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">HTTPS</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">TCP</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — TransportServer</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Via ConfigMap</td>
      <td style="background:#e3f2fd;color:#0d47a1;padding:7px 12px;border:1px solid #e0e0e0;">🔄 Roadmap — TCPRoute</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — TransportServer + BIG-IP LTM</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">UDP</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — TransportServer</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Via ConfigMap</td>
      <td style="background:#e3f2fd;color:#0d47a1;padding:7px 12px;border:1px solid #e0e0e0;">🔄 Roadmap — UDPRoute</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — L4Route (UDP)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — TransportServer + BIG-IP LTM</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">gRPC</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — native</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial — requires annotations</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — GRPCRoute</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial — via HTTP/2 profile; no native gRPC CRD</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">WebSocket</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP WebSocket profile</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">TLS Passthrough</td>

      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Via annotation</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — TLSRoute</td>
      <td style="background:#ffffff;color:#9e9e9e;padding:7px 12px;border:1px solid #e0e0e0;"><span style="color:#bdbdbd;">—</span></td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — TLSProfile passthrough + BIG-IP SSL passthrough</td>
    </tr>
    <tr>
      <td colspan="6" style="background:#1f3864;color:#fff;font-weight:bold;padding:8px 12px;border:1px solid #bdbdbd;">🔷  BNK-EXCLUSIVE CAPABILITIES (not available in NGINX-based solutions)</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Telco Protocols (SCTP, Diameter, SIP, GTP, NGAP)</td>

      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — native TMM support for 4G/5G telco protocols</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial — BIG-IP supports SCTP; limited 5G native support vs BNK</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">NVIDIA BlueField DPU Acceleration</td>

      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — hardware offload: crypto, network processing, security</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No — BIG-IP uses FPGA/ASIC hardware offload (not BlueField DPU)</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Egress Traffic Management (SNAT pools)</td>

      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Limited</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Limited</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Limited</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — F5SPKEgress + F5SPKSnatpool CRDs</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP SNAT pools natively; exposed via CIS CRDs</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Intrusion Prevention System (IPS)</td>

      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — F5-IPSD (SNORT-compatible deep packet inspection)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP AFM (Advanced Firewall Manager) IPS/IDS</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Topology Hiding (internal cluster obscuring)</td>

      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — ingress topology hiding</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP SNAT + topology hiding via VIP abstraction</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Per-Subscriber Traffic Visibility / Billing</td>

      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — compliance tracking and billing telemetry</td>
      <td style="background:#fff8e1;color:#e65100;padding:7px 12px;border:1px solid #e0e0e0;">⚠️ Partial — BIG-IP Analytics; not Kubernetes-native subscriber model</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">AI / LLM Traffic Routing (f5-analyzer)</td>

      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — f5-analyzer routes LLM traffic based on complexity/load</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No — not available in CIS</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">MCP Server Proxy (Agentic AI)</td>

      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — reverse proxy for MCP servers with auth &amp; data classification</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No — not available in CIS</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">Firewall Policy (Stateful ACL)</td>

      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — F5BigFwPolicy — granular stateful ACL</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP AFM stateful firewall + Policy CRD L3/L7 firewall</td>
    </tr>
    <tr>
      <td style="background:#fafafa;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">iRules (Programmable Data Plane)</td>

      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — F5 iRules (TCL-based programmable data path)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — full iRules support via BIG-IP; referenced in Policy/VS CRDs</td>
    </tr>
    <tr>
      <td style="background:#ffffff;color:#212121;font-weight:500;padding:7px 12px;border:1px solid #e0e0e0;white-space:nowrap;">CPU Offload / Energy Efficiency</td>

      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#ffebee;color:#b71c1c;padding:7px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — 99% lower CPU utilization vs NGINX; 190x energy efficiency (BlueField)</td>
      <td style="background:#e8f5e9;color:#1b5e20;padding:7px 12px;border:1px solid #e0e0e0;">✅ Yes — BIG-IP FPGA/ASIC hardware offload (SSL, compression, etc.)</td>
    </tr>
    <tr>
      <td colspan="6" style="background:#424242;color:#fff;font-weight:bold;padding:8px 12px;border:1px solid #bdbdbd;">📖 Legend</td>
    </tr>
    <tr>
      <td style="background:#e8f5e9;color:#1b5e20;font-weight:bold;padding:6px 12px;border:1px solid #e0e0e0;">✅ Yes</td>
      <td colspan="5" style="background:#e8f5e9;color:#424242;padding:6px 12px;border:1px solid #e0e0e0;">Feature fully supported</td>
    </tr>
    <tr>
      <td style="background:#ffebee;color:#b71c1c;font-weight:bold;padding:6px 12px;border:1px solid #e0e0e0;">❌ No</td>
      <td colspan="5" style="background:#ffebee;color:#424242;padding:6px 12px;border:1px solid #e0e0e0;">Feature not available</td>
    </tr>
    <tr>
      <td style="background:#fff8e1;color:#e65100;font-weight:bold;padding:6px 12px;border:1px solid #e0e0e0;">⚠️ Partial / Limited</td>
      <td colspan="5" style="background:#fff8e1;color:#424242;padding:6px 12px;border:1px solid #e0e0e0;">Feature partially supported or has caveats</td>
    </tr>
    <tr>
      <td style="background:#f3e5f5;color:#4a148c;font-weight:bold;padding:6px 12px;border:1px solid #e0e0e0;">💰 Commercial (Plus)</td>
      <td colspan="5" style="background:#f3e5f5;color:#424242;padding:6px 12px;border:1px solid #e0e0e0;">Available in NGINX Plus or NGINX One commercial tier only</td>
    </tr>
    <tr>
      <td style="background:#e3f2fd;color:#0d47a1;font-weight:bold;padding:6px 12px;border:1px solid #e0e0e0;">🔄 Roadmap</td>
      <td colspan="5" style="background:#e3f2fd;color:#424242;padding:6px 12px;border:1px solid #e0e0e0;">Feature planned but not yet available</td>
    </tr>
    <tr>
      <td style="background:#fff3e0;color:#bf360c;font-weight:bold;padding:6px 12px;border:1px solid #e0e0e0;">🧪 Experimental</td>
      <td colspan="5" style="background:#fff3e0;color:#424242;padding:6px 12px;border:1px solid #e0e0e0;">Feature available but not production-ready</td>
    </tr>
    <tr>
      <td style="background:#f5f5f5;color:#212121;font-weight:bold;padding:6px 12px;border:1px solid #e0e0e0;">—  N/A</td>
      <td colspan="5" style="background:#f5f5f5;color:#424242;padding:6px 12px;border:1px solid #e0e0e0;">Not applicable</td>
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

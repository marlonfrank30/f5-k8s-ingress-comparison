# 🧭 Decision Guide — Choosing the Right F5 Kubernetes Ingress Solution

Use this guide to determine which solution best fits your environment and requirements.

---

## Decision Flowchart

```
START: What is your primary constraint or driver?
│
├─► Are you still running community ingress-nginx?
│   └─► YES ──► ⚠️  MIGRATE NOW — EOL since March 2026
│               │
│               ├─► Fastest migration, same NGINX engine?
│               │   └──► F5 NGINX Ingress Controller (NIC)
│               │
│               ├─► Long-term portability, multi-tenant platform?
│               │   └──► F5 NGINX Gateway Fabric (NGF)
│               │
│               └─► Existing BIG-IP hardware / OpenShift?
│                   └──► F5 BIG-IP CIS
│
├─► Do you have an existing F5 BIG-IP investment?
│   └─► YES ──► F5 BIG-IP Container Ingress Services (CIS)
│               (leverages existing hardware/licensing)
│
├─► Are you in Telco / Service Provider / 5G / AI Factory?
│   └─► YES ──► F5 BIG-IP Next for Kubernetes (BNK)
│               (carrier-grade protocols + DPU offload)
│
├─► Are you building a greenfield cloud-native platform?
│   └─► YES ──► F5 NGINX Gateway Fabric (NGF)
│               (Gateway API standard, multi-tenant, no lock-in)
│
├─► Do you need TCP/UDP ingress today?
│   └─► YES ──► F5 NGINX Ingress Controller (NIC)
│               (TransportServer CRD — NGF has this on roadmap only)
│
└─► Default enterprise cloud-native ingress?
    └──► F5 NGINX Ingress Controller (NIC)
         (proven, full-featured, broad community)
```

---

## Scoring Matrix

Score each criterion 1–5 based on importance to your environment, then multiply by the product rating.

| Criterion | Weight | NIC | ingress-nginx | NGF | BNK | CIS |
|---|---|---|---|---|---|---|
| Active maintenance / security patches | High | ✅ 5 | ❌ 0 | ✅ 5 | ✅ 5 | ✅ 5 |
| Migration effort from ingress-nginx | High | ✅ 5 | — | ✅ 4 | ⚠️ 2 | ⚠️ 2 |
| Gateway API / portability | Medium | ⚠️ 2 | ⚠️ 1 | ✅ 5 | ✅ 5 | ⚠️ 2 |
| L4 TCP/UDP today | Medium | ✅ 5 | ⚠️ 2 | ❌ 1 | ✅ 5 | ✅ 5 |
| WAF capability | High | 💰 4 | ⚠️ 2 | 💰 4 | ✅ 5 | ✅ 5 |
| Multi-tenancy / role model | Medium | ⚠️ 2 | ⚠️ 2 | ✅ 5 | ✅ 5 | ✅ 4 |
| Hardware offload / performance | Low-Med | ❌ 1 | ❌ 1 | ❌ 1 | ✅ 5 | ✅ 4 |
| Telco / 5G protocol support | Use-case | ❌ 1 | ❌ 1 | ❌ 1 | ✅ 5 | ⚠️ 2 |
| Existing BIG-IP leverage | Use-case | ⚠️ 2 | ❌ 1 | ⚠️ 2 | ⚠️ 3 | ✅ 5 |
| OIDC / APM / MFA | Medium | ✅ 4 | ⚠️ 1 | 🔄 2 | ✅ 4 | ✅ 5 |
| Vendor lock-in (lower = better) | Medium | ⚠️ 3 | ⚠️ 3 | ✅ 5 | ⚠️ 3 | ❌ 2 |
| OSS / zero licensing cost | Low | ✅ 5 | ✅ 5 | ✅ 5 | ❌ 1 | ⚠️ 3 |

---

## Use Case Profiles

### Profile A — Enterprise Web Application Platform (Cloud-Native, New Build)

**Recommendation: NGF**

You are building a new Kubernetes platform from scratch. You have multiple application teams deploying into the same cluster. You want a clean separation of concerns between platform engineers (who manage the Gateway) and developers (who manage HTTPRoutes). You care about long-term portability and not being locked into proprietary annotations.

Choose NGF. It is the only solution with a proper three-role model and native multi-tenant design. Its Gateway API conformance means your HTTPRoute definitions are portable.

---

### Profile B — Migration from ingress-nginx (Brownfield)

**Recommendation: NIC**

You are running `kubernetes/ingress-nginx` and need to migrate urgently due to EOL. You want the path of least resistance — same NGINX data plane, similar configuration model, and the `ingress2gateway` tool to automate annotation migration.

Choose NIC. Nearly all ingress-nginx features have a direct NIC equivalent. The migration guide covers the most common annotation mappings.

---

### Profile C — Enterprise with Existing BIG-IP Hardware

**Recommendation: CIS**

Your organisation already owns F5 BIG-IP appliances or has VE licensing. BIG-IP is already in your traffic path. You want to extend it into Kubernetes without replacing hardware, and you need the full BIG-IP feature set: APM for SSO/MFA, Advanced WAF, iRules, SNAT pools, hardware SSL offload.

Choose CIS. It connects your existing BIG-IP to Kubernetes with a lightweight in-cluster controller, and exposes the full BIG-IP feature set through Kubernetes-native CRDs and AS3.

---

### Profile D — OpenShift Environment

**Recommendation: CIS (primary) or NIC (secondary)**

OpenShift has specific requirements — route handling, OLM deployment, Red Hat certification. CIS has the deepest OpenShift integration: NextGen Routes support, Red Hat certification, OLM via OperatorHub, and multi-cluster capabilities for OpenShift migrations. NIC is also OpenShift-compatible and is a good choice if you do not have BIG-IP.

---

### Profile E — Telco / Service Provider / 5G Core

**Recommendation: BNK**

You are deploying a 5G core (UPF, SMF, AMF), 4G EPC, or other carrier-grade Kubernetes workload. You need SCTP, GTP, Diameter, SIP, NGAP protocol support. You need carrier-grade performance and per-subscriber visibility. Your infrastructure includes NVIDIA SmartNICs or plans to.

Choose BNK. It is the only solution in this comparison with native telco protocol support and DPU acceleration.

---

### Profile F — AI Factory / LLM Inference

**Recommendation: BNK**

You are building an AI inference platform and need intelligent routing of LLM traffic based on request complexity and GPU backend availability. You need high throughput with minimum CPU overhead to leave compute for inference.

Choose BNK. The `f5-analyzer` module provides LLM-aware routing. DPU offload recovers CPU cycles spent on SSL and ACL enforcement back to inference workloads.

---

## Two-Tier Architecture (NIC + CIS IngressLink)

For organisations that want cloud-native developer experience *and* BIG-IP enterprise capabilities, CIS supports a two-tier model via `IngressLink`:

```
External Client
      │
      ▼
┌───────────────────────┐
│  BIG-IP (External)    │  ← L4 load balancing, SSL offload,
│  CIS + IngressLink    │    DDoS, global traffic management
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│  NIC (In-Cluster)     │  ← L7 routing, path-based rules,
│  VirtualServer CRDs   │    NGINX App Protect WAF, JWT
└──────────┬────────────┘
           │
           ▼
    Backend Services
```

This model lets NetOps manage the BIG-IP tier (CIS) while DevOps manages the NGINX tier (NIC), with clean team boundaries.

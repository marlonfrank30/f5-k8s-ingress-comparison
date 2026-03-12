# 📦 Product Overview — F5 Kubernetes Ingress Solutions

This document provides a deep-dive on each of the five solutions covered in this comparison.

---

## 1. F5 NGINX Ingress Controller (NIC)

**Repository:** [`nginxinc/kubernetes-ingress`](https://github.com/nginxinc/kubernetes-ingress)
**License:** Apache 2.0 (OSS) + Commercial (NGINX Plus)

### What it is
NIC is the F5-maintained NGINX-based Kubernetes ingress controller. It uses the standard Kubernetes Ingress API and extends it with a rich set of Custom Resource Definitions (CRDs) — `VirtualServer`, `VirtualServerRoute`, `TransportServer`, `Policy`, and `GlobalConfiguration`.

### Key Strengths
- **Production-proven.** Deployed in tens of thousands of Kubernetes clusters globally.
- **Richest NGINX feature surface.** Native gRPC, TCP/UDP via TransportServer, JWT, OIDC, rate limiting, mTLS — all through CRDs.
- **Best migration target from ingress-nginx.** Familiar NGINX mental model; `ingress2gateway` tooling supports NIC as a provider.
- **NGINX Plus integration.** Active health checks, dynamic reconfiguration without reload, NGINX One Console, key-value store, upstream zone sharing.
- **NGINX App Protect WAF.** OWASP Top 10, gRPC schema validation, OpenAPI protection.

### CRD Reference

| CRD | Purpose |
|---|---|
| `VirtualServer` | Primary L7 virtual host definition (replaces Ingress) |
| `VirtualServerRoute` | Modular route definition for multi-team deployments |
| `TransportServer` | L4 TCP/UDP virtual server |
| `Policy` | Rate limiting, JWT, OIDC, WAF, mTLS, IP access control |
| `GlobalConfiguration` | Cluster-wide listener and upstream defaults |

### When to Use NIC
- Migrating from community `ingress-nginx` with minimum friction
- TCP/UDP ingress required today (not on NGF roadmap yet)
- NGINX App Protect WAF requirement with NGINX-native configuration model
- Teams already familiar with NGINX

---

## 2. Community Ingress-NGINX ⚠️ EOL March 2026

**Repository:** [`kubernetes/ingress-nginx`](https://github.com/kubernetes/ingress-nginx)
**License:** Apache 2.0

> ⚠️ **This project reached End of Life in March 2026.** No further security patches will be released. Migration is urgent.

### What it is
The community-maintained ingress controller originally built by the Kubernetes project. Historically the most widely deployed Kubernetes ingress solution. It is annotation-driven, Lua-extended NGINX, and uses the standard Ingress API.

### Why it is EOL
The project maintainers announced feature freeze and EOL in 2025, citing:
- Architectural limitations of the annotation model
- Fragility of the Lua extension approach
- Kubernetes Gateway API superseding the Ingress API as the standard

### Migration Paths
See the full [Migration Guide](migration-guide.md).

---

## 3. F5 NGINX Gateway Fabric (NGF)

**Repository:** [`nginx/nginx-gateway-fabric`](https://github.com/nginx/nginx-gateway-fabric)
**License:** Apache 2.0 (OSS) + Commercial (NGINX Plus / NGINX One)

### What it is
NGF is F5's implementation of the Kubernetes **Gateway API** — the CNCF-standardised next-generation successor to the Ingress API. It is the strategic long-term platform for cloud-native Kubernetes ingress at F5.

### Key Strengths
- **CNCF Gateway API conformant.** Routes (`HTTPRoute`, `GRPCRoute`, `TLSRoute`) are portable across any Gateway API implementation.
- **Three-role model.** Infrastructure Provider → Cluster Operator → Application Developer. Purpose-built for platform engineering teams.
- **Annotation-less design.** All configuration is schema-validated through Gateway API CRDs.
- **Cross-namespace routing.** Native `ReferenceGrant` support.
- **`ingress2gateway` migration target.** Automated migration from ingress-nginx annotations to HTTPRoute resources.

### Gateway API Resource Model

```
GatewayClass  (set by Infrastructure Provider)
      │
      ▼
  Gateway      (managed by Cluster Operator)
      │
      ├── HTTPRoute   (managed by App Developer)
      ├── GRPCRoute
      ├── TLSRoute
      └── ReferenceGrant  (cross-namespace access control)
```

### Current Limitations (Roadmap)
- L4 TCP/UDP routing (TCPRoute/UDPRoute)
- JWT and OIDC authentication
- Response caching

### When to Use NGF
- Greenfield Kubernetes platforms
- Platform teams serving multiple application teams (multi-tenant)
- Long-term portability is a priority
- Adopting Gateway API as the organisation-wide standard

---

## 4. F5 BIG-IP Next for Kubernetes (BNK)

**Documentation:** [clouddocs.f5.com/bigip-next-for-kubernetes](https://clouddocs.f5.com/bigip-next-for-kubernetes/latest)
**License:** Commercial — F5 / MyF5 entitlement required

### What it is
BNK brings the F5 Traffic Management Microkernel (TMM) natively into Kubernetes as a pod-based data plane. It implements the Gateway API plus F5-specific CRDs for advanced security, networking, and carrier-grade capabilities. It optionally offloads processing to NVIDIA BlueField-3 DPUs.

### Key Strengths
- **F5 TMM data plane in-cluster.** Same engine as BIG-IP hardware, running as Kubernetes pods.
- **DPU acceleration.** NVIDIA BlueField-3 integration. SoftBank PoC: 99% lower CPU utilization vs software NGINX; 190x energy efficiency.
- **Native telco protocols.** SCTP, GTP, Diameter, SIP, NGAP — carrier-grade Kubernetes.
- **AI / LLM Traffic Routing.** `f5-analyzer` routes LLM inference traffic by request complexity and backend load.
- **MCP Server Proxy.** Reverse proxy for Model Context Protocol servers with authentication and data classification.
- **Advanced security.** F5 WAF, F5-IPSD IPS (SNORT-compatible), F5BigFwPolicy stateful ACL, DDoS mitigation.

### BNK CRD Reference

| CRD | Purpose |
|---|---|
| `CNEInstance` | BNK instance configuration |
| `BNKSecPolicy` | Security policy (WAF, auth, rate limit) |
| `BNKNetPolicy` | Network policy enforcement |
| `F5BigFwPolicy` | Stateful L3/L4 firewall ACL |
| `F5SPKEgress` | Egress traffic control |
| `F5SPKSnatpool` | SNAT pool configuration |
| `F5SPKGlobalOptions` | Global TMM options |

### When to Use BNK
- Telco / service provider Kubernetes (5G core, 4G EPC, vIMS)
- AI factory and LLM inference infrastructure
- High-throughput workloads where NGINX CPU cost is prohibitive
- Unified security: WAF + IPS + stateful firewall in a single in-cluster data plane

---

## 5. F5 BIG-IP Container Ingress Services (CIS)

**Repository:** [`F5Networks/k8s-bigip-ctlr`](https://github.com/F5Networks/k8s-bigip-ctlr)
**License:** Apache 2.0 (CIS itself) — requires F5 BIG-IP entitlement

### What it is
CIS is an **external ADC connector** — a Kubernetes controller that watches cluster resources and translates them into BIG-IP configuration via AS3 (declarative API) or iControl REST. The data plane is an F5 BIG-IP appliance or Virtual Edition running outside the cluster.

### Key Strengths
- **Full BIG-IP module ecosystem.** APM (SSO, OIDC, MFA), Advanced WAF, AFM (IPS/IDS, stateful firewall), GTM/DNS, LTM.
- **Hardware-accelerated SSL.** FPGA/ASIC offload on BIG-IP appliances.
- **Enterprise HA.** BIG-IP Active/Standby pairs with config-sync + connection mirroring. Primary/Secondary CIS redundancy.
- **iRules.** Full TCL-based programmable data plane referenced from CRDs.
- **OpenShift native.** Red Hat certified, NextGen Routes, OLM deployment.
- **Multi-cluster.** Near-zero-downtime migrations via multi-cluster service discovery.

### CIS Resource Model

| Resource | Purpose |
|---|---|
| `VirtualServer` | L7 virtual host with pools and policies |
| `TransportServer` | L4 TCP/UDP virtual server |
| `IngressLink` | Two-tier: NIC as L7 front-end, BIG-IP as L4 front-end |
| `Policy` | WAF, firewall, persistence, iRules, profiles |
| `ExternalDNS` | BIG-IP GTM DNS record management |
| `TLSProfile` | TLS termination / passthrough configuration |
| AS3 ConfigMap | Raw AS3 JSON for advanced BIG-IP configuration |

### When to Use CIS
- Existing F5 BIG-IP investment that needs Kubernetes integration
- OpenShift environments
- Full APM capabilities (OIDC federation, MFA, SSO) at ingress
- NetOps + DevOps model where network team manages BIG-IP separately

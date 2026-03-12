# 🔄 Migration Guide — From ingress-nginx (EOL) to F5 Solutions

Community `kubernetes/ingress-nginx` reached **End of Life in March 2026**. This guide covers migration paths to the three F5 solutions that make sense as migration targets.

---

## Why You Must Migrate Now

After EOL:
- No security patches for NGINX CVEs
- No bug fixes
- No Kubernetes version compatibility updates
- The project repository is archived

Running EOL ingress on a production cluster is a compliance and security risk.

---

## Migration Path Overview

```
ingress-nginx (EOL)
        │
        ├──► Path 1: F5 NGINX Ingress Controller (NIC)
        │    Effort: LOW  │  Risk: LOW  │  Feature parity: HIGH
        │
        ├──► Path 2: F5 NGINX Gateway Fabric (NGF)
        │    Effort: MEDIUM  │  Risk: LOW  │  Portability: HIGH
        │
        └──► Path 3: F5 BIG-IP CIS
             Effort: HIGH  │  Risk: MEDIUM  │  Requires: BIG-IP
```

---

## Path 1: ingress-nginx → NIC (Recommended for most)

### Overview
NIC uses the same NGINX data plane. Most ingress-nginx annotations have direct CRD equivalents in NIC. The `ingress2gateway` tool (with NIC provider) automates the conversion of standard Ingress resources.

### Common Annotation Mappings

| ingress-nginx Annotation | NIC Equivalent |
|---|---|
| `nginx.ingress.kubernetes.io/proxy-connect-timeout` | `VirtualServer.spec.upstreams[].connect-timeout` |
| `nginx.ingress.kubernetes.io/proxy-read-timeout` | `VirtualServer.spec.upstreams[].read-timeout` |
| `nginx.ingress.kubernetes.io/rate-limit` | `Policy` CRD — rateLimit |
| `nginx.ingress.kubernetes.io/auth-type: basic` | `Policy` CRD — basicAuth |
| `nginx.ingress.kubernetes.io/auth-url` (OIDC) | `Policy` CRD — oidc |
| `nginx.ingress.kubernetes.io/enable-cors` | `VirtualServer.spec.routes[].action.proxy.requestHeaders` |
| `nginx.ingress.kubernetes.io/rewrite-target` | `VirtualServer.spec.routes[].action.rewrite` |
| `nginx.ingress.kubernetes.io/ssl-redirect` | `VirtualServer` — redirect action |
| `nginx.ingress.kubernetes.io/use-regex` | `VirtualServer.spec.routes[].path` (regex supported) |
| `nginx.ingress.kubernetes.io/proxy-buffering` | `VirtualServer.spec.upstreams[].buffering` |
| `nginx.ingress.kubernetes.io/configuration-snippet` | `VirtualServer` — use `upstreamSettingsPolicy` or `Policy` CRD |

### Step-by-Step Migration

**Step 1: Install NIC alongside ingress-nginx (parallel)**
```bash
helm repo add nginx-stable https://helm.nginx.com/stable
helm install nic nginx-stable/nginx-ingress \
  --namespace nginx-ingress \
  --create-namespace \
  --set controller.ingressClass=nginx-new
```

**Step 2: Run ingress2gateway (optional — for Ingress → HTTPRoute)**
```bash
# Install the tool
go install sigs.k8s.io/ingress2gateway@latest

# Generate NIC VirtualServer resources
ingress2gateway print \
  --input-file ingress.yaml \
  --providers nginx
```

**Step 3: Convert critical Ingresses to VirtualServer CRDs**

```yaml
# Before (ingress-nginx)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 8080
```

```yaml
# After (NIC VirtualServer)
apiVersion: k8s.nginx.org/v1
kind: VirtualServer
metadata:
  name: my-app
spec:
  host: myapp.example.com
  upstreams:
  - name: my-service
    service: my-service
    port: 8080
    readTimeout: 60s
  routes:
  - path: /api
    action:
      pass: my-service
      rewrite:
        path: /
```

**Step 4: Shift traffic (canary)**
Update DNS or LoadBalancer to point to NIC service. Monitor.

**Step 5: Decommission ingress-nginx**
```bash
helm uninstall ingress-nginx -n ingress-nginx
```

---

## Path 2: ingress-nginx → NGF

### Overview
NGF uses the Kubernetes Gateway API. Ingress resources become `HTTPRoute`, `GRPCRoute`, etc. The `ingress2gateway` tool supports conversion.

### Step-by-Step Migration

**Step 1: Install NGF**
```bash
helm repo add nginx-stable https://helm.nginx.com/stable
helm install ngf nginx-stable/nginx-gateway-fabric \
  --namespace nginx-gateway \
  --create-namespace
```

**Step 2: Create GatewayClass and Gateway**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: nginx
spec:
  controllerName: gateway.nginx.org/nginx-gateway-controller
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: prod-gateway
  namespace: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    port: 80
    protocol: HTTP
  - name: https
    port: 443
    protocol: HTTPS
    tls:
      certificateRefs:
      - name: tls-secret
```

**Step 3: Convert Ingress resources to HTTPRoute**
```bash
ingress2gateway print \
  --input-file ingress.yaml \
  --providers nginx \
  --output-format gateway-api
```

```yaml
# Generated HTTPRoute
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: my-app
spec:
  parentRefs:
  - name: prod-gateway
    namespace: nginx-gateway
  hostnames:
  - myapp.example.com
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api
    filters:
    - type: URLRewrite
      urlRewrite:
        path:
          type: ReplacePrefixMatch
          replacePrefixMatch: /
    backendRefs:
    - name: my-service
      port: 8080
```

---

## Path 3: ingress-nginx → CIS

### Overview
CIS uses an external BIG-IP as the data plane. This migration requires BIG-IP hardware or VE. It is a significant architectural change but appropriate for organisations with existing BIG-IP investments.

### Prerequisites
- F5 BIG-IP (hardware appliance or Virtual Edition) — v14.1+
- BIG-IP licensed with LTM module (minimum)
- Network connectivity between Kubernetes nodes and BIG-IP

### Step-by-Step Migration

**Step 1: Deploy CIS**
```bash
helm repo add f5-stable https://f5networks.github.io/charts/stable
helm install cis f5-stable/f5-bigip-ctlr \
  --namespace kube-system \
  --set args.bigip-url=https://192.168.1.245 \
  --set args.bigip-partition=kubernetes \
  --set args.pool-member-type=cluster \
  --set args.namespace=default \
  --set bigip-login.username=admin \
  --set bigip-login.password=<password>
```

**Step 2: Create VirtualServer CRDs**
```yaml
apiVersion: cis.f5.com/v1
kind: VirtualServer
metadata:
  name: my-app
  labels:
    f5cr: "true"
spec:
  host: myapp.example.com
  virtualServerAddress: "192.168.1.100"
  pools:
  - path: /api
    service: my-service
    servicePort: 8080
    rewrite: /
    monitor:
      type: http
      send: "GET / HTTP/1.0\r\n\r\n"
      recv: "200"
```

---

## Feature Gap Analysis

Features in ingress-nginx that require special attention during migration:

| ingress-nginx Feature | NIC Migration | NGF Migration | CIS Migration |
|---|---|---|---|
| `configuration-snippet` | Use Policy CRD or upstreamSettingsPolicy | Not supported by design | Use iRules in Policy CRD |
| `server-snippet` | Use GlobalConfiguration | Not supported | Use iRules |
| Lua custom plugins | Rewrite as NGINX Policy | Use extensionRef filters | Use iRules (TCL) |
| ModSecurity WAF | Use NGINX App Protect | Use F5 WAF (Plus) | Use F5 Advanced WAF |
| Canary annotations | VirtualServer traffic splitting | HTTPRoute weight | VirtualServer pool weights |
| Rate limiting annotations | Policy CRD — rateLimit | RateLimit policy | Policy CRD + BIG-IP rate shaping |
| External auth (`auth-url`) | Policy CRD — oidc/jwt | extensionRef (roadmap) | BIG-IP APM |
| TCP/UDP ConfigMap ingress | TransportServer CRD | On roadmap | TransportServer CRD |
| `proxy-ssl-*` annotations | Policy CRD — egressMTLS | BackendTLSPolicy | TLSProfile CRD |

---

## Rollback Plan

Always run old and new ingress controllers in parallel during migration:

```bash
# Old: ingress-nginx on ingressClass=nginx
# New: NIC/NGF on ingressClass=nginx-new

# Test new ingressClass on non-critical workloads first
# Use weighted DNS or traffic splitting to shift load gradually
# Maintain old controller until all Ingresses are migrated and validated
```

# migration-vers-gateway-api

## 1. Installation du Gateway API CRD

Installez les CRDs Gateway API v1.5.0 :

```bash
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.5.0/standard-install.yaml
```

**Source :** [Gateway API Getting Started](https://gateway-api.sigs.k8s.io/guides/getting-started/#installing-a-gateway-controller)

## 2. Installation des CRDs du provider (NGINX Gateway Fabric)

**Documentation :**
- [NGINX Gateway Fabric Quickstart](https://docs.nginx.com/nginx-gateway-fabric/get-started)
- [Installation Helm](https://docs.nginx.com/nginx-gateway-fabric/install/helm/)

Installation standard :
```bash
helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric \
  --create-namespace -n nginx-gateway
```

**Personnalisation du déploiement (ex: NodePort) :**
```bash
helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric \
  --create-namespace -n nginx-gateway \
  --set nginx.service.type=NodePort
```

## 3. Fonctionnement du Gateway API

![Gateway API Architecture](./images/gatewayapi.png)

## 4. Différence entre Gateway API et API Gateway

**API Gateway** : Outil qui agrège les APIs d'applications pour une gestion centralisée (authentification, rate limiting, etc.). Interface commune pour les consommateurs externes.

**Gateway API** : Ensemble de ressources Kubernetes pour modéliser le networking de services. Ressource principale : `Gateway` qui déclare le type de GatewayClass et sa configuration. Expressive, extensible et orientée rôles.

**Note** : Certains API Gateways peuvent être configurés via Gateway API.

**Architecture NGINX Gateway Fabric :** [Documentation](https://docs.nginx.com/nginx-gateway-fabric/overview/gateway-architecture/)

## 5. Différences Gateway API vs Ingress Controller

Gateway API est plus expressive que l'Ingress avec :
- Routage avancé (HTTPRoute, TCPRoute, etc.)
- Modèle orienté rôles
- Extensible via GatewayClasses



## 6. Installation Istio Gateway API

### Ajout du repo Helm
```bash
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
```

### Installation Istio Base
```bash
helm install istio-base istio/base -n istio-system \
  --create-namespace --wait
```

### Installation Istiod (Control Plane)
```bash
helm install istiod istio/istiod -n istio-system \
  --set profile=ambient --wait
```

### Installation CNI
```bash
helm install istio-cni istio/cni -n istio-system \
  --set profile=ambient --wait
```

### Installation Ztunnel (Layer 4)
```bash
helm install ztunnel istio/ztunnel -n istio-system --wait
```

### Vérification
```bash
helm ls -n istio-system
```

## 7. Test et Validation

**Ancienne approche Istio (Ingress/VirtualService)** : Le trafic était géré par VirtualService.

**Nouvelle approche Gateway API** : Définir `Gateway` + `HTTPRoute`.

**Port-forward pour test :**
```bash
kubectl port-forward svc/hello-gateway-istio 8080:80 --address 0.0.0.0
```

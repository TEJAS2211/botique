# Microservices Kubernetes Deployment with Kustomize

This repository contains a microservices application deployed using Kubernetes and **Kustomize** for configuration management across multiple environments.

---

## 📌 What is Kustomize?

Kustomize is a configuration management tool built into `kubectl` that enables you to customize Kubernetes objects through overlays without modifying the base YAML files.

### 🔑 Key Features

- **Template-Free**: Uses plain YAML without templating syntax
- **Overlay Support**: Manage multiple environments like `dev`, `stg`, `prod`
- **Built-In Support**: Native to `kubectl` since v1.14+
- **Layered Approach**: Supports base + overlay directory pattern
- **Custom Transformers**: Labels, namespaces, annotations, image overrides, etc.

---

## ⚙️ Prerequisites

Before you begin, ensure the following tools are available:

- A running **Kubernetes Cluster**
- `kubectl` configured to access the cluster
- `kustomize` CLI (optional but recommended for advanced features)

### ✅ Installation

#### Option 1: Use kubectl built-in (v1.14+)

```bash
kubectl apply -k ./overlays/dev
```

#### Option 2: Install Latest Kustomize CLI

```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
sudo mv kustomize /usr/local/bin/
kustomize version --short
```

---

## 🗂️ Project Structure

```bash
botique/
├── base/                              # Base configurations for all services
│   ├── ad/                            # Advertisement service
│   │   ├── ad-deploy.yaml             # Deployment configuration
│   │   ├── ad-sa.yaml                 # Service account
│   │   ├── ad-service.yaml            # Service configuration
│   │   └── kustomization.yaml         # Kustomize configuration
│   ├── cart/                          # Shopping cart service
│   │   ├── cart-deploy.yaml
│   │   ├── cart-sa.yaml
│   │   ├── cart-service.yaml
│   │   └── kustomization.yaml
│   ├── checkout/                      # Checkout service
│   │   ├── checkout-deploy.yaml
│   │   ├── checkout-sa.yaml
│   │   ├── checkout-service.yaml
│   │   └── kustomization.yaml
│   ├── currency/                      # Currency conversion service
│   │   ├── currency-deploy.yaml
│   │   ├── currency-sa.yaml
│   │   ├── currency-service.yaml
│   │   └── kustomization.yaml
│   ├── email/                         # Email service
│   │   ├── email-deploy.yaml
│   │   ├── email-sa.yaml
│   │   ├── email-service.yaml
│   │   └── kustomization.yaml
│   ├── frontend/                      # Frontend application
│   │   ├── frontend-deploy.yaml
│   │   ├── frontend-sa.yaml
│   │   ├── frontend-service.yaml
│   │   └── kustomization.yaml
│   ├── loadgenerator/                 # Load testing service
│   │   ├── kustomization.yaml
│   │   ├── loadgenerator-deploy.yaml
│   │   └── loadgenerator-service.yaml
│   ├── payments/                      # Payment processing service
│   │   ├── kustomization.yaml
│   │   ├── payments-deploy.yaml
│   │   ├── payments-sa.yaml
│   │   └── payments-service.yaml
│   ├── productcatalog/                # Product catalog service
│   │   ├── kustomization.yaml
│   │   ├── productcatalog-deploy.yaml
│   │   ├── productcatalog-sa.yaml
│   │   └── productcatalog-service.yaml
│   ├── recommendation/                # Product recommendation service
│   │   ├── kustomization.yaml
│   │   ├── recommendation-deploy.yaml
│   │   ├── recommendation-sa.yaml
│   │   └── recommendation-service.yaml
│   ├── rediscart/                     # Redis cart service
│   │   ├── kustomization.yaml
│   │   ├── rediscart-deploy.yaml
│   │   ├── rediscart-sa.yaml
│   │   └── rediscart-service.yaml
│   ├── shipping/                      # Shipping service
│   │   ├── kustomization.yaml
│   │   ├── shipping-deploy.yaml
│   │   ├── shipping-sa.yaml
│   │   └── shipping-service.yaml
│   └── kustomization.yaml             # Main base kustomization
├── components/                        # Reusable components
│   ├── with-loadgenerator/            # Component with load generator
│   │   ├── kustomization.yaml
│   │   └── loadgenerator.patch.yaml
│   └── without-loadgenerator/         # Component without load generator
│       ├── delete-loadgenerator-sa.patch.yaml
│       ├── delete-loadgenerator.patch.yaml
│       └── kustomization.yaml
└── overlays/                          # Environment-specific configurations
    ├── dev/                           # Development environment
    │   ├── ad-deploy.patch.yaml
    │   ├── cart-deploy.patch.yaml
    │   ├── checkout-deploy.patch.yaml
    │   ├── config-patch.yaml
    │   ├── currency-deploy.patch.yaml
    │   ├── frontend-deploy.patch.yaml
    │   ├── hpa.yaml                   # Horizontal Pod Autoscaler
    │   ├── ingress.yaml               # Ingress configuration
    │   ├── kustomization.yaml
    │   ├── monitoring-patch.yaml
    │   ├── ns.yaml                    # Namespace
    │   ├── pdb.yaml                   # Pod Disruption Budget
    │   ├── productcatalog-deploy.patch.yaml
    │   ├── redis-cart-deploy.patch.yaml
    │   ├── security-patch.yaml
    │   └── shipping-deploy.patch.yaml
    ├── prod/                          # Production environment
    │   ├── [same structure as dev]
    ├── qa/                            # QA environment
    │   ├── [same structure as dev]
    └── staging/                       # Staging environment
        ├── [same structure as dev]
```

---

## 🔨 Core Kustomize Concepts

### 🔁 Transformers

- `commonLabels`
- `commonAnnotations`
- `namespace`
- `namePrefix` / `nameSuffix`
- `images` (newName, newTag)

### 📦 Generators

```yaml
configMapGenerator:
  - name: app-config
    literals:
      - mode=dev

secretGenerator:
  - name: db-secret
    literals:
      - password=admin123
```

### 🔧 Patches

#### Strategic Merge Patch (SMP)

```yaml
patchesStrategicMerge:
  - patch.yaml
```

#### JSON6902 Patch

```yaml
patches:
  - target:
      kind: Deployment
      name: nginx-deployment
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 3
```

---

## 💻 Commands and Usage

### 🔧 Build and Apply

```bash
kustomize build overlays/dev
kubectl apply -k overlays/dev
kustomize build overlays/dev | kubectl apply -f -
```

### 🔍 Preview & Validate

```bash
kubectl diff -k overlays/dev
kustomize build overlays/dev | kubectl apply --dry-run=client -f -
```

### 🩹 Delete

```bash
kubectl delete -k overlays/dev
kustomize build overlays/dev | kubectl delete -f -
```

---

## 🚀 Environment Deployment Workflow

```bash
# Dev Environment
kubectl apply -k overlays/dev

# Staging Environment
kubectl apply -k overlays/staging

# Production
kubectl apply -k overlays/prod
```

---

## 📊 Kustomize vs Helm

### Helm

- Uses Go templating language.
- Offers package management via charts.
- Supports conditionals, loops, and logic.
- Templates are not valid YAML—requires rendering.
- Good for highly dynamic and reusable apps.

### Kustomize

- Uses plain YAML—template-free.
- Overlay system for environment-specific customization.
- No additional templating logic.
- Native to `kubectl`.
- Simpler and more readable for declarative configuration.

### When to Use What

| Use Case                                       | Choose Helm | Choose Kustomize |
| ---------------------------------------------- | ----------- | ---------------- |
| Need package management (charts, dependencies) | ✅           | ❌                |
| Want to keep YAML valid and declarative        | ❌           | ✅                |
| Require logic (e.g., if/else, loops)           | ✅           | ❌                |
| Simple environment overlays (dev/stg/prod)     | ❌           | ✅                |
| Using built-in `kubectl` support               | ❌           | ✅                |

Use **Helm** when you need advanced packaging and templating capabilities. Use **Kustomize** when you want simplicity, readability, and to keep all configurations native and declarative.

---

## 🛠️ Best Practices

1. **Base Configuration** should remain generic and environment-agnostic.
2. Use **overlays** to isolate environment-specific configurations.
3. Prefer **strategic merge patches** for readability and reusability.
4. Validate all configurations with `dry-run` before applying.
5. Use **generators** to dynamically create secrets and configmaps.
6. Maintain **clean folder structure** and consistent naming.
7. Leverage **components** for reusable functionality (e.g., caching, DB).
8. Keep configurations **under version control** and well-documented.

---

## 🧪 Troubleshooting

| Issue                     | Solution                                                                 |
| ------------------------- | ------------------------------------------------------------------------ |
| Missing/invalid fields    | Use `kustomize build` or `kubectl apply --dry-run=client` for validation |
| Image not pulled          | Verify image tag and registry credentials                                |
| ConfigMap/Secret mismatch | Ensure generators are updated properly                                   |
| Incorrect resource output | Check patch precedence and base/overlay conflict                         |

---

## 📚 Additional Resources

- 📘 [Official Kustomize Docs](https://kustomize.io/)
- 📘 [Kubernetes Docs](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/)
- 📘 [Kustomize Best Practices](https://github.com/kubernetes-sigs/kustomize/blob/master/docs/best_practices.md)

---

## ✅ Getting Started (Quick Commands)

```bash
# Apply development configuration
kubectl apply -k overlays/dev

# Validate production deployment
kustomize build overlays/prod | kubectl apply --dry-run=client -f -

# Delete staging setup
kubectl delete -k overlays/staging
```

---

## 📦 CI/CD Integration

```bash
# Set image dynamically using CI/CD pipeline
kustomize edit set image api=my-api:${GITHUB_SHA}

# Commit the patch
git commit -am "Update image to ${GITHUB_SHA}"
git push
```

---


# Roadmap DevOps

Avance y siguientes pasos. Actualizado: 26‑oct‑2026.

## Dónde estoy

| Tema | Estado | Notas |
|---|---|---|
| Linux / WSL | 🟢 Uso diario | [linux](linux/README.md), [entorno](00-entorno-local/README.md) |
| Docker | 🟢 Base completa | [docker](docker/README.md) |
| Kubernetes | 🟡 Fundamentos | [kubernetes](kubernetes/README.md) |
| Helm | 🟡 Charts y templating | [helm](helm/README.md) |
| Istio | 🟡 Traffic management en curso | [istio](istio/README.md) |
| Terraform | 🟢 Curso completo | [terraform](terraform/README.md) |
| AWS | 🟡 IAM / S3 / DynamoDB / EC2 | [aws](aws/README.md) |
| CI/CD, GitOps | 🔴 Pendiente | — |
| Observabilidad | 🔴 Pendiente | — |

## Siguientes pasos (en orden)

### 1. Terminar Istio (ahora)
- [ ] Practicar en local: `virtual-service-reviews-90-10.yaml`, ruteo por header (jason)
- [ ] Fault injection, timeouts, retries, circuit breaker
- [ ] mTLS (`PeerAuthentication` STRICT) y `AuthorizationPolicy`
- [ ] Jaeger y Grafana con Bookinfo

### 2. Completar Kubernetes (el resto depende de esto)
- [ ] Namespaces, ConfigMaps, Secrets
- [ ] Probes, requests/limits, HPA
- [ ] Volúmenes: PV, PVC, StorageClass
- [ ] Ingress / Gateway API
- [ ] RBAC, ServiceAccounts
- [ ] Debugging: `describe`, `logs`, `events`, `exec`
- 🎯 Meta: **CKAD** y después **CKA**

### 3. Helm avanzado
- [ ] Rehacer `~/helm/webapp-nginx` desde cero
- [ ] Dependencias (subcharts), hooks, `helm package` y repos OCI

### 4. GitOps y CI/CD
- [ ] Pipelines: GitHub Actions o Azure DevOps (build → push → deploy)
- [ ] **ArgoCD**: Application apuntando a un chart de Helm en Git
- [ ] Desplegar Bookinfo con ArgoCD en el clúster local

### 5. Terraform + nube
- [ ] Un proyecto real: VPC + EKS/AKS con módulos y remote state
- [ ] Terraform en un pipeline (plan en el PR, apply al hacer merge)
- 🎯 Meta: **Terraform Associate**

### 6. Observabilidad
- [ ] Prometheus + Grafana (kube-prometheus-stack con Helm)
- [ ] Logs (Loki) y trazas (OpenTelemetry)

## Proyecto integrador (para el portafolio)

> App de microservicios → imágenes Docker → chart de Helm → desplegada con ArgoCD en un
> clúster creado con Terraform → con Istio (canary + mTLS) → observada con Prometheus,
> Grafana y Kiali.

Une todo lo que estoy aprendiendo y se puede mostrar en una entrevista.

# Notas DevOps

Mis apuntes de estudio y de práctica, organizados por tema. Cada carpeta tiene un
`README.md` principal y se pueden agregar más archivos dentro (por ejemplo,
`istio/traffic-management.md`).

| Carpeta | Tema |
|---|---|
| [RETOMAR](RETOMAR.md) | **Dónde me quedé y cómo seguir desde otra PC** |
| [ROADMAP](ROADMAP.md) | **Avance y siguientes pasos** |
| [00-entorno-local](00-entorno-local/README.md) | Cómo funciona mi PC de laboratorio: Windows, WSL2, Docker Desktop, kind, minikube y contextos de kubectl |
| [docker](docker/README.md) | run, imágenes, Dockerfile, registry, Compose |
| [kubernetes](kubernetes/README.md) | Arquitectura, Pods, ReplicaSets, Deployments, Services |
| [istio](istio/README.md) | Service mesh con Istio (curso de KodeKloud) |
| [helm](helm/README.md) | Gestor de paquetes de Kubernetes: charts, templating, testing |
| [terraform](terraform/README.md) | Infraestructura como código |
| [aws](aws/README.md) | IAM, S3, DynamoDB, EC2, CLI |
| [linux](linux/README.md) | Comandos y administración de Linux |

## Versiones instaladas (26-oct-2026)

| Herramienta | Versión | Dónde |
|---|---|---|
| Ubuntu (WSL2) | 24.04.1 LTS | Distro de WSL |
| Docker Engine | 29.9.2 | Docker Desktop (Windows) |
| kubectl | v1.36.0 | WSL |
| Kubernetes (docker-desktop) | v1.34.3 | Cluster de Docker Desktop |
| kind | v0.31.0 | `/usr/local/bin/kind` |
| minikube | v1.38.1 | `/usr/local/bin/minikube` (también hay `C:\Kubernetes\minikube.exe`) |
| istioctl / Istio | 1.31.3 | `~/istio-1.31.3` |
| Helm | v4.2.3 | `/usr/sbin/helm` |
| Terraform | v1.15.2 | WSL |

## Convenciones

- Escribir los comandos en bloques de código, listos para copiar.
- Agregar una sección **"Problemas que me pasaron"** en cada tema, con el error y la
  solución.
- Marcar con ⚠️ las **correcciones** o cosas que cambiaron respecto al curso.
- Hacer los diagramas en Mermaid (se ven en VS Code con una extensión de Mermaid y en
  GitHub).

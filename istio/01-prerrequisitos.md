# Prerrequisitos para Istio

## Monolito vs microservicios

- **Monolito**: toda la app en un solo binario o despliegue. Escalar o cambiar una parte
  obliga a redesplegar todo.
- **Microservicios**: cada parte es un servicio independiente que se puede escalar,
  desplegar y escribir en un lenguaje distinto. Lo que se gana en flexibilidad se paga en
  **red**: ahora hay muchas llamadas entre servicios, y cada una necesita:
  - seguridad (cifrado, autenticación entre servicios),
  - reintentos, timeouts y circuit breaking,
  - observabilidad (métricas, logs, trazas),
  - control del tráfico (canary, A/B).

Antes esto se programaba **dentro de cada servicio**, con librerías distintas por lenguaje.
Un service mesh lo saca a la infraestructura.

## Repaso de Kubernetes que hace falta

- **Pods, Deployments y labels** → [kubernetes/02](../kubernetes/02-pods-replicasets-deployments.md)
- **Services**: Istio enruta según los Services de Kubernetes. Cada Service y Deployment
  debe llevar los labels `app` y `version`, que Istio usa para métricas y para los
  *subsets* → [kubernetes/03](../kubernetes/03-networking-services.md)

## Sidecar

Un contenedor **extra** dentro del mismo Pod que agrega funcionalidad sin tocar la app. En
Istio el sidecar es un **proxy Envoy** (contenedor `istio-proxy`) que intercepta **todo**
el tráfico de entrada y salida del Pod mediante reglas de iptables, que configura el init
container `istio-init`.

Por eso los Pods con Istio aparecen como `2/2` en READY: el contenedor de la app y el proxy.

## Envoy

Proxy L7 de alto rendimiento, de código abierto (creado en Lyft y graduado en la CNCF).
Hace:

- balanceo de carga, reintentos, timeouts y circuit breaker,
- TLS / mTLS,
- métricas y trazas,
- enrutamiento por headers, paths y pesos.

Se configura **dinámicamente** por las APIs **xDS** (LDS listeners, RDS routes, CDS
clusters, EDS endpoints). Eso hace `istiod`: le envía la configuración a cada Envoy.

## Service mesh

Es la capa de infraestructura que se encarga de la comunicación entre servicios. Se divide
en:

- **Data plane**: los proxies (Envoy), por donde pasa el tráfico.
- **Control plane**: el componente que configura los proxies (`istiod`).

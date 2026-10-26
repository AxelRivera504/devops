# Arquitectura de Istio

```mermaid
flowchart LR
    USER(["Cliente"]) --> IGW

    subgraph istio-system
        ISTIOD["istiod<br/>(Pilot + Citadel + Galley)<br/>config, certificados mTLS, xDS"]
        IGW["istio-ingressgateway<br/>(Envoy)"]
        EGW["istio-egressgateway<br/>(Envoy)"]
    end

    subgraph default["namespace default (istio-injection=enabled)"]
        subgraph PODA["Pod A"]
            APPA["app"] <--> EA["envoy sidecar"]
        end
        subgraph PODB["Pod B"]
            APPB["app"] <--> EB["envoy sidecar"]
        end
    end

    IGW --> EA
    EA -- "mTLS" --> EB
    EB --> EGW --> EXT(["Servicio externo"])

    ISTIOD -. "xDS config + certs" .-> IGW
    ISTIOD -. "xDS config + certs" .-> EA
    ISTIOD -. "xDS config + certs" .-> EB
    ISTIOD -. "xDS config + certs" .-> EGW
```

## Historia: de varios componentes a istiod

Antes de la versión 1.5 el plano de control eran varios microservicios. Ahora están unidos
en un solo binario, **istiod**:

| Componente antiguo | Función | Hoy |
|---|---|---|
| **Pilot** | Service discovery y envío de configuración a los Envoy por xDS | dentro de istiod |
| **Citadel** | CA: emite y rota certificados para mTLS | dentro de istiod |
| **Galley** | Valida la configuración y la distribuye internamente | dentro de istiod |
| Mixer | Políticas y telemetría | **eliminado** (ahora lo hace Envoy) |

## Planos

- **Control plane** (`istiod`): traduce los recursos de Istio (VirtualService,
  DestinationRule, Gateway…) a configuración de Envoy y la reparte. También emite los
  certificados.
- **Data plane**: los Envoy (sidecars + gateways).
- **Istio agent** (`pilot-agent`): corre en el contenedor `istio-proxy` junto a Envoy. Lo
  arranca, le pasa los certificados (SDS) y hace de intermediario con istiod.

## Inyección del sidecar

```bash
kubectl label namespace default istio-injection=enabled
kubectl get ns -L istio-injection
```

- Un webhook (*mutating admission webhook*) de istiod modifica cada Pod nuevo y le agrega
  `istio-init` e `istio-proxy`.
- Los Pods que ya existían **no** se modifican: hay que hacer `kubectl rollout restart deploy`.
- Para excluir un Pod: anotación `sidecar.istio.io/inject: "false"`.

## Gateways

- **Ingress gateway**: Envoy en el borde, para el tráfico que **entra** al mesh.
- **Egress gateway**: punto de salida controlado hacia servicios externos.

## Modo ambient (para saber que existe)

Es un modo nuevo de Istio **sin sidecars**. Usa `ztunnel` (L4 y mTLS) por nodo y
*waypoint proxies* opcionales para L7. El curso usa el modo sidecar.

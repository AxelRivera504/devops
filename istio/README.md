# Istio

Service mesh. Lo estoy estudiando con el curso de KodeKloud (Istio Service Mesh).
Entorno: ver [00-entorno-local](../00-entorno-local/README.md).

**Estado actual:** Istio **1.31.3** con perfil `demo` instalado en el contexto
`docker-desktop`, con `istio-injection=enabled` en el namespace `default`.

| Archivo | Tema |
|---|---|
| [01-prerrequisitos.md](01-prerrequisitos.md) | Monolito vs microservicios, Services, sidecar, Envoy, qué es un service mesh |
| [02-arquitectura.md](02-arquitectura.md) | istiod (Pilot/Citadel/Galley), data plane, inyección, gateways |
| [03-instalacion-y-bookinfo.md](03-instalacion-y-bookinfo.md) | Instalación, perfiles, Bookinfo, IST0102, Kiali, generar tráfico, simular fallas, limpieza |
| [04-traffic-management-gateways.md](04-traffic-management-gateways.md) | Gateway, VirtualService, DestinationRule, canary, ruteo por header |

## Avance

- [x] Prerrequisitos y arquitectura
- [x] Instalación + Bookinfo + Kiali
- [x] Gateways y acceso por port-forward
- [~] Traffic management (VirtualService / DestinationRule) ← **voy aquí**
- [ ] Fault injection, timeouts, retries, circuit breaker
- [ ] Seguridad (mTLS, AuthorizationPolicy)
- [ ] Observabilidad (Jaeger, Grafana)

## Comandos útiles

```bash
istioctl version
istioctl analyze -A
istioctl proxy-status                      # ¿están sincronizados los Envoy con istiod?
istioctl proxy-config routes <pod>
istioctl proxy-config cluster <pod>
istioctl proxy-config endpoints <pod>
kubectl get vs,dr,gw -A
kubectl logs <pod> -c istio-proxy
kubectl port-forward -n istio-system svc/istio-ingressgateway 8080:80
istioctl dashboard kiali
```

## Problemas que me pasaron

| Síntoma | Causa | Solución |
|---|---|---|
| Pods en `1/1` en vez de `2/2` | Namespace sin la etiqueta o pods creados antes de ponerla | Etiquetar el namespace y correr `kubectl rollout restart deploy -n <ns>` |
| `IST0102` en `istioctl analyze` | Namespace sin `istio-injection=enabled` | Igual que el anterior |
| Gateway `EXTERNAL-IP <pending>` (minikube) | No hay LoadBalancer | `minikube tunnel` |
| No abre la IP del gateway desde Windows | Es una IP de la red interna de Docker | `kubectl port-forward` |
| 503 después de aplicar un VirtualService con subsets | No existe la DestinationRule | Aplicar primero `destination-rule-all.yaml` |
| 404 por el gateway | El `host` del VS no coincide con el header Host | Usar `"*"` o `curl -H "Host: bookinfo.app"` |
| Grafo de Kiali vacío | No hay tráfico | Loop de `curl` |
| `istioctl: command not found` | PATH sin cargar | `source ~/.bashrc` |
| `istioctl verify-install` no existe | Lo quitaron en versiones nuevas | Usar `istioctl x precheck` / `istioctl analyze` |
| Pods `Pending` | Poca RAM | Docker Desktop → Settings → Resources |

> Nota: también tengo `~/istio-1.30.3` de antes. Se puede borrar si ya no lo uso.

# Retomar desde otra PC

Última sesión: **26‑oct‑2026**. Curso: KodeKloud, Istio Service Mesh.

## Dónde me quedé

- ✔ Istio 1.31.3 (perfil `demo`) instalado en `docker-desktop`, con `istio-injection=enabled`
  en `default`.
- ✔ Bookinfo desplegado (6 Pods en `2/2`) y respondiendo dentro del mesh.
- ✔ Entendí la diferencia entre el **componente** gateway (se instala solo) y el
  **recurso** `Gateway` (lo creo yo) → [istio/04](istio/04-traffic-management-gateways.md).
- 🔄 **Práctica en curso: editar el Gateway** (cambiar `hosts` a `bookinfo.app`).
  - Mi copia del YAML: [istio/practicas/bookinfo-gateway.yaml](istio/practicas/bookinfo-gateway.yaml)
  - Estado al cerrar: el `VirtualService` `bookinfo` estaba aplicado, pero el `Gateway`
    **no** → hay que volver a aplicar el archivo completo.

## 1. Preparar la PC nueva

Requisitos: WSL2 + Docker Desktop con Kubernetes activado (ver
[00-entorno-local](00-entorno-local/README.md)), `kubectl` y `git`.

```bash
# Notas
git clone git@github.com:AxelRivera504/notas-devops.git ~/notas-devops

# istioctl (misma versión)
V=1.31.3
curl -sfL -o /tmp/istio-$V.tar.gz \
  https://github.com/istio/istio/releases/download/$V/istio-$V-linux-amd64.tar.gz
tar -xzf /tmp/istio-$V.tar.gz -C ~
mkdir -p ~/.local/bin && ln -sf ~/istio-$V/bin/istioctl ~/.local/bin/istioctl
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
```

## 2. Dejar el clúster como lo tenía

```bash
kubectl config use-context docker-desktop
istioctl x precheck
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled --overwrite

cd ~/istio-1.31.3
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
kubectl wait --for=condition=Ready pod --all --timeout=300s
kubectl get pods                       # todos en 2/2
```

## 3. Seguir con la práctica del Gateway

```bash
cd ~/notas-devops/istio/practicas
kubectl apply -f bookinfo-gateway.yaml
kubectl get gateway,virtualservice
istioctl analyze

# Terminal 1
kubectl port-forward -n istio-system svc/istio-ingressgateway 8080:80
# Terminal 2
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8080/productpage   # 200
```

Luego:
1. Cambiar `"*"` → `"bookinfo.app"` en **los dos** `hosts:` (Gateway y VirtualService).
2. `kubectl apply -f bookinfo-gateway.yaml`
3. Comparar:
   ```bash
   curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8080/productpage                         # 404
   curl -s -o /dev/null -w "%{http_code}\n" -H "Host: bookinfo.app" http://localhost:8080/productpage # 200
   ```
4. Romperlo a propósito (hosts distintos en Gateway y VS) y ver qué dice `istioctl analyze`.
5. Volver a dejar `"*"`.

> El Gateway usa `port: 8080` porque es el puerto del **Pod** Envoy. El Service
> `istio-ingressgateway` recibe en el 80 y lo manda al 8080.

## 4. Lo que sigue después

Archivos en `~/istio-1.31.3/samples/bookinfo/networking/`:

1. `destination-rule-all.yaml` (subsets v1/v2/v3; va **primero**)
2. `virtual-service-all-v1.yaml` → todo a v1
3. `virtual-service-reviews-test-v2.yaml` → login como `jason` ve v2
4. `virtual-service-reviews-90-10.yaml` → `80-20` → `50-v3` (canary)
5. `virtual-service-ratings-test-delay.yaml` / `-test-abort.yaml` (fault injection)
6. Kiali: `kubectl apply -f samples/addons && istioctl dashboard kiali`

Plan general: [ROADMAP.md](ROADMAP.md).

## Al terminar una sesión

```bash
cd ~/notas-devops
git add -A && git commit -m "Avance: <qué hice>" && git push
```

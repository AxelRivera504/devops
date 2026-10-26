# Kubernetes

Fuente: KodeKloud (Kubernetes for Beginners). Mi clúster local está explicado en
[00-entorno-local](../00-entorno-local/README.md).

| Archivo | Tema |
|---|---|
| [01-arquitectura.md](01-arquitectura.md) | Qué es K8s, master/worker, componentes, Docker vs containerd, ctr/nerdctl/crictl |
| [02-pods-replicasets-deployments.md](02-pods-replicasets-deployments.md) | YAML, Pods, ReplicationController, ReplicaSet, labels, Deployments, rollouts |
| [03-networking-services.md](03-networking-services.md) | Red de Pods, CNI, Services (NodePort, ClusterIP, LoadBalancer) |
| [04-microservicios-y-cloud.md](04-microservicios-y-cloud.md) | Práctica de la voting app y Kubernetes en la nube |

## Jerarquía que hay que tener clara

```
Cluster
 └── Node (máquina)
      └── Pod (unidad mínima)
           └── Contenedor(es)

Deployment ──crea──► ReplicaSet ──crea──► Pods
Service ──selecciona con labels──► Pods
```

## Chuleta de comandos

```bash
# Info del clúster
kubectl cluster-info
kubectl get nodes -o wide
kubectl describe node <nodo>

# Pods
kubectl run nginx --image=nginx
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl logs <pod> [-c contenedor] [-f]
kubectl exec -it <pod> -- sh
kubectl delete pod <pod>

# Generar YAML sin crear nada (muy útil en el examen)
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl create deployment web --image=nginx --replicas=3 --dry-run=client -o yaml > deploy.yaml

# Crear / actualizar desde archivos
kubectl create -f archivo.yaml      # crea (falla si ya existe)
kubectl apply  -f archivo.yaml      # crea o actualiza (declarativo)
kubectl apply  -f carpeta/          # todos los YAML de una carpeta
kubectl delete -f archivo.yaml

# ReplicaSets / Deployments
kubectl get rs
kubectl scale --replicas=6 rs/<nombre>
kubectl edit rs <nombre>
kubectl get deploy
kubectl set image deployment/<deploy> <contenedor>=nginx:1.27
kubectl rollout status  deployment/<deploy>
kubectl rollout history deployment/<deploy>
kubectl rollout undo    deployment/<deploy>

# Services
kubectl get svc
kubectl get endpoints <svc>         # ¿el Service encontró Pods?
kubectl port-forward svc/<svc> 8080:80

# Ver todo
kubectl get all
kubectl explain deployment.spec     # documentación de los campos
```

## Objetos vistos

- [x] Pod
- [x] ReplicationController (`~/rc-definition.yaml`)
- [x] ReplicaSet (`~/replicaset-definition.yml`)
- [x] Deployment
- [x] Service (ClusterIP / NodePort / LoadBalancer)
- [ ] Namespace
- [ ] ConfigMap / Secret
- [ ] Ingress
- [ ] Volumes / PV / PVC
- [ ] Probes (liveness / readiness)
- [ ] Resources (requests / limits) y HPA

## Errores típicos

| Error | Significa |
|---|---|
| `ImagePullBackOff` / `ErrImagePull` | No puede bajar la imagen (nombre/tag mal escrito, registry privado sin credenciales) |
| `CrashLoopBackOff` | El contenedor arranca y se cae una y otra vez → revisar `kubectl logs` |
| `Pending` | El scheduler no encuentra un nodo con recursos suficientes |
| Endpoints `<none>` | El `selector` del Service no coincide con los labels de los Pods |
| `no matches for kind "ReplicaSet" in version "v1"` | `apiVersion` equivocado → `apps/v1` |

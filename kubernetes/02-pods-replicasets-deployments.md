# Pods, ReplicaSets y Deployments

## YAML en 30 segundos

```yaml
fruit: apple            # clave: valor
fruits:                 # lista: cada elemento lleva "-"
  - orange
  - apple
banana:                 # diccionario (mapa): propiedades anidadas
  calories: 105
  fat: 0.4
```

La **indentación importa**: los hermanos deben quedar al mismo nivel.

## Los 4 campos raíz de todo objeto

```yaml
apiVersion:   # versión de la API
kind:         # tipo de objeto
metadata:     # nombre, labels (diccionario)
spec:         # lo que quieres que exista (cambia según el kind)
```

| Kind | apiVersion |
|---|---|
| Pod | `v1` |
| Service | `v1` |
| ReplicaSet | `apps/v1` |
| Deployment | `apps/v1` |

---

## Pod

Es el **objeto más pequeño** de Kubernetes: encapsula uno o más contenedores.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:            # es una lista porque puede haber varios contenedores
    - name: nginx-container
      image: nginx
```

```bash
kubectl apply -f pod-definition.yaml
kubectl get pods -o wide        # en qué nodo está y qué IP tiene
kubectl describe pod myapp-pod
```

- **Escalar = crear más Pods**, no agregar contenedores a un Pod existente. Normalmente la
  relación es de un contenedor de la app por Pod.
- **Multi-container pods**: se usan para *helpers*, *sidecars* o *agentes de logs* que
  viven y mueren con la app. Comparten la red (se hablan por `localhost`) y los volúmenes.
- `kubectl run nginx --image=nginx` crea un **Pod**. Para crear un Deployment de forma
  imperativa se usa `kubectl create deployment`.
- `create` vs `apply`: `create` falla si el objeto ya existe; `apply` crea o actualiza. En
  el día a día se usa `apply`.

---

## ReplicationController → ReplicaSet

Los **controllers** son procesos que vigilan objetos y reaccionan. Este en particular:

- Mantiene **N réplicas** siempre vivas: si se cae un Pod, crea otro. Sirve incluso con
  una sola réplica, por alta disponibilidad.
- Reparte la carga entre varios Pods, y entre nodos si hace falta.

**ReplicationController** es la versión antigua; **ReplicaSet** es la actual.

```yaml
apiVersion: apps/v1          # ← si pones v1 da error "no matches for kind ReplicaSet"
kind: ReplicaSet
metadata:
  name: myapp-replicaset
spec:
  replicas: 3
  selector:                  # OBLIGATORIO en ReplicaSet
    matchLabels:
      type: front-end
  template:                  # plantilla del Pod (metadata + spec de un Pod)
    metadata:
      labels:
        type: front-end      # debe coincidir con el selector
    spec:
      containers:
        - name: nginx
          image: nginx
```

### ¿Por qué el `selector` si ya está el template?

Porque el ReplicaSet también **adopta Pods que ya existían** con esos labels. Si ya hay 3
Pods `type: front-end` y pides 3 réplicas, no crea ninguno más. Y si después creas a mano
un Pod con ese label, el ReplicaSet lo elimina porque sobra.

### Labels y selectors

Los labels son etiquetas `clave: valor` que identifican los Pods. Los selectors son filtros
basados en esas etiquetas. Así es como un ReplicaSet (o un Service) sabe **cuáles** de los
cientos de Pods del clúster le pertenecen.

### Escalar

```bash
# 1) editar replicas en el YAML y luego:
kubectl replace -f replicaset-definition.yml
# 2) o de forma imperativa (no modifica el archivo):
kubectl scale --replicas=6 -f replicaset-definition.yml
kubectl scale --replicas=6 replicaset myapp-replicaset
# 3) o editando el objeto vivo:
kubectl edit replicaset myapp-replicaset
```

---

## Deployment

Está **por encima** del ReplicaSet y agrega:

- **Rolling updates**: actualizar la versión sin downtime.
- **Rollback**: volver a la versión anterior.
- **Pausar y reanudar** cambios.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      type: front-end
  template:
    metadata:
      labels:
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
```

```bash
kubectl create deployment web --image=nginx --replicas=3   # imperativo
kubectl get all        # Deployment → ReplicaSet → Pods
```

### Rollouts y versiones

Crear o cambiar la imagen de un Deployment dispara un **rollout**, que crea un ReplicaSet
nuevo y queda registrado como una revisión (Revision 1, 2…).

```bash
kubectl rollout status  deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
kubectl rollout undo    deployment/myapp-deployment   # vuelve a la revisión anterior
```

### Estrategias

| Estrategia | Cómo | Downtime |
|---|---|---|
| **Recreate** | Baja todos los Pods viejos y luego sube los nuevos. En los eventos se ve el ReplicaSet viejo bajar a 0 y el nuevo subir a 5. | Sí |
| **RollingUpdate** (por defecto) | Va reemplazando de a poco: baja uno y sube uno. | No |

### Cómo actualizar

```bash
kubectl apply -f deployment.yaml                                   # recomendado: el YAML queda como fuente de verdad
kubectl set image deployment/myapp-deployment nginx-container=nginx:1.9.1   # rápido, pero el YAML queda desactualizado
```

Por dentro, al actualizar se crea **ReplicaSet‑2** que va subiendo Pods mientras
**ReplicaSet‑1** baja a 0. El viejo se conserva justamente para poder hacer
`rollout undo`.

### Trucos de Linux que usé

```bash
cat -n deployment-2.yaml | sed -n '15,22p'   # ver solo las líneas 15 a 22
```

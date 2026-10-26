# Testing y debug de charts

De menor a mayor profundidad:

| Paso | Comando | Qué detecta |
|---|---|---|
| 1. Lint | `helm lint ./chart` | Errores de sintaxis en templates/YAML, campos que faltan en Chart.yaml, buenas prácticas |
| 2. Template | `helm template rel ./chart` | Muestra el YAML final **sin** contactar al clúster. Errores de indentación y valores mal referenciados. |
| 3. Template debug | `helm template rel ./chart --debug` | Muestra el YAML aunque sea inválido, para ver dónde se rompe |
| 4. Dry run | `helm install rel ./chart --dry-run=server --debug` | Valida contra el **API server**: detecta un `kind`/`apiVersion` inválido o un campo que no existe en el spec (por ejemplo, `container:` en lugar de `containers:`) |
| 5. helm test | `helm test rel` | Ejecuta Pods de prueba sobre el release ya instalado |

`lint` y `template` no detectan errores de **Kubernetes**, como un campo mal escrito en el
spec. Para eso hace falta el paso 4 (o `helm template ... | kubectl apply --dry-run=server -f -`).

## Renderizar solo un archivo

```bash
helm template rel ./chart -s templates/deployment.yaml
helm template rel ./chart --set replicaCount=5 -f values-prod.yaml
```

## helm test

Los tests son Pods (normalmente Jobs) en `templates/tests/` con la anotación `test`. Pasan
si el contenedor termina con código 0.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ .Release.Name }}-test-connection"
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['{{ .Release.Name }}-nginx:{{ .Values.service.port }}']
  restartPolicy: Never
```

```bash
helm install web1 ./webapp-nginx
helm test web1
```

## Instalar de forma segura

```bash
# Helm 4: si falla, regresa solo a la revisión anterior
helm upgrade --install web1 ./webapp-nginx --rollback-on-failure --wait --timeout 5m
```

> ⚠️ Helm 4: `--atomic` ahora se llama `--rollback-on-failure`. Si sigues un tutorial con
> `--atomic` y falla, es por eso.

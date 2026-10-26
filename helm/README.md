# Helm

Gestor de paquetes de Kubernetes. Fuente: KodeKloud (Helm for Beginners).

Versión instalada: **v4.2.3** (`/usr/sbin/helm`). Las prácticas están en `~/helm`
(`~/helm/webapp-nginx`) y en `~/example1`.

Helm usa el mismo kubeconfig y el mismo contexto que `kubectl`, así que revisa siempre a qué
cluster apuntas (`kubectl config current-context`).

| Archivo | Tema |
|---|---|
| [01-conceptos-y-comandos.md](01-conceptos-y-comandos.md) | Qué es Helm, chart/repo/release, lifecycle, upgrade/rollback |
| [02-crear-charts-templating.md](02-crear-charts-templating.md) | Estructura de un chart, Chart.yaml, values, objetos built-in, práctica webapp-nginx |
| [03-funciones-pipelines.md](03-funciones-pipelines.md) | Funciones (`upper`, `quote`, `default`…) y pipelines |
| [04-control-de-flujo-y-scope.md](04-control-de-flujo-y-scope.md) | `if/else`, `with`, `range`, `$`, espacios en blanco |
| [05-testing-y-debug.md](05-testing-y-debug.md) | `lint`, `template`, `--dry-run`, `helm test` |

## Chuleta

```bash
# Repos
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo list
helm repo update
helm search hub wordpress          # busca en Artifact Hub
helm search repo wordpress         # busca en los repos agregados

# Instalar / ver
helm install <release> <repo/chart> -n <ns> --create-namespace
helm install <release> ./mi-chart --set replicaCount=3 -f mis-values.yaml
helm list -A
helm status <release>
helm get values <release>          # valores que le pasé
helm get manifest <release>        # YAML que se aplicó

# Actualizar / volver atrás
helm upgrade <release> <chart> --version 1.2.3
helm upgrade --install <release> <chart> -f values.yaml   # idempotente (CI/CD)
helm history <release>
helm rollback <release> <revisión>
helm uninstall <release> -n <ns>

# Desarrollar charts
helm create mi-chart
helm lint mi-chart/
helm template mi-release mi-chart/ | less
helm install mi-release mi-chart/ --dry-run=server --debug
helm pull bitnami/wordpress --untar   # bajar el chart para leerlo/modificarlo
helm test <release>
```

## Helm 4 vs lo que enseña el curso

El curso usa Helm 3. En Helm 4 cambió esto:

| Helm 3 | Helm 4 |
|---|---|
| `--atomic` | `--rollback-on-failure` |
| `--wait` (bool) | `--wait` sigue existiendo y además acepta estrategias: `watcher` (default), `hookOnly`, `legacy` |
| `--dry-run` | Usar `--dry-run=client` o `--dry-run=server` de forma explícita |

## Problemas que me pasaron

| Síntoma | Causa | Solución |
|---|---|---|
| `Error: INSTALLATION FAILED: cannot re-use a name that is still in use` | El release ya existe | `helm upgrade --install` o `helm uninstall` |
| `nil pointer evaluating interface {}.xxx` | El valor no existe en `values.yaml` (typo o nivel mal indentado) | Revisar values / usar `default` o `if` |
| `unknown flag: --atomic` | Helm 4 | `--rollback-on-failure` |
| YAML inválido al renderizar | Indentación del template | `helm template` + `nindent` |

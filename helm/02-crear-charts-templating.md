# Crear charts y templating

## Estructura de un chart

```bash
helm create webapp-nginx
```

```
webapp-nginx/
├── Chart.yaml          # metadata del chart
├── values.yaml         # valores por defecto
├── charts/             # dependencias (subcharts)
├── templates/          # YAML con plantillas Go
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── _helpers.tpl    # plantillas con nombre (no generan objetos)
│   ├── NOTES.txt       # mensaje que se muestra al instalar
│   └── tests/
├── README.md
└── LICENSE
```

## Chart.yaml

```yaml
apiVersion: v2              # Helm 3+
name: webapp-nginx
description: Mi primer chart
type: application           # o "library" (solo helpers, no se instala)
version: 0.1.0              # versión del CHART
appVersion: "1.27.0"        # versión de la APP que empaqueta (informativo)
dependencies:
  - name: mariadb
    version: 11.x.x
    repository: https://charts.bitnami.com/bitnami
keywords: [web, nginx]
maintainers:
  - name: Axel
```

**`version` vs `appVersion`**: si cambias el chart (templates, values), subes `version`.
`appVersion` es la versión del software (nginx 1.27). Son independientes.

## Templating

Helm usa el lenguaje de plantillas de Go. Todo lo que está entre `{{ }}` se evalúa.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-nginx
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: nginx
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

```yaml
# values.yaml
replicaCount: 2
image:
  repository: nginx
  tag: "1.27"
```

El `.` del inicio es el **scope raíz**. Los nombres de los objetos built-in empiezan con
mayúscula y lo que defines tú en values puede ir en minúscula.

### Objetos built-in

| Objeto | Ejemplos |
|---|---|
| `.Release` | `.Release.Name`, `.Release.Namespace`, `.Release.Revision`, `.Release.IsUpgrade`, `.Release.IsInstall`, `.Release.Service` |
| `.Chart` | `.Chart.Name`, `.Chart.Version`, `.Chart.AppVersion` (campos de `Chart.yaml`) |
| `.Capabilities` | `.Capabilities.KubeVersion`, `.Capabilities.APIVersions.Has "apps/v1"` |
| `.Values` | Lo de `values.yaml` + `-f` + `--set` |
| `.Template` | `.Template.Name`, `.Template.BasePath` |

### ¿Por qué `{{ .Release.Name }}` en los nombres?

Para poder instalar el mismo chart varias veces en el mismo namespace sin que choquen los
nombres de los objetos.

### Precedencia de valores (de menor a mayor)

1. `values.yaml` del chart
2. `-f mis-values.yaml` (si hay varios, gana el último)
3. `--set clave=valor`

## Práctica: webapp-nginx

Carpeta: `~/helm/webapp-nginx` (ahora mismo está vacía; hay que rehacerla).

```bash
cd ~/helm
helm create webapp-nginx
# limpiar templates que no se usan
rm -rf webapp-nginx/templates/{hpa.yaml,ingress.yaml,serviceaccount.yaml,httproute.yaml,tests}
# editar deployment.yaml, service.yaml y values.yaml
helm lint webapp-nginx/
helm template web1 webapp-nginx/
helm install web1 webapp-nginx/
helm install web2 webapp-nginx/ --set replicaCount=3   # mismo chart, otro release
kubectl get deploy,svc
helm uninstall web1 web2
```

Errores que salen al hacerlo:

- Borrar `serviceaccount.yaml` pero dejar `serviceAccountName: {{ include ... }}` en el
  deployment → error en el template. Hay que quitar también esa referencia (y la de
  `_helpers.tpl` si ya no se usa).
- Escribir `.values.replicaCount` en minúscula → nil pointer. Es `.Values`.
- `image: {{ .Values.image.repository }}:{{ .Values.image.tag }}` sin comillas cuando el tag
  parece número (`1.20`) → YAML lo interpreta como float. Usar comillas o `quote`.

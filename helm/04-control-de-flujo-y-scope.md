# Control de flujo y scope

## Espacios en blanco: `{{-` y `-}}`

- `{{-` borra los espacios y saltos de línea **antes**.
- `-}}` borra los de **después**.

Sin el guion quedan líneas vacías en el YAML generado. Si quitas demasiados, se pegan las
líneas. Revísalo siempre con `helm template`.

## if / else if / else

```yaml
metadata:
  name: {{ .Release.Name }}-nginx
  {{- if .Values.orgLabel }}
  labels:
    org: {{ .Values.orgLabel }}
  {{- else if eq .Values.env "prod" }}
  labels:
    org: produccion
  {{- else }}
  labels:
    org: default
  {{- end }}
```

Se considera **falso**: `false`, `0`, `""`, `nil`, lista vacía y mapa vacío.

Uso típico: crear un objeto solo si está habilitado.

```yaml
{{- if .Values.serviceAccount.create }}
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ .Release.Name }}-robot-sa
{{- end }}
```

Comparaciones: `eq`, `ne`, `lt`, `le`, `gt`, `ge`, `and`, `or`, `not`. Se escriben como
funciones: `{{ if and .Values.a (eq .Values.env "prod") }}`.

## with: cambiar el scope

```yaml
# values.yaml
app:
  ui:
    bg: red
    fg: black
  db:
    name: users
    conn: mongodb://localhost:27020/mydb
```

Sin `with` hay que escribir rutas largas: `.Values.app.ui.bg`.

```yaml
data:
  {{- with .Values.app }}
    {{- with .ui }}
  background: {{ .bg }}
  foreground: {{ .fg }}
    {{- end }}
    {{- with .db }}
  database: {{ .name }}
  connection: {{ .conn }}
    {{- end }}
  release: {{ $.Release.Name }}      # ← con . sería .Values.app.Release → error
  {{- end }}
```

Dentro de un `with` (o de un `range`), `.` apunta al objeto actual. Para volver a la raíz
se usa **`$`**: `$.Release.Name`, `$.Values.x`.

Además, `with` funciona como un `if`: si el valor está vacío, el bloque no se genera.

## range: loops

```yaml
# values.yaml
regions:
  - ohio
  - newyork
  - ontario
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-regioninfo
data:
  regions:
  {{- range .Values.regions }}
  - {{ . | quote }}
  {{- end }}
```

Resultado:

```yaml
  regions:
  - "ohio"
  - "newyork"
  - "ontario"
```

Con índice o con mapas:

```yaml
{{- range $i, $region := .Values.regions }}
  region-{{ $i }}: {{ $region }}
{{- end }}

{{- range $key, $val := .Values.labels }}
  {{ $key }}: {{ $val | quote }}
{{- end }}
```

## Variables

```yaml
{{- $relname := .Release.Name -}}
{{- with .Values.app }}
release: {{ $relname }}
{{- end }}
```

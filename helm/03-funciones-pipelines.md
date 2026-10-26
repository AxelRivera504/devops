# Funciones y pipelines

## Funciones

Sintaxis: `{{ funcion argumento1 argumento2 }}`

```yaml
image: {{ upper .Values.image.repository }}          # NGINX
image: {{ quote .Values.image.repository }}          # "nginx"
image: {{ replace "x" "y" .Values.image.repository }}
image: {{ default "nginx" .Values.image.repository }}   # si está vacío usa "nginx"
```

| Tipo | Funciones |
|---|---|
| String | `upper`, `lower`, `title`, `trim`, `quote`, `squote`, `replace`, `trunc 63`, `trimSuffix "-"`, `contains`, `hasPrefix` |
| Defaults | `default`, `required "mensaje" .Values.x` (falla si no existe), `empty`, `coalesce` |
| Indentación | `indent N`, `nindent N` (nueva línea + N espacios) |
| YAML / JSON | `toYaml`, `toJson`, `fromYaml` |
| Listas / dicts | `list`, `dict`, `get`, `hasKey`, `keys`, `first`, `last`, `has` |
| Lógica / comparación | `eq`, `ne`, `lt`, `gt`, `and`, `or`, `not` |
| Otros | `include`, `tpl`, `b64enc`, `sha256sum`, `now`, `randAlphaNum` |

Son funciones de Go templates + la librería **Sprig**. Referencia:
<https://helm.sh/docs/chart_template_guide/function_list/>

## Pipelines

Igual que `|` en Linux: la salida de una función entra como **último argumento** de la
siguiente.

```yaml
image: {{ .Values.image.repository | upper | quote }}           # "NGINX"
image: {{ .Values.image.repository | default "nginx" | quote }}
```

## El patrón más común: `toYaml | nindent`

```yaml
# values.yaml
resources:
  limits:
    cpu: 100m
    memory: 128Mi

# templates/deployment.yaml
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

`nindent 12` agrega un salto de línea y 12 espacios a **cada línea**, para que el bloque
quede bien anidado.

## Plantillas con nombre (`_helpers.tpl`)

Los archivos que empiezan con `_` no generan objetos. Ahí se definen bloques reutilizables:

```yaml
{{- define "webapp.labels" -}}
app.kubernetes.io/name: {{ .Chart.Name }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

```yaml
metadata:
  labels:
    {{- include "webapp.labels" . | nindent 4 }}
```

- Se usa `include` (no `template`) porque devuelve un string que se puede pasar por un
  pipeline (`| nindent 4`).
- El `.` al final pasa el scope; sin él, dentro del helper `.Release` sería nil.

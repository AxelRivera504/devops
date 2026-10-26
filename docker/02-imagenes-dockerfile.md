# Imágenes y Dockerfile

## ¿Cómo se crea una imagen propia?

1. Anota en orden los pasos que harías para desplegar la app **a mano**.
   Ejemplo, una web en Python con Flask:
   1. Sistema operativo: Ubuntu
   2. `apt update`
   3. Instalar dependencias con `apt`
   4. Instalar dependencias de Python con `pip`
   5. Copiar el código a `/opt`
   6. Arrancar el servidor con `flask run`
2. Convierte esos pasos en un **Dockerfile**.
3. Construye la imagen con `docker build`.
4. Publícala con `docker push`.

```dockerfile
FROM ubuntu

RUN apt-get update && apt-get install -y python3 python3-pip
RUN pip install flask flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

```bash
docker build -t usuario/my-custom-app .   # el "." es el contexto (donde está el Dockerfile)
docker login
docker push usuario/my-custom-app
```

> ⚠️ Corrección respecto a la diapositiva del curso: la forma correcta es
> `docker build -t nombre .`. El curso muestra `docker build Dockerfile -t …`, pero a
> `docker build` se le pasa la **carpeta** de contexto, no el archivo. Si el archivo no se
> llama `Dockerfile`, se indica con `-f`.

## Formato: INSTRUCCIÓN argumento

| Instrucción | Qué hace |
|---|---|
| `FROM` | Imagen base. **Todo Dockerfile empieza con FROM.** |
| `RUN` | Ejecuta un comando durante la construcción (instalar paquetes, etc.) |
| `COPY` | Copia archivos del host a la imagen |
| `ENV` | Define variables de entorno |
| `WORKDIR` | Directorio de trabajo |
| `EXPOSE` | Documenta el puerto que usa la app |
| `CMD` | Comando por defecto al arrancar (se puede sobrescribir) |
| `ENTRYPOINT` | Ejecutable fijo al arrancar (los argumentos se le agregan) |

## Arquitectura en capas

**Cada instrucción crea una capa** que contiene solo los cambios respecto a la anterior:

```
Capa 1: Ubuntu base              120 MB
Capa 2: paquetes apt             306 MB
Capa 3: paquetes pip             6.3 MB
Capa 4: código fuente            229 B
Capa 5: ENTRYPOINT               0 B
```

- `docker history <imagen>` muestra las capas y lo que pesa cada una.
- **Caché de capas**: si falla el paso 3 y lo corriges, el siguiente build reutiliza las
  capas 1 y 2. Por eso conviene poner primero lo que cambia poco (dependencias) y al final
  lo que cambia mucho (el código).

## Variables de entorno

En lugar de dejar valores fijos en el código (por ejemplo, el color de fondo de la app), se
leen de una variable de entorno:

```python
color = os.environ.get('APP_COLOR')
```

```bash
docker run -e APP_COLOR=blue  simple-webapp-color
docker run -e APP_COLOR=green simple-webapp-color
docker inspect <contenedor>   # en "Config" → "Env" se ven las variables
```

## CMD vs ENTRYPOINT

`docker run ubuntu` arranca y se detiene enseguida. La imagen usa `CMD ["bash"]`, y bash
sin terminal termina al instante. Un contenedor **no es una VM**: está hecho para correr un
proceso y vive solo mientras ese proceso vive.

```dockerfile
# Dos formas de escribir CMD
CMD sleep 5               # forma shell
CMD ["sleep", "5"]        # forma JSON (exec): primero el ejecutable, luego los argumentos
```

| | CMD | ENTRYPOINT |
|---|---|---|
| `docker run img sleep 10` | **Reemplaza** todo el CMD | — |
| `docker run img 10` | — | Se **agrega** al final: `sleep 10` |

Patrón recomendado, que combina ambos para tener un valor por defecto:

```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

```bash
docker run ubuntu-sleeper        # → sleep 5
docker run ubuntu-sleeper 10     # → sleep 10
docker run --entrypoint sleep2.0 ubuntu-sleeper 10   # → sleep2.0 10
```

Si solo hay ENTRYPOINT y no pasas argumentos, falla con `sleep: missing operand`.

## Editar con nano

`nano -A archivo` abre el archivo. `Ctrl+O` y luego Enter guardan; `Ctrl+X` sale.

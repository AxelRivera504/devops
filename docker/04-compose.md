# Docker Compose

Cuando una aplicación tiene varios servicios (web, base de datos, cache…), en lugar de
correr varios `docker run` a mano se describen todos en un archivo YAML
(`docker-compose.yml` o `compose.yaml`) y se levantan juntos.

```bash
# Sin Compose
docker run mmumshad/simple-webapp
docker run mongodb
docker run redis:alpine
docker run ansible
```

```yaml
# compose.yaml
services:
  web:
    image: "mmumshad/simple-webapp"
    ports:
      - "8080:5000"
  database:
    image: "mongodb"
  messaging:
    image: "redis:alpine"
  orchestration:
    image: "ansible"
```

```bash
docker compose up -d      # levanta todo el stack
docker compose ps
docker compose logs -f
docker compose down       # detiene y borra el stack
```

- Todos los cambios quedan guardados en el archivo, así que el stack es fácil de
  versionar, repetir y mantener.
- Compose trabaja sobre **un solo host Docker**. Para orquestar contenedores en varios
  nodos se usa Kubernetes.

> ⚠️ Corrección: en las notas decía "al final solo tendremos que correr un simple
> docker run". El comando correcto es `docker compose up`. Además, el comando moderno es
> `docker compose`, con espacio; `docker-compose`, con guion, es la versión antigua.

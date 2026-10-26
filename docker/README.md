# Docker

Fuente: curso de KodeKloud (Docker for Beginners). En mi PC, Docker corre a través de
Docker Desktop ([ver entorno local](../00-entorno-local/README.md)).

| Archivo | Tema |
|---|---|
| [01-docker-run.md](01-docker-run.md) | `docker run`, tags y modo interactivo (`-i`, `-t`) |
| [02-imagenes-dockerfile.md](02-imagenes-dockerfile.md) | Dockerfile, capas, variables de entorno, CMD vs ENTRYPOINT |
| [03-registry.md](03-registry.md) | Docker Hub, registries privados y registry propio |
| [04-compose.md](04-compose.md) | Docker Compose |
| [05-docker-windows-mac.md](05-docker-windows-mac.md) | Cómo funciona Docker en Windows y Mac |

## Chuleta rápida

```bash
docker run nginx                     # crea y arranca un contenedor
docker run -d --name web -p 8080:80 nginx   # en segundo plano, puerto host:contenedor
docker run -it ubuntu bash           # interactivo con terminal
docker run -e APP_COLOR=blue app     # variable de entorno
docker ps        /  docker ps -a     # contenedores corriendo / todos
docker stop web  /  docker rm web    # detener / borrar
docker images    /  docker rmi nginx # imágenes / borrar imagen
docker logs -f web
docker exec -it web sh               # entrar a un contenedor que ya corre
docker inspect web                   # todo el detalle (env, red, mounts…)
docker build -t usuario/app:1.0 .    # construir imagen
docker history usuario/app:1.0       # capas y tamaños
docker login && docker push usuario/app:1.0
docker compose up -d / docker compose down
```

## Ideas clave

- Un contenedor **vive lo que vive su proceso principal**. Si el proceso termina, el
  contenedor queda `Exited`.
- Una imagen es una **plantilla de solo lectura hecha de capas**. El contenedor es una
  instancia en ejecución de esa imagen.
- Docker comparte el kernel del host, así que un contenedor Linux necesita un kernel Linux.
  Por eso en Windows hay una VM de Linux por debajo (WSL2).

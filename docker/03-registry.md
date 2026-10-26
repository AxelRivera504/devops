# Docker Registry

Un registry es el **repositorio central donde se guardan las imágenes**. Si los
contenedores fueran la lluvia, el registry serían las nubes.

## Anatomía del nombre de una imagen

```
docker.io / library / nginx : 1.27
   │          │        │       └─ tag
   │          │        └─ imagen / repositorio
   │          └─ usuario u organización ("library" = imágenes oficiales)
   └─ registry (si no lo pones, es Docker Hub: docker.io)
```

- `nginx` en realidad es `docker.io/library/nginx:latest`.
- Otros registries: `gcr.io` (Google), `registry.k8s.io` (imágenes de Kubernetes), ECR
  (AWS), ACR (Azure).

## Registry privado

```bash
docker login private-registry.io
docker run private-registry.io/apps/internal-app
```

Si no inicias sesión, Docker dice que la imagen no existe. Siempre hay que hacer login
antes de `pull` o `push` contra un registry privado.

## Montar tu propio registry

El registry también es una imagen (`registry:2`) y expone su API en el puerto 5000.

```bash
docker run -d -p 5000:5000 --name my-registry --restart=always registry:2

docker image tag my-image localhost:5000/my-image   # renombrar apuntando al registry
docker push localhost:5000/my-image
docker pull localhost:5000/my-image                 # desde el mismo host
docker pull 192.168.56.100:5000/my-image            # desde otro host de la red
```

> En las notas originales los flags aparecían como `–name` y `–restart`, con un guion
> largo, porque el procesador de texto los autocorrigió. En la terminal siempre van con
> **dos guiones** (`--`).

# docker run

## Tags (versiones)

```bash
docker run redis          # sin tag → usa "latest"
docker run redis:4.0      # versión específica (tag)
```

Lo que va después de `:` es el **tag**. Si no lo pones, Docker usa `latest`.

> Buena práctica: en producción, fija siempre un tag concreto. `latest` cambia con el
> tiempo y puede romper despliegues.

## Entrada estándar: `-i` y `-t`

Ejemplo: una app que pide tu nombre y luego te saluda.

| Comando | Resultado |
|---|---|
| `docker run app` | No espera la entrada. Imprime el saludo sin nombre y termina. |
| `docker run -i app` | Lee lo que escribes (STDIN), pero no muestra el prompt "Enter your name:". |
| `docker run -it app` | Modo interactivo **y** terminal: funciona igual que correrla localmente. |

- `-i` (**interactive**): conecta la entrada estándar del host con el contenedor.
- `-t` (**tty**): le asigna una pseudo‑terminal, que es lo que hace que veas el prompt.

## Otros flags que se usan siempre

| Flag | Para qué |
|---|---|
| `-d` | Correr en segundo plano (detached) |
| `-p 8080:80` | Publicar el puerto 80 del contenedor en el 8080 del host |
| `--name web` | Ponerle nombre al contenedor |
| `-e VAR=valor` | Variable de entorno |
| `-v /ruta/host:/ruta/contenedor` | Montar un volumen |
| `--rm` | Borrar el contenedor cuando termine |
| `--restart=always` | Reiniciarlo siempre que se caiga |
| `--entrypoint cmd` | Sobrescribir el ENTRYPOINT de la imagen |

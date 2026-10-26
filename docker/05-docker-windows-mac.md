# Docker en Windows y Mac

Docker **comparte el kernel del host**. Un contenedor Linux necesita un kernel Linux, así que
en Windows y en Mac siempre hay una VM de Linux por debajo.

## Windows

| Opción | Cómo funciona | Estado |
|---|---|---|
| **Docker Toolbox** | VirtualBox + una VM ligera (Boot2Docker) con Docker. Trae Docker Engine, Docker Machine, Compose y Kitematic. | Obsoleta, era para Windows viejos |
| **Docker Desktop** | Usa la virtualización nativa: antes Hyper‑V, **hoy WSL2** | La actual. Es la que uso yo |

- Por defecto, Docker Desktop corre **contenedores Linux**. Puede cambiarse a
  **contenedores Windows** desde el menú "Switch to Windows containers".
- Hay dos tipos de contenedores Windows:
  - **Windows Server containers**: comparten el kernel del host, igual que en Linux.
  - **Hyper‑V isolation**: cada contenedor corre dentro de una VM mínima con su propio
    kernel, lo que da más aislamiento.
- Imágenes base de Windows: **Windows Server Core** (pesada) y **Nano Server** (ligera,
  algo así como el "Alpine" de Windows).
- VirtualBox y Hyper‑V tradicionalmente no podían convivir en el mismo host.

## Mac

| Opción | Cómo funciona |
|---|---|
| Docker Toolbox | VirtualBox + Boot2Docker (obsoleto) |
| Docker Desktop | VM con el framework de virtualización nativo de macOS (antes HyperKit) |

No existen "contenedores Mac": en Mac solo se corren contenedores Linux.

## Mi caso

Docker Desktop en Windows con WSL2. La explicación completa está en
[00-entorno-local](../00-entorno-local/README.md).

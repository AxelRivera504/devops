# Linux

Mi distro: **Ubuntu 24.04.1 LTS** en WSL2, con systemd activado. Para ver cómo encaja con
Windows: [00-entorno-local](../00-entorno-local/README.md).

## Comandos útiles

```bash
# Sistema
cat /etc/os-release
uname -a
free -h ; df -h ; nproc
systemctl status <servicio>
journalctl -u <servicio> -f

# Procesos y red
ps aux | grep <algo>
ss -tulpn                       # puertos escuchando
curl -v http://localhost:8080

# Archivos
find . -name "*.yaml"
grep -rn "texto" .
chmod +x script.sh
ln -sf <origen> <enlace>        # enlace simbólico (así puse istioctl en el PATH)

# PATH
echo $PATH
which <comando>
source ~/.bashrc
```

## Problemas que me pasaron

_(ir agregando)_

# Provisioners, taint, debug e import

## Provisioners

Ejecutan comandos o scripts en la máquina local o en el recurso remoto.

```hcl
resource "aws_instance" "webserver" {
  ami           = "ami-0edab43b6fa892279"
  instance_type = "t2.micro"
  key_name      = aws_key_pair.web.key_name
  vpc_security_group_ids = [aws_security_group.ssh-access.id]

  provisioner "remote-exec" {
    inline = [
      "sudo apt update",
      "sudo apt install -y nginx",
      "sudo systemctl enable --now nginx",
    ]
  }

  connection {
    type        = "ssh"
    host        = self.public_ip
    user        = "ubuntu"
    private_key = file("/root/.ssh/web")
  }

  provisioner "local-exec" {
    command = "echo ${self.public_ip} >> /tmp/ips.txt"
  }

  provisioner "local-exec" {
    when       = destroy                       # destroy-time provisioner
    command    = "echo 'Instancia destruida' >> /tmp/state.txt"
  }
}
```

| Opción | Efecto |
|---|---|
| `remote-exec` | Ejecuta en el recurso (necesita `connection`: SSH o WinRM) |
| `local-exec` | Ejecuta en la máquina donde corre Terraform |
| `when = destroy` | Se ejecuta al destruir el recurso, no al crearlo |
| `on_failure = fail` (default) | Si falla, el apply falla y el recurso queda **tainted** |
| `on_failure = continue` | Ignora el error |

**Buena práctica:** usar provisioners **lo menos posible** (último recurso). Se prefieren
las opciones nativas:
- `user_data` en EC2 (o `custom_data` en Azure) para el script de arranque,
- imágenes ya preparadas con **Packer**,
- herramientas de configuración como Ansible.

```hcl
resource "aws_instance" "webserver" {
  ...
  user_data = <<-EOF
    #!/bin/bash
    apt update && apt install -y nginx
    systemctl enable --now nginx
  EOF
}
```

## Taint → `-replace`

Cuando un recurso queda mal (falla un provisioner o alguien lo modificó a mano), se puede
forzar que se recree:

```bash
# Antes:
terraform taint aws_instance.webserver
terraform untaint aws_instance.webserver
# Ahora (recomendado, taint está deprecado):
terraform apply -replace="aws_instance.webserver"
```

## Debug

```bash
export TF_LOG=TRACE           # TRACE, DEBUG, INFO, WARN, ERROR
export TF_LOG_PATH=/tmp/terraform.log
terraform plan
unset TF_LOG TF_LOG_PATH
```

`TF_LOG_CORE` y `TF_LOG_PROVIDER` sirven para separar los logs de Terraform y los del provider.

## Import

Para que Terraform administre un recurso que ya existe (creado a mano).

### Forma clásica (CLI)

```hcl
# 1) escribir el bloque del recurso (vacío o con lo mínimo)
resource "aws_instance" "webserver-2" {
  # ...
}
```

```bash
# 2) importar
terraform import aws_instance.webserver-2 i-026e13be10d5326f7
# 3) ajustar el .tf hasta que `terraform plan` diga "No changes"
terraform state show aws_instance.webserver-2
```

### Forma moderna (Terraform 1.5+): bloque `import`

```hcl
import {
  to = aws_instance.webserver-2
  id = "i-026e13be10d5326f7"
}
```

```bash
terraform plan -generate-config-out=generated.tf   # genera el código del recurso automáticamente
terraform apply
```

Así queda **en código** y se puede revisar en un PR, y no hace falta escribir el recurso a
mano.

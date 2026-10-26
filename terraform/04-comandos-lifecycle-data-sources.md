# Comandos, lifecycle y data sources

## Comandos

```bash
terraform validate                   # sintaxis y referencias
terraform fmt [-recursive]           # formato canónico
terraform show [-json]               # state actual de forma legible
terraform providers                  # providers que usa la configuración
terraform providers mirror /ruta     # copiar providers (entornos sin internet)
terraform output
terraform apply -refresh-only        # actualiza el state con la realidad sin cambiar infraestructura
terraform graph | dot -Tsvg > g.svg  # grafo de dependencias
```

> ⚠️ `terraform refresh` está deprecado. Ahora se usa
> `terraform plan -refresh-only` / `terraform apply -refresh-only`.

## Mutable vs immutable infrastructure

- **Mutable**: actualizar en sitio (subir nginx 1.17 → 1.18 en el mismo servidor). Con el
  tiempo cada servidor queda distinto (*configuration drift*).
- **Immutable**: se crea un recurso nuevo con la versión nueva y se borra el viejo. Es lo
  que hace Terraform cuando un cambio *forces replacement* (`-/+`).

## Lifecycle rules

```hcl
resource "local_file" "pet" {
  filename = "/root/pets.txt"
  content  = "We love pets!"

  lifecycle {
    create_before_destroy = true    # primero crea el nuevo y luego borra el viejo
    # prevent_destroy     = true    # el apply/destroy falla si intentaría borrarlo (DBs)
    # ignore_changes      = [tags]  # ignora cambios hechos fuera de Terraform en esos campos
    # ignore_changes      = all
    # replace_triggered_by = [aws_ami.x.id]
  }
}
```

- `prevent_destroy` **no** te protege si borras el bloque del código.
- `create_before_destroy` con un recurso que tiene nombre fijo (como este `local_file`)
  puede terminar **borrando el archivo nuevo**, porque ambos tienen el mismo nombre. Ojo
  con esto.

## Data sources

Sirven para **leer** recursos que Terraform no administra (creados a mano, por otro equipo o
por otro proyecto de Terraform).

```hcl
data "local_file" "dog" {
  filename = "/root/dog.txt"
}

resource "local_file" "pet" {
  filename = "/root/pets.txt"
  content  = data.local_file.dog.content
}
```

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]   # Canonical
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"]
  }
}
# ami = data.aws_ami.ubuntu.id
```

| | resource | data |
|---|---|---|
| Palabra clave | `resource` | `data` |
| Hace | Crea, actualiza, destruye | Solo lee |
| También llamado | managed resource | data resource |

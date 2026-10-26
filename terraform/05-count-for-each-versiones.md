# count, for_each y version constraints

## count

```hcl
variable "filename" {
  default = ["/root/pets.txt", "/root/dogs.txt", "/root/cats.txt"]
}

resource "local_file" "pet" {
  count    = length(var.filename)
  filename = var.filename[count.index]
  content  = "hola"
}
# Se crean: local_file.pet[0], pet[1], pet[2]
```

**Problema**: los recursos se identifican por **índice**. Si quitas
`"/root/pets.txt"` (el índice 0), todos se recorren: pet[0] pasa a ser dogs y pet[1] pasa a
ser cats → Terraform **reemplaza** pet[0] y pet[1] y **destruye** pet[2].

`count` va bien para N copias idénticas o para un condicional
(`count = var.crear ? 1 : 0`).

## for_each

```hcl
variable "filename" {
  type    = set(string)       # for_each necesita un map o un set (no una list)
  default = ["/root/pets.txt", "/root/dogs.txt", "/root/cats.txt"]
}

resource "local_file" "pet" {
  for_each = var.filename     # o toset(var.lista)
  filename = each.value
  content  = "hola"
}
# Se crean: local_file.pet["/root/pets.txt"], ...
```

Los recursos se identifican por **clave**, así que quitar un elemento borra **solo** ese.

Con un map: `each.key` y `each.value`.

```hcl
resource "aws_iam_user" "users" {
  for_each = toset(["lucy", "mark", "anna"])
  name     = each.key
}

output "pets" {
  value = local_file.pet           # con for_each es un map
}
```

## Version constraints

```hcl
terraform {
  required_version = ">= 1.10"
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}
```

| Expresión | Significa |
|---|---|
| `= 1.2.0` o `1.2.0` | Exactamente esa |
| `!= 2.0.0` | Cualquiera menos esa |
| `> 1.2.0`, `>= 1.2.0`, `<`, `<=` | Comparación |
| `> 1.2.0, < 2.0.0, != 1.4.0` | Combinadas |
| `~> 1.2` | **>= 1.2 y < 2.0** (solo puede subir el **último** número que escribes) |
| `~> 1.2.0` | **>= 1.2.0 y < 1.3.0** |

> ⚠️ Corrección respecto a las notas: `~> 5.20` significa **>= 5.20.0 y < 6.0.0**, no
> "solo 5.20.x". Para fijar el patch hay que escribir `~> 5.20.0`.

Para actualizar después de cambiar la restricción: `terraform init -upgrade`.

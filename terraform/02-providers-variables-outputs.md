# Providers, variables, outputs y dependencias

## Providers

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"     # <namespace>/<tipo> en el registry
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

Tipos de providers en el registry:

| Tier | Quién lo mantiene | Ejemplos |
|---|---|---|
| **Official** | HashiCorp | aws, azurerm, google, local, random |
| **Partner** | Empresa tecnológica, verificada | bigip (F5), heroku, digitalocean |
| **Community** | Usuarios | ucloud, netapp-gcp |

`terraform init` los baja a `.terraform/providers/` y crea `.terraform.lock.hcl`. La
dirección completa es `registry.terraform.io/hashicorp/aws`.

Varios proveedores del mismo tipo (por ejemplo, dos regiones) se configuran con `alias`:

```hcl
provider "aws" {
  alias  = "east2"
  region = "us-east-2"
}
resource "aws_s3_bucket" "b" {
  provider = aws.east2
  bucket   = "mi-bucket"
}
```

## Variables

```hcl
variable "filename" {
  type        = string
  default     = "/root/pets.txt"
  description = "Ruta del archivo"
  sensitive   = false
  validation {
    condition     = startswith(var.filename, "/root")
    error_message = "Debe estar en /root."
  }
}

resource "local_file" "pet" {
  filename = var.filename
  content  = "My favorite pet is ${var.pet_name}"   # interpolación
}
```

### Tipos

| Tipo | Ejemplo |
|---|---|
| `string` | `"hola"` |
| `number` | `1`, `3.14` |
| `bool` | `true` |
| `any` | por defecto |
| `list(string)` | `["Mr", "Mrs", "Sir"]` → `var.prefix[0]` |
| `map(string)` | `{ cat = "Jerry", dog = "Tom" }` → `var.pets["cat"]` |
| `set(string)` | como list pero **sin duplicados** y sin orden |
| `object({...})` | `object({ name = string, age = number })` |
| `tuple([...])` | `tuple([string, number, bool])` (tipos distintos en orden) |

### Formas de pasar valores

```bash
terraform apply                                   # pregunta en modo interactivo si no hay default
terraform apply -var "filename=/root/x.txt"
export TF_VAR_filename=/root/x.txt
terraform apply -var-file=variables.tfvars        # terraform.tfvars y *.auto.tfvars se cargan solos
```

### Precedencia (de menor a mayor)

1. Variables de entorno `TF_VAR_*`
2. `terraform.tfvars` / `terraform.tfvars.json`
3. `*.auto.tfvars` (en orden alfabético)
4. `-var` y `-var-file` en la línea de comandos (gana el último) ← **la más alta**

## Atributos y dependencias

Se usa el atributo de un recurso en otro con `<tipo>.<nombre>.<atributo>`:

```hcl
resource "random_pet" "my-pet" {
  length = 1
}

resource "local_file" "pet" {
  filename = "/root/pets.txt"
  content  = "My pet is ${random_pet.my-pet.id}"   # dependencia IMPLÍCITA
}
```

- **Implícita**: Terraform detecta la referencia y crea primero `random_pet`. Al destruir,
  lo hace en orden inverso.
- **Explícita**: cuando no hay referencia pero sí hay un orden necesario:

```hcl
resource "local_file" "pet" {
  ...
  depends_on = [random_pet.my-pet]
}
```

## Outputs

```hcl
output "pet-name" {
  value       = random_pet.my-pet.id
  description = "Nombre generado"
  sensitive   = false
}
```

```bash
terraform output              # todos
terraform output pet-name     # uno
terraform output -json
```

Sirven para mostrar datos tras el apply, pasárselos a scripts o a otras herramientas
(Ansible) y exponer valores de un módulo hacia afuera.

> ⚠️ Corrección respecto a las notas: `terraform graph` **no** muestra outputs. Genera el
> grafo de dependencias en formato DOT:
> `terraform graph | dot -Tsvg > graph.svg` (necesita `apt install graphviz`).

# Módulos, funciones, condicionales y workspaces

## Módulos

Cualquier carpeta con `.tf` es un módulo. Donde ejecutas `terraform` es el **root module**.
Los módulos evitan copiar y pegar y estandarizan la infraestructura.

```
mi-proyecto/
├── main.tf                 # root module
└── modules/
    └── payroll-app/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

```hcl
# Módulo local
module "us_payroll" {
  source         = "../modules/payroll-app"
  app_region     = "us-east-1"
  ami            = "ami-24e140119877avm"
}

# Módulo del registry
module "iam_iam-user" {
  source  = "terraform-aws-modules/iam/aws//modules/iam-user"
  version = "5.28.0"
  name    = "max"
  create_iam_user_login_profile = false
  create_iam_access_key         = false
}
```

```bash
terraform get        # baja los módulos (init también lo hace)
```

- Las **variables** del módulo son su "API" de entrada y los **outputs** su salida
  (`module.us_payroll.instance_id`).
- Para módulos del registry: fijar siempre `version` y leer la doc de inputs y outputs.

## Funciones

Para probarlas: `terraform console`.

```bash
$ terraform console
> length(["a","b","c"])
3
> upper("hola")
"HOLA"
```

| Tipo | Funciones |
|---|---|
| Numéricas | `max(-1, 2, 5)`, `min(...)`, `ceil(10.1)`→11, `floor(10.9)`→10, `max(var.lista...)` (expandir con `...`) |
| String | `split(",", "a,b")`, `join(",", ["a","b"])`, `lower`, `upper`, `title`, `substr("hola", 0, 2)`, `replace`, `format` |
| Colecciones | `length`, `index(var.lista, "x")`, `element(var.lista, 2)`, `contains(var.lista, "x")`, `toset`, `tolist`, `concat`, `flatten`, `distinct` |
| Maps | `keys`, `values`, `lookup(var.map, "clave", "default")`, `merge` |
| Archivos | `file("ruta")`, `templatefile("x.tpl", { ... })`, `filebase64` |
| Otras | `try(...)`, `can(...)`, `cidrsubnet`, `jsonencode`, `yamldecode` |

## Operadores y condicionales

- Aritméticos: `+ - * / %`
- Comparación: `== != > >= < <=`
- Lógicos: `&& || !`

```hcl
# condición ? si_verdadero : si_falso
resource "random_password" "pw" {
  length = var.length < 8 ? 8 : var.length
}

resource "aws_instance" "bastion" {
  count = var.env == "prod" ? 1 : 0          # crear o no
}
```

### Expresiones `for` y `locals`

```hcl
locals {
  common_tags = { team = "devops", env = var.env }
  upper_users = [for u in var.users : upper(u)]
  admins      = { for u in var.users : u => "admin" if u != "guest" }
}
# tags = local.common_tags
```

## Workspaces (CLI)

Permiten usar **el mismo código** con **states distintos** (por ejemplo, dev y prod).

```bash
terraform workspace list
terraform workspace new ProjectA
terraform workspace new ProjectB
terraform workspace select ProjectA
terraform workspace show
```

```hcl
variable "ami" {
  type = map(string)
  default = {
    ProjectA = "ami-0edab43b6fa892279"
    ProjectB = "ami-0c2f25c1f66a1ff4d"
  }
}

resource "aws_instance" "web" {
  ami           = lookup(var.ami, terraform.workspace)
  instance_type = "t2.micro"
  tags = { Name = terraform.workspace }
}
```

- Con el backend local, los states quedan en `terraform.tfstate.d/<workspace>/`.
- El workspace `default` usa `terraform.tfstate` en la raíz.
- En la práctica, muchos equipos prefieren **carpetas separadas por entorno**
  (`envs/dev`, `envs/prod`) en lugar de workspaces, porque se ve explícitamente qué se
  aplica dónde.

# Terraform

Infraestructura como código. Fuente: KodeKloud (Terraform Basics + Terraform for AWS).
Versión instalada: **v1.15.2**. La parte de AWS está en [../aws](../aws/README.md).

| Archivo | Tema |
|---|---|
| [01-iac-y-fundamentos.md](01-iac-y-fundamentos.md) | IaC, tipos de herramientas, HCL, flujo init/plan/apply |
| [02-providers-variables-outputs.md](02-providers-variables-outputs.md) | Providers, variables, tipos, precedencia, outputs, dependencias |
| [03-state-y-remote-backend.md](03-state-y-remote-backend.md) | State, remote state en S3, locking, comandos `state` |
| [04-comandos-lifecycle-data-sources.md](04-comandos-lifecycle-data-sources.md) | Comandos, mutable vs immutable, lifecycle, data sources |
| [05-count-for-each-versiones.md](05-count-for-each-versiones.md) | `count`, `for_each`, version constraints |
| [06-provisioners-import-debug.md](06-provisioners-import-debug.md) | Provisioners, taint/`-replace`, logs, `import` |
| [07-modulos-funciones-workspaces.md](07-modulos-funciones-workspaces.md) | Módulos, funciones, condicionales, consola, workspaces |

## Flujo básico

```bash
terraform init          # descarga los providers e inicializa el backend
terraform fmt           # formatea el código
terraform validate
terraform plan          # muestra qué cambiaría
terraform apply         # (-auto-approve para no confirmar)
terraform destroy
terraform state list
terraform output
```

## Símbolos del plan

| Símbolo | Significa |
|---|---|
| `+` | crear |
| `-` | destruir |
| `~` | modificar en sitio |
| `-/+` | destruir y recrear (*forces replacement*) |
| `<=` | leer (data source) |

## Problemas que me pasaron

| Síntoma | Causa | Solución |
|---|---|---|
| `Error acquiring the state lock` | Otro apply en curso, o uno que se cortó | Esperar; si es seguro: `terraform force-unlock <ID>` |
| `Backend configuration changed` | Cambié el bloque `backend` | `terraform init -migrate-state` o `-reconfigure` |
| `Inconsistent dependency lock file` | Cambié la versión de un provider | `terraform init -upgrade` |
| El provisioner no se ejecuta otra vez | Los provisioners de creación solo corren al crear | `terraform apply -replace=<recurso>` |

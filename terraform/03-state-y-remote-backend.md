# State y remote backend

## ¿Para qué el state?

`terraform.tfstate` (JSON) es el **mapa** entre el código y los recursos reales:

- Saber qué existe y con qué IDs, para crear, actualizar o borrar lo justo.
- Guardar metadata, como las **dependencias**, para destruir en el orden correcto aunque
  ya no estén en el código.
- **Rendimiento**: caché de atributos. En proyectos grandes se puede usar
  `terraform plan -refresh=false`.
- **Colaboración**: si el state es compartido, todo el equipo ve lo mismo.

Antes de cada plan o apply, Terraform hace un **refresh**: compara el state con la
infraestructura real.

## Consideraciones

- El state puede contener **datos sensibles** (IPs, contraseñas, llaves) en texto plano.
- **No** se sube a Git y **no** se edita a mano: se usan los comandos `terraform state`.
- Si dos personas aplican a la vez con states locales distintos → conflictos o recursos
  duplicados. Por eso existe el **remote state con locking**.

## Remote backend en S3

```hcl
terraform {
  backend "s3" {
    bucket       = "kodekloud-terraform-state-bucket01"
    key          = "finance/terraform.tfstate"
    region       = "us-west-1"
    use_lockfile = true          # locking nativo en S3 (Terraform 1.10+)
    encrypt      = true
  }
}
```

> ⚠️ Actualización respecto al curso: el curso usa `dynamodb_table = "state-locking"`
> para el lock. Desde **Terraform 1.10** S3 soporta locking nativo con
> `use_lockfile = true`, y `dynamodb_table` está **deprecado**. Con mi versión (1.15) uso
> `use_lockfile`.

```bash
terraform init                  # al agregar el backend pregunta si quieres migrar el state local
terraform init -migrate-state
rm terraform.tfstate            # el local ya no hace falta
```

En **Azure** el equivalente es el backend `azurerm` (Storage Account + container), que hace
locking con *blob leases*.

## State locking

Mientras se ejecuta un apply, Terraform bloquea el state para que nadie más lo modifique.
El backend local lo hace con un archivo y los remotos con su propio mecanismo.

```bash
terraform force-unlock <LOCK_ID>   # solo si un proceso se cortó y dejó el lock
```

## Comandos `terraform state`

```bash
terraform state list                          # recursos en el state
terraform state show aws_s3_bucket.finance    # atributos de uno
terraform state mv aws_dynamodb_table.state-locking aws_dynamodb_table.state-locking-db
                                              # renombrar sin destruir (cambiar también el .tf)
terraform state pull                          # descargar el state remoto (JSON) → útil con jq
terraform state pull | jq '.resources[] | select(.name == "state-locking-db")'
terraform state rm aws_instance.web           # que Terraform "olvide" un recurso (no lo borra)
terraform state push                          # subir un state (peligroso)
```

Hoy, en lugar de `state mv` se recomienda el bloque `moved` en el código:

```hcl
moved {
  from = aws_dynamodb_table.state-locking
  to   = aws_dynamodb_table.state-locking-db
}
```

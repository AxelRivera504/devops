# IAM (Identity and Access Management)

| Concepto | Qué es |
|---|---|
| **Root account** | El email con el que creaste la cuenta. Tiene acceso total. **Solo** para tareas de facturación y de cuenta; protegerlo con MFA y no usarlo en el día a día. |
| **User** | Persona o aplicación. Acceso por consola (contraseña) o programático (access keys). |
| **Group** | Conjunto de usuarios que comparten permisos (developers, admins). |
| **Role** | Identidad que **asume** un servicio o usuario de forma temporal. Por ejemplo, un EC2 que necesita leer S3. Sin llaves fijas. |
| **Policy** | Documento JSON con los permisos. Se adjunta a users, groups o roles. |

**Least privilege**: dar solo los permisos necesarios.

## Política

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }
  ]
}
```

Esta es `AdministratorAccess`. Hay políticas administradas por AWS (`AmazonS3ReadOnlyAccess`,
etc.) y políticas propias.

## IAM con Terraform

```hcl
provider "aws" {
  region = "us-west-2"
}

resource "aws_iam_user" "admin-user" {
  name = "lucy"
  tags = {
    Description = "Technical Team Leader"
  }
}

resource "aws_iam_policy" "adminUser" {
  name   = "AdminUsers"
  policy = file("admin-policy.json")       # o un heredoc <<EOF ... EOF
}

resource "aws_iam_user_policy_attachment" "lucy-admin-access" {
  user       = aws_iam_user.admin-user.name
  policy_arn = aws_iam_policy.adminUser.arn
}
```

Con varios usuarios desde una variable:

```hcl
variable "project-sapphire-users" {
  type    = list(string)
  default = ["mary", "jack", "jill", "mack", "buzz", "mater"]
}

resource "aws_iam_user" "users" {
  for_each = toset(var.project-sapphire-users)
  name     = each.value
}
```

También se puede usar el data source `aws_iam_policy_document` en lugar de escribir el
JSON a mano.

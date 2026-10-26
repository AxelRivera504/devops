# AWS

Lo visto en KodeKloud (Terraform for AWS). Cómo se crea todo con Terraform:
[../terraform](../terraform/README.md).

| Archivo | Tema |
|---|---|
| [01-iam.md](01-iam.md) | Usuarios, grupos, roles, políticas, IAM con Terraform |
| [02-s3-dynamodb.md](02-s3-dynamodb.md) | Buckets, bucket policies, DynamoDB |
| [03-ec2.md](03-ec2.md) | AMIs, tipos de instancia, key pairs, security groups, user_data |

## Conceptos

- **Cloud computing**: recursos bajo demanda, pagando por uso.
- **Región**: zona geográfica (por ejemplo, `us-east-1`). Cada una tiene varias **Availability
  Zones** (AZ), que son datacenters aislados (`us-east-1a`, `1b`…). Para alta disponibilidad
  se reparte la carga entre AZ.
- **Modelo de responsabilidad compartida**: AWS se encarga de la seguridad **del** cloud
  (hardware, red, datacenters) y tú de la seguridad **en** el cloud (datos, IAM, SO, red de
  tus VPC, cifrado).

## AWS CLI

```bash
aws configure                     # Access Key, Secret Key, región, formato
cat ~/.aws/config ~/.aws/credentials
aws sts get-caller-identity       # ¿con qué identidad estoy autenticado?
aws <servicio> <comando> help

aws iam create-user --user-name lucy
aws iam list-users
aws s3 ls
aws s3 cp archivo.txt s3://mi-bucket/
aws ec2 describe-instances --query 'Reservations[].Instances[].InstanceId'
```

En los laboratorios de KodeKloud se usa `--endpoint http://aws:4566` (un AWS simulado con
LocalStack).

## Credenciales en Terraform (de peor a mejor)

1. ❌ `access_key` y `secret_key` escritas en el bloque `provider` → nunca.
2. ✔ `~/.aws/credentials` / `aws configure` (Terraform los lee solo).
3. ✔ Variables de entorno `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`.
4. ✔✔ Roles y credenciales temporales (SSO, OIDC desde CI/CD, roles de instancia).

# S3 y DynamoDB

## S3 (Simple Storage Service)

Almacenamiento de **objetos** (archivos), no de bloques ni de sistema de archivos.

- **Bucket**: contenedor. El nombre es **único a nivel mundial** y debe ser compatible con
  DNS (minúsculas, sin `_`).
- **Objeto**: archivo + metadata. Máximo 5 TB por objeto.
- URL: `https://<bucket>.s3.<region>.amazonaws.com/<key>`
- Acceso: **bucket policies** (JSON a nivel bucket) y ACLs (legado, hoy desactivadas por
  defecto). Por defecto todo es **privado** y está activado *Block Public Access*.

```hcl
resource "aws_s3_bucket" "finance" {
  bucket = "finance-21092020"
  tags = {
    Description = "Finance and Payroll"
  }
}

resource "aws_s3_object" "finance-2020" {          # antes: aws_s3_bucket_object
  bucket = aws_s3_bucket.finance.id
  key    = "finance-2020.doc"
  source = "/root/finance/finance-2020.doc"
}

data "aws_iam_group" "finance-data" {
  group_name = "finance-analysts"
}

resource "aws_s3_bucket_policy" "finance-policy" {
  bucket = aws_s3_bucket.finance.id
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "*"
      Effect    = "Allow"
      Resource  = "${aws_s3_bucket.finance.arn}/*"
      Principal = { AWS = data.aws_iam_group.finance-data.arn }
    }]
  })
}
```

> En el provider AWS actual, el versionado, el cifrado y el ciclo de vida son **recursos
> aparte** (`aws_s3_bucket_versioning`, `aws_s3_bucket_server_side_encryption_configuration`,
> etc.), no argumentos dentro de `aws_s3_bucket`.

> ⚠️ Corrección respecto al curso: `Principal` debe ser un usuario o un rol. **Los grupos
> IAM no se pueden usar como Principal** en una bucket policy. El ejemplo del curso funciona
> en LocalStack, pero en AWS real falla. Lo correcto es dar el permiso al grupo con una
> política IAM.

## DynamoDB

Base de datos **NoSQL** clave‑valor/documentos, totalmente administrada (serverless),
con latencia de milisegundos y replicación entre AZ.

- **Primary key**: *partition key* (hash) y opcionalmente *sort key* (range).
- Cada ítem puede tener atributos distintos.

```hcl
resource "aws_dynamodb_table" "cars" {
  name         = "cars"
  hash_key     = "VIN"
  billing_mode = "PAY_PER_REQUEST"

  attribute {               # solo se declaran los atributos de las keys
    name = "VIN"
    type = "S"              # S string, N number, B binary
  }
}

resource "aws_dynamodb_table_item" "car-items" {
  table_name = aws_dynamodb_table.cars.name
  hash_key   = aws_dynamodb_table.cars.hash_key
  item = <<EOF
{
  "Manufacturer": {"S": "Toyota"},
  "Make":  {"S": "Corolla"},
  "Year":  {"N": "2004"},
  "VIN":   {"S": "4Y1SL65848Z411439"}
}
EOF
}
```

Antes, DynamoDB se usaba para el **lock del state de Terraform**. Ahora se usa
`use_lockfile = true` en S3 (ver [terraform/03](../terraform/03-state-y-remote-backend.md)).

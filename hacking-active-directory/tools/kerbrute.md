---
icon: windows
---

# Kerbrute

## KERBRUTE — Enumeración de usuarios en Active Directory

Kerbrute permite enumerar cuentas de dominio aprovechando el comportamiento de Kerberos Pre-Authentication, que en muchos entornos no genera alertas ni logs visibles, lo que lo convierte en una técnica relativamente sigilosa.

## Escenarios de partida habituales

Cuando no se nos proporciona un usuario inicial, el punto de apoyo en un dominio suele venir de:

* Credenciales en texto plano
* Hash NTLM de un usuario
* Shell como SYSTEM en un equipo unido al dominio
* Shell en el contexto de un usuario de dominio
* Enumeración de usuarios con Kerbrute (sin autenticación previa)

### ¿Qué hace Kerbrute?

* Valida nombres de usuario de Active Directory
* No requiere credenciales válidas
* Utiliza Kerberos para diferenciar usuarios existentes de no existentes
* Devuelve resultados rápidos

Los usuarios válidos obtenidos pueden usarse posteriormente para:

* Password spraying
* AS-REP roasting
* Ataques dirigidos

## Obtener Kerbrute

### Opción A: Descargar binarios precompilados

Repositorio oficial:

{% embed url="https://github.com/ropnop/kerbrute/releases/latest" %}

### Opción B: Clonar y compilar

Clonar el repositorio:

```bash
sudo git clone https://github.com/ropnop/kerbrute.git
```

#### 1. Opciones de compilación

Compilar todo:

```bash
sudo make all
```

Los binarios se generan en el directorio `dist/`.

#### 3. Verificar binarios

Listar binarios:

```bash
ls dist/
```

#### 4. Probar el binario

Ejecutar Kerbrute (Linux x64):

```bash
./kerbrute_linux_amd64
```

#### 5. Añadir Kerbrute al PATH&#x20;

Mover el binario:

```bash
sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute
```

Comprobar:

```bash
kerbrute
```

#### 6. Enumeración de usuarios

Ejecutar enumeración apuntando al Domain Controller:

```bash
kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users
```

Parámetros:

* -d → dominio
* \--dc → IP del DC
* jsmith.txt → lista de usuarios
* -o → archivo de salida

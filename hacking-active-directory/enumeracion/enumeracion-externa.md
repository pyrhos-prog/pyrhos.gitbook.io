# Enumeración externa

Antes de un pentesting podemos hacer una enumeración externa para conseguir informacion relevante como:&#x20;

* Validar la información que le proporciona el cliente en el documento de alcance
* Asegurarse de tomar medidas en contra del alcance adecuado cuando trabaja de forma remota
* Buscar cualquier información que sea de acceso público y que pueda afectar el resultado de su prueba, como credenciales filtradas

Haciendo esto garantizamos que la prueba sea lo más completa posible para nuestro cliente, ya que identificamos filtraciones de información y violaciones de datos.

Esto puede ser tan simple como obtener un nombre de usuario del sitio web principal o de las redes sociales del cliente.&#x20;

También podemos profundizar tanto como escanear repositorios de GitHub en busca de credenciales dejadas en envíos de código, buscar en documentos enlaces a una intranet o sitios accesibles de forma remota y simplemente buscar cualquier información que pueda indicarnos cómo está configurado el entorno empresarial.

## ¿Que estamos buscando?

| **Espacio IP (IP Space)**                        | ASN, rangos IP públicos, infraestructura expuesta, proveedores cloud y registros DNS.                                         |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| **Información de dominio**                       | Dominios, subdominios, servidores DNS, correo, VPN, aplicaciones web y tecnologías de seguridad visibles.                     |
| **Formato de usuarios y correos**                | Convención de nombres de usuario y correos corporativos (ej: `nombre.apellido@empresa.com`) para enumeración y autenticación. |
| **Divulgación de información (Data Disclosure)** | Documentos públicos (PDF, DOCX, XLSX, PPTX), metadatos, repositorios y cualquier dato interno expuesto accidentalmente.       |
| **Datos de brechas (Breach Data)**               | Usuarios, correos, contraseñas o hashes filtrados en fugas de datos públicas.                                                 |

## ¿Como lo buscamos?

| **ASN e IPs**                           | Identificar rangos de IP pertenecientes a una organización y su proveedor.            | [IANA](https://www.iana.org/), [ARIN](https://www.arin.net/), [RIPE](https://www.ripe.net/), [BGP Toolkit](https://bgp.he.net/)                                        |
| --------------------------------------- | ------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Dominios y DNS**                      | Obtener información sobre dominios, subdominios y registros DNS.                      | [Domaintools](https://www.domaintools.com/), [ICANN](https://lookup.icann.org/lookup), [PTRArchive](http://ptrarchive.com/), `dig`, `nslookup`, DNS públicos (8.8.8.8) |
| **Redes sociales**                      | Recopilar información sobre empleados, tecnologías usadas y estructura de la empresa. | LinkedIn, X/Twitter, Facebook, noticias                                                                                                                                |
| **Web corporativa**                     | Encontrar contactos, documentación, tecnologías y posibles objetivos.                 | Secciones "About Us", "Contact", PDFs, blogs corporativos                                                                                                              |
| **Repositorios y almacenamiento cloud** | Buscar código fuente, credenciales expuestas o archivos públicos.                     | GitHub, AWS S3, Azure Blob Storage, Google Dorks                                                                                                                       |
| **Bases de datos de brechas**           | Comprobar si correos corporativos o contraseñas han sido filtrados.                   | [HaveIBeenPwned](https://haveibeenpwned.com/), [DeHashed](https://haveibeenpwned.com/)                                                                                 |

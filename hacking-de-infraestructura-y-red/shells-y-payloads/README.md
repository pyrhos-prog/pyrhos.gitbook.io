---
icon: building-magnifying-glass
---

# Shells y Payloads

## Introducción&#x20;

### ¿Qué es una shell?

Una **shell** es un programa que proporciona al usuario una interfaz para introducir instrucciones en el sistema y ver la salida de texto. En el contexto del pentesting, obtener una shell en un sistema objetivo es uno de los resultados más valiosos de una explotación exitosa.

Cuando se escucha a un pentester decir _"pillé una shell"_, _"reventé una shell"_ o _"estoy dentro"_, significa que ha explotado con éxito una vulnerabilidad y ha obtenido acceso interactivo a la shell del sistema operativo del objetivo.

#### ¿Por qué obtener una shell?

La shell da acceso directo al **sistema operativo**, a los **comandos del sistema** y al **sistema de archivos**. Con ese acceso se puede:

* Enumerar el sistema en busca de vectores de escalada de privilegios
* Pivotar hacia otros sistemas de la red
* Transferir archivos (herramientas, loot)
* Mantener persistencia en el sistema
* Exfiltrar datos
* Documentar detalladamente el ataque

Sin una shell establecida, el avance en un sistema objetivo es muy limitado. Además, las shells de línea de comandos son **más difíciles de detectar** que un acceso gráfico por VNC o RDP, más rápidas para navegar el sistema y más fáciles de automatizar.

#### Perspectivas de una shell

| Contexto        | Descripción                                                                                                |
| --------------- | ---------------------------------------------------------------------------------------------------------- |
| **Informática** | Entorno de texto para administrar tareas y enviar instrucciones. Ejemplos: Bash, Zsh, cmd, PowerShell      |
| **Explotación** | Resultado de explotar una vulnerabilidad para obtener acceso interactivo remoto a un host                  |
| **Web**         | Webshell — explota una vulnerabilidad de subida de archivos para ejecutar instrucciones desde el navegador |

### ¿Qué es un payload?

Un **payload** es el código diseñado para explotar una vulnerabilidad en un sistema. Es lo que se entrega al objetivo para obtener acceso.

| Contexto         | Definición                                                                  |
| ---------------- | --------------------------------------------------------------------------- |
| **Redes**        | La parte encapsulada de datos de un paquete                                 |
| **Informática**  | La porción de un conjunto de instrucciones que define la acción a realizar  |
| **Programación** | La porción de datos referenciada por una instrucción                        |
| **Explotación**  | Código diseñado para explotar una vulnerabilidad y establecer acceso remoto |

Los payloads son el vehículo que entrega la shell. Sin un payload efectivo, no hay acceso.

### Anatomía de una shell

Toda shell en un sistema operativo se compone de tres elementos que trabajan juntos:

```
Sistema Operativo
      ↓
Emulador de terminal (aplicación que abre la ventana)
      ↓
Intérprete de lenguaje de comandos (Bash, PowerShell, cmd...)
```

#### Emuladores de terminal comunes

| Emulador                       | SO                    |
| ------------------------------ | --------------------- |
| Windows Terminal, cmder, PuTTY | Windows               |
| Alacritty, kitty               | Windows, Linux, macOS |
| GNOME Terminal, Konsole, xterm | Linux                 |
| Terminal, iTerm2               | macOS                 |

#### Identificar el intérprete activo

El signo `$` en el prompt indica shells como Bash, Ksh o POSIX. El prompt `PS C:\>` indica PowerShell. Para confirmar el intérprete en uso desde Linux:

```bash
# Ver los procesos activos de shell
ps

# Ver la variable de entorno SHELL
env | grep SHELL
# SHELL=/bin/bash
```

#### CMD vs PowerShell

|                           | CMD              | PowerShell              |
| ------------------------- | ---------------- | ----------------------- |
| **Origen**                | MS-DOS original  | Moderno, basado en .NET |
| **Output**                | Texto            | Objetos .NET            |
| **Historial de comandos** | No guarda        | Guarda historial        |
| **Disponibilidad**        | Desde Windows XP | Desde Windows 7         |
| **Rastro forense**        | Menor            | Mayor                   |
| **Ejecución de scripts**  | Batch (.bat)     | .ps1, cmdlets           |

Usar **CMD** cuando: el objetivo es antiguo, se necesitan interacciones simples, se quiere dejar menos rastro.

Usar **PowerShell** cuando: se necesitan cmdlets, interacción con objetos .NET, acceso a servicios cloud, o scripts complejos.

> El emulador de terminal que se usará en el objetivo dependerá de lo que esté disponible en ese sistema. En un pentest real no se elige — se usa lo que hay.

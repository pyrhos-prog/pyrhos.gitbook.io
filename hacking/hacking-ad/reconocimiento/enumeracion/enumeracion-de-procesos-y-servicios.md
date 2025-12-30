# Enumeración de Procesos y Servicios

## Como enumerar procesos de Windows

```bash
ps
```

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

También podemos enumerar procesos con el comando:

```bash
net start
```

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

## Conocer PID de un proceso

```bash
pgrep explorer.exe
```

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

O también podemos ver los PID de todos los procesos con el siguiente comando:

```bash
wmic ervice list brief
```

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

## Migrar a un proceso

```bash
migrate 2176
```

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

## Enumerar tareas programadas

```bash
tasklist /SVC
```

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

O también tenemos este otro comando:

```bash
schtasks /query /fo LIST
```

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

---
icon: gears
---

# Configuración de Kali Linux

### 1. ZSH y Oh-My-Zsh:

#### Instalar ZSH y configurar como shell por defecto

```bash
sudo apt install zsh -y
chsh -s $(which zsh)
```

#### Instalar Oh-My-Zsh

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### Instalación de paquetes clave

```bash
sudo apt install -y kali-linux-default kali-tools-top10 kali-tools-wireless kali-tools-web kali-tools-exploitation
```

### 2. Configuración de Alias para ahorrar tiempo

#### Editar .zshrc

```bash
# Actualizar el sistema operativo por completo
alias update="sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y"
# Limpiar los logs
alias cleanlogs="sudo find /var/log -type f -exec truncate -s 0 {} \;"
# Buscar exploits
alias se="searchsploit"
```

#### Aplicar los cambios

```bash
source ~/.zshrc
```

### 3. Hardening

#### UFW

```bash
sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
```

### 4. Herramientas extra

#### Kerbrute

```bash
sudo apt install golang -y
go install github.com/ropnop/kerbrute@latest
```

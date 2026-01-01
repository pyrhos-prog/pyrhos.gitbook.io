# FeroxBuster

Es una herramienta de descubrimiento de contenido rápida y recursiva escrita en Rust. Está diseñado para el descubrimiento por fuerza bruta de contenido no vinculado en aplicaciones web,, es más una herramienta de "navegación forzada" que un fuzzer como `ffuf`.

Para instalar `FeroxBuster`, puedes utilizar el siguiente comando:

```
curl -sL https://raw.githubusercontent.com/epi052/feroxbuster/main/install-nix.sh | sudo bash -s $HOME/.local/bin
```

#### Casos de uso <a href="#use-cases-2" id="use-cases-2"></a>

| `Recursive Scanning`         | Realice escaneos recursivos para descubrir directorios y archivos anidados.                       |
| ---------------------------- | ------------------------------------------------------------------------------------------------- |
| `Unlinked Content Discovery` | Identifique contenido que no esté vinculado dentro de la aplicación web.                          |
| `High-Performance Scans`     | Benefíciese del rendimiento de Rust para realizar descubrimientos de contenido de alta velocidad. |

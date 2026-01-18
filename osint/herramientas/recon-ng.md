---
icon: magnifying-glass
---

# Recon-ng

Recon-ng es una que se utiliza comúnmente en pruebas de penetración y evaluaciones de seguridad para recopilar información valiosa sobre un objetivo, como direcciones de correo electrónico, nombres de dominio, subdominios, información de red, entre otros.

<div data-with-frame="true"><figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure></div>

Al iniciar una búsqueda de módulos, puede que no encuentre ninguno porque no están instalados:

```bash
modules search
```

<div data-with-frame="true"><figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure></div>

Debemos instalar módulos desde el marketplace:

<div data-with-frame="true"><figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure></div>

Podemos buscar módulos disponibles con:

```bash
marketplace search
```

<div data-with-frame="true"><figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure></div>

Por ejemplo, para ver la información de un módulo específico:

```bash
marketplace info discovery/info_disclosure/interesting_files
```

<div data-with-frame="true"><figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure></div>

Para instalar el módulo seleccionado:

```bash
marketplace install discovery/info_disclosure/interesting_files
```

<div data-with-frame="true"><figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure></div>

Después de la instalación, podemos verificar que el módulo está disponible:

```bash
modules search
```

<div data-with-frame="true"><figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure></div>

Y cargar el módulo con:

```bash
modules load discovery/info_disclosure/interesting_files
```

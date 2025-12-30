---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/BGEOiV6kHOgZlzy7weRR/recon-ng
---

# Recon NG

Recon-ng es una que se utiliza comúnmente en pruebas de penetración y evaluaciones de seguridad para recopilar información valiosa sobre un objetivo, como direcciones de correo electrónico, nombres de dominio, subdominios, información de red, entre otros.

<div data-with-frame="true"><figure><img src="../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure></div>

Al iniciar una búsqueda de módulos, puede que no encuentre ninguno porque no están instalados:

```bash
modules search
```

<div data-with-frame="true"><figure><img src="../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure></div>

Debemos instalar módulos desde el marketplace:

<div data-with-frame="true"><figure><img src="../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure></div>

Podemos buscar módulos disponibles con:

```bash
marketplace search
```

<div data-with-frame="true"><figure><img src="../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure></div>

Por ejemplo, para ver la información de un módulo específico:

```bash
marketplace info discovery/info_disclosure/interesting_files
```

<div data-with-frame="true"><figure><img src="../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure></div>

Para instalar el módulo seleccionado:

```bash
marketplace install discovery/info_disclosure/interesting_files
```

<div data-with-frame="true"><figure><img src="../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure></div>

Después de la instalación, podemos verificar que el módulo está disponible:

```bash
modules search
```

<div data-with-frame="true"><figure><img src="../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure></div>

Y cargar el módulo con:

```bash
modules load discovery/info_disclosure/interesting_files
```


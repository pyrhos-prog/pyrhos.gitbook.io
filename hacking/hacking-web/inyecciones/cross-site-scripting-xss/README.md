# Cross-Site Scripting (XSS)

#### ¿Qué es XSS?

XSS (Cross-Site Scripting) es una vulnerabilidad muy común en aplicaciones web. Ocurre cuando una página no limpia ni valida correctamente lo que escriben los usuarios y muestra ese contenido directamente en el navegador.

Si un atacante introduce código JavaScript malicioso (por ejemplo, en un comentario o formulario), ese código puede quedar guardado o reflejado en la página. Cuando otro usuario visita esa página, su navegador ejecuta el JavaScript sin que lo note.

![Matriz de riesgo con ejes: Probabilidad (baja a alta) e Impacto (baja a alta), mostrando las estrategias: Reducir, Evitar, Aceptar, Transferir.](../../.gitbook/assets/xss_risk_chart_1.jpg)

***

***

#### ¿Dónde se ejecuta XSS y a quién afecta?

* XSS se ejecuta solo en el navegador del usuario, no en el servidor.
* No ataca directamente al backend, pero afecta a los usuarios que visitan la página vulnerable.
* Por eso se considera un riesgo medio:\
  impacto bajo en el servidor + muy frecuente = riesgo importante.

***

#### ¿Qué puede hacer un atacante con XSS?

Como el ataque usa JavaScript, puede hacer casi cualquier cosa que permita el navegador, por ejemplo:

* Robar cookies de sesión (y secuestrar cuentas).
* Realizar acciones en nombre del usuario, como cambiar contraseñas.
* Mostrar anuncios falsos o redirigir a sitios maliciosos.
* Ejecutar scripts que consuman recursos (como minería de criptomonedas).

***

#### Limitaciones de XSS

* El código se limita al motor JavaScript del navegador (como V8 en Chrome).
* Normalmente solo puede interactuar con el mismo dominio del sitio vulnerable.
* No puede ejecutar código directamente en el sistema operativo.

Sin embargo, si existe una **vulnerabilidad grave en el navegador**, un XSS puede ser el primer paso para explotarla y llegar a ejecutar código en el equipo del usuario.

***

#### Casos reales importantes

* **Gusano Samy (MySpace, 2005):**\
  Se propagó automáticamente a más de un millón de perfiles en un solo día usando XSS.
* **TweetDeck (Twitter, 2014):**\
  Un XSS provocó miles de retuits automáticos en minutos.
* **Google y Apache:**\
  Incluso plataformas muy grandes han tenido vulnerabilidades XSS explotables en años recientes.

***

### Tipos de XSS

| Tipo                             | Descripción                                                                                                                                                                                                         |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Stored (Persistent) XSS`        | Ocurre cuando la entrada del usuario se almacena en la base de datos back-end.                                                                                                                                      |
| `Reflected (Non-Persistent) XSS` | Se produce cuando la entrada del usuario se muestra en la página después de ser procesada por el servidor backend, pero sin almacenarse                                                                             |
| `DOM-based XSS`                  | Ocurre cuando se muestra directamente en el navegador y se procesa completamente en el lado del cliente, sin llegar al servidor back-end (a través de parámetros HTTP del lado del cliente o etiquetas de anclaje). |

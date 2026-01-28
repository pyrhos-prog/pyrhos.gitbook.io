# IDOR

Un IDOR es una vulnerabilidad que permite acceder a objetos internos (Ds, UUIDs, nombres de archivos, índices) sin verificar si el usuario autenticado esta autorizado o no.

#### Ejemplo

```
GET /api/users/123/profile
```

Mi usuario tiene el identificador 123

```
GET /api/users/124/profile
```

Si cambio el identificador de mi usuario a otro número como 124 y hay un IDOR voy a poder ver el /profile de la cuenta con el identificador 124


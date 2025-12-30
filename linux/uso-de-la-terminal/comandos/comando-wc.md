# comando wc

El comando wc sirve para contar **líneas, palabras y caracteres** de uno o varios archivos o de la entrada estándar.

Por ejemplo, si lo ponemos sin ninguna opción nos cuenta el número de líneas, palabras y el número de bytes de un archivo.

```
pyrhos@debian:~/Documentos$ wc ping.log 
 15 107 786 ping.log
```

Para contar el numero de líneas se usa la opción -l

```
pyrhos@debian:~/Documentos$ wc -l ping.log 
15 ping.log
```

Para el número de carácteres se usa -m&#x20;

```
pyrhos@debian:~/Documentos$ wc -m ping.log 
786 ping.log
```

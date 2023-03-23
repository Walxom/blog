---
title: Explotación de permisos SUID
date: 2021-12-27 10:24:23 -0500
categories: [Privilege Escalation, Linux]
tags: [privesc,linux,suid]
image:
  path: https://i.ibb.co/bLwX9yS/explotacion-de-permisos-suid.png
  alt: Explotación de permisos SUID 
---

Hay veces que es necesario que un programa se ejecute con los privilegios de su propietario en lugar de con los privilegios del usuario que lo ejecuta.

## Permisos SUID

El bit de SUID/setuid (Set User ID) se activa sobre un fichero añadiéndole 4000 a la representación octal de los permisos de un archivo y agregándole permisos de ejecución al propietario, al hacer esto en lugar de la x aparecerá una s o una S si no hemos otorgado el permiso de ejecución correspondiente (en este caso el bit no tiene efecto):

```terminal
chmod 4100 file
```

![SUID "s"](https://i.ibb.co/mtRxbD1/f-4100.png)

```terminal
chmod 4000 file
```

![SUID "S"](https://i.ibb.co/y6HcLcj/f-4000.png)

El bit SUID activado sobre un fichero indica que todo aquel que ejecute el archivo va a tener durante la ejecución los mismos privilegios que quién lo creó, por ejemplo: si el administrador crea un fichero y lo `_setuida_`, todo aquel usuario que lo ejecute va a disponer de sus permisos hasta que el programa finalice.

Un ejemplo sería este código:

```c
#include <stdio.h>
#include <unistd.h>

main(){
	printf("\nUID: %d, EUID: %d\n",getuid(),geteuid());
}
```

Como usuario root lo compilamos y le agregamos permisos **SUID**:

```terminal
cc <filename>.c -o suid
chmod u+s ./suid
```

Ahora nos convertiremos en un usuario normal y ejecutaremos el programa que hemos compilado, el resultado debería ser algo así:

```terminal
./sys

UID: 1000, EUID: 0
```

> **UID**: Es el usuario original
> **EUID**: Es el usuario al que cambiaste

## GTFOBINS

[GTFOBINS](https://gtfobins.github.io/) es una lista seleccionada de binarios de **Unix** que se pueden utilizar para eludir las restricciones de seguridad locales en sistemas mal configurados. Mayormente, esta página se usa como fuente de información para escalar privilegios en los sistemas **Linux**.

Vamos a escoger un binario de sus listas e intentaremos escalaremos privilegios. En este caso para hacerlo muy sencillo escogeremos el binario `find`:

![Find - SUID](https://i.ibb.co/Stb59sz/find-suid-0.png)

> Agregar el permiso SUID a `find`: `chmod u+s /usr/bin/find`

Leeremos y ejecutaremos lo que nos dice en el apartado de **SUID**:

![Find - SUID](https://i.ibb.co/Ky20hgL/find-suid-1.png)

### Privilege Escalation / SUID - find

![Find - SUID](https://i.ibb.co/8mmp0Zk/find-suid-2.png)

### Explicación

Solo es cuestión de analizar:

1.  `find` tiene un parámetro que te permite ejecutar comandos a nivel de sistema.
2.  También tiene el permiso `suid` agregado, lo que significa que podemos ejecutar el programa/comando como su propietario.
3.  Viendo quien es el propietario de `find` nos damos cuenta de que es `root`.
4.  `bash` tiene el parámetro `-p` que te permite convertirte en el usuario propietario de su binario por si tiene el permiso `suid`.

Sabiendo esto podemos llegar a la conclusión que debemos abusar del parámetro `-exec` y ejecutar el comando `bash -p` para convertirnos en su propietario, en este caso `root`.

## Buscar ficheros que tengan el permiso SUID

```terminal
find / -perm -4000 -ls 2>/dev/null
```

| Parámetro | Descripción |
|-|-|
| find | buscar archivos en una jerarquía de directorios |
| / | desde que directorio empezar a buscar |
| -perm -4000 | especificamos que queremos buscar ficheros que contengan los permisos `4000` o SUID |
| -ls | seria como ejecutar un `ls -l` en cada fichero de salida |
| 2>/dev/null| que dirija todos los errores al `/dev/null` |

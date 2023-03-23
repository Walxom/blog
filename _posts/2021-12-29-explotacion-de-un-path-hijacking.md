---
title: Explotación de un PATH Hijacking
date: 2021-12-29 17:59:23 -0500
categories: [Privilege Escalation, Linux]
tags: [privesc,linux,path-hijacking]
image:
  path: https://i.ibb.co/kxjF9wz/path-hijacking.png
  alt: Explotación de un PATH Hijacking 
---

En los sistemas operativos hay un concepto llamado las variables de entorno, quizás algunos ya hayamos usado estos sin darnos cuenta, ya sea `%AppData%` para encontrar el directorio donde poner los mods de Minecraft o `%Temp%` para borrar los archivos temporales (si eres de Microsoft).

Las variables de entorno son valores dinámicos que afectan a los programas o procesos que se ejecutan en un sistema operativo. Estas se pueden, modificar, guardar y eliminar.

En Linux puedes ver la lista completa de variables de entorno de muchas formas, pero las más utilizadas son con el comando `env`, `set`, `printenv`.

![env](https://i.ibb.co/ZKtDH90/1.png)

## Variables de entorno comunes en Linux

| Variable | Valor |
|:-:|:-|
| PATH | lista ordenada de rutas donde se buscaran ejecutables. |
| HOME | directorio de inicio del usuario actual. |
| SHELL | nombre de la shell interactiva que se está ejecutando actualmente. |
| TERM | nombre de la terminal que se está ejecutando. |
| HOSTNAME | nombre de host del sistema. |
| USER | nombre del usuario actual. |
| PWD | ruta al directorio de trabajo actual. |
| LANG | idioma por defecto del sistema. |
| RANDOM | valor de número aleatorio. |
| PS1 | prompt actual. |

Para mí la manera más fácil de imprimir el contenido de una variable en específica es con: `echo $<VARNAME>`

```terminal
echo $PATH

/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```

pero también existe otra forma como con: `printenv <VARNAME>`

```terminal
printenv PATH

/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```

hay muchos más comandos que te pueden imprimir las variables de entorno, pero estos dos son los más comunes e usados.

## Crear o modificar una variable de entorno en Linux

El comando `export` sirve para exportar variables a procesos secundarios. La sintaxis básica de este comando se vería así: `export <VARNAME>=<value>`. Se puede modificar una variable de entorno de la siguiente manera:

```
export TERM=xterm
echo $TERM

xterm
```

Para crear sería lo mismo que modificar una variable de entorno nueva.

## Eliminar una variable de entorno en Linux

Se puede eliminar una variable de entorno con en Linux con el comando `unset`, de la siguiente manera:

```terminal
unset PATH
echo $PATH
```

Ya habiendo aprendido un poco sobre las variables de entorno, puedo explicar como se puede llegar a escalar privilegios con estas, en especial con una.

## Variable de entorno PATH en Linux

Se usa para ubicar comandos o ficheros binarios dentro de la jerarquía de directorios. El `PATH` normalmente se crea en un conjunto de directorios fijos donde el sistema siempre buscara comandos que un usuario escriba. Estas tienen orden de prioridad que empieza desde la derecha hasta la izquierda.

## Ruta absoluta y relativa

Esto son dos conceptos básicos que debemos de saber para entender como funciona este ataque.

-   **La ruta absoluta:** siempre contiene el elemento raíz y la lista completa de directorios necesaria para ubicar el archivo.
    
-   **La ruta relativa:** debe combinarse con otra ruta para acceder a un archivo. Por ejemplo, Downloads/Firefox es una ruta relativa.

## ¿Qué es el Path Hijacking?

Como su propio nombre indica Path Hijacking (Secuestro de rutas), este ataque consiste en secuestrar las rutas habituales definidas por la variable de entorno PATH, donde en vez de buscar los comandos por `/usr/local/bin:/usr/bin:/bin`, nosotros podemos decirle en donde buscar. Esto en un fichero binario que contenga el permiso SUID asignado, puede ser muy peligroso.

## Privilege Escalation

Para demostrar y poner en práctica esta técnica, voy a escoger un ejercicio del módulo [Linux PrivEsc](https://tryhackme.com/room/linuxprivesc) de la plataforma [TryHackMe](https://tryhackme.com/).

![Tasks](https://i.ibb.co/Gt0FkZC/linux-privesc.png)

### SUID / SGID Executables - Environment Variables

Para empezar vamos a ver que contiene el `PATH` y su orden de prioridad de las rutas.

![printenv](https://i.ibb.co/cKqgYkQ/2.png)

No vemos nada raro, son las rutas habituales de un sistema operativo GNU/Linux. El laboratorio nos dice que tiene un archivo binario llamado `suid-env` en la ruta `/usr/local/bin/`:

![suid-env](https://i.ibb.co/qkLZBHc/3.png)

si le pasamos el archivo ejecutable al comando `strings`, este nos permitirá ver los caracteres legibles para nosotros dentro de cualquier archivo:

![strings](https://i.ibb.co/RQwsFMz/4.png)

Como podemos ver esta ejecutando `service apache2 start`, y no está especificando la ruta absoluta del comando `service`, y por ende significa que está usando el `PATH` para encontrar el comando. Vemos también que el ejecutable tiene asignado el permiso SUID y su propietario es root.

Lo que haremos será remplazar el comando `service` para que en vez que inicie un servicio, le asigne el permiso SUID a la `bash`.

Nos metemos al directorio `/tmp` y con el comando `echo` creamos un archivo que se llame `service`, dentro de el voy a poner `chmod u+s /bin/bash` para que le asigne el permiso SUID a la `bash`, este es el archivo que remplazara al comando `service` de la ruta de `/usr/bin`. Para finalizar le asignaremos el permiso de ejecución al “nuevo comando” con `chmod +x service` :

![service](https://i.ibb.co/d0gH0LQ/5.png)

Ya teniendo el “nuevo comando” listo, podemos agregarle una nueva ruta a nuestro PATH y ponerla como primera en la orden de prioridad. En este caso como yo tengo el nuevo comando `service` en el directorio `/tmp`, lo exportaré como `export PATH=/tmp:$PATH`:

![export PATH - new](https://i.ibb.co/C73Ts28/6.png)

Ahora llego la hora de la acción, si ejecutamos el `suid-env` podemos ver que en vez de arrancar el servicio apache, nos asignó el permiso SUID a la `/bin/bash`:

![/bin/bash - SUID](https://i.ibb.co/pz16rD0/7.png)

y así de fácil es como escalamos privilegios:

![bash -p](https://i.ibb.co/DWvVgX9/8.png)


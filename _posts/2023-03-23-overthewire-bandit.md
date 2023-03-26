---
title: OverTheWire - Bandit
date: 2023-03-25 19:31:25 -0500
categories: [OverTheWire]
tags: [overthewire,bandit]
#image:
#	path:
#	alt:
---

Este es un juego creado por OverTheWire para principiantes con la finalidad de enseñar lo básico sobre los fundamentos de Linux.

Hay varias cosas que puede intentar cuando no está seguro de cómo continuar:

-   Primero, si conoce un comando, pero no sabe cómo usarlo, pruebe el manual (página man) ingresando `man <comando>`. Por ejemplo, `man ls` para aprender sobre el comando "ls".
-   En segundo lugar, si no hay una página de man, el comando puede ser una función del shell. En ese caso utiliza el comando `help <X>`. Por ejemplo, `help cd`
-   También tienes al comando `whatis`, sirve para mostrar las descripciones de las páginas del manual de un comando.
-   Además, tu motor de búsqueda favorito es tu amigo. Aprende a usarlo. Yo recomiendo Google.

## Level 0 → Level 1

El objetivo de este nivel es que te conectes al juego usando SSH. El host al que debes conectarte es [bandit.labs.overthewire.org](http://bandit.labs.overthewire.org/), en el puerto `2220`. El nombre de usuario es `bandit0` y la contraseña es `bandit0`.

### Resolución:

SSH (o Secure Shell, en español: intérprete de órdenes seguro) es el nombre de un protocolo y del programa que lo implementa cuya principal función es el acceso remoto a un servidor por medio de un canal seguro en el que toda la información está cifrada.

En resumidas palabras, SSH permite a los usuarios conectarse a un host remotamente de forma segura, ya que establecerá una conexión cifrada.

La contraseña para el siguiente nivel está almacenada en un archivo llamado `readme` ubicado en el directorio `home`. Usa esa contraseña para entrar en `bandit1` usando SSH. 

> Siempre que encuentres la contraseña para un nivel, usa SSH (en el puerto 2220) para entrar en ese nivel y continuar el juego.
{: .prompt-info }

```bash
ssh bandit0@bandit.labs.overthewire.org -p2220
# password: bandit0

cat readme
more readme
```

Los dos comandos (`cat` y `more`) sirven para mostrar el contenido de un archivo.

![OverTheWire - Bandit0](https://i.ibb.co/k1vJR8b/StLGf6x.png)

## Level 1 → Level 2

La contraseña para el siguiente nivel se almacena en un archivo llamado **`-`** ubicado en el directorio de inicio.

### Resolución:

Primero debemos entender dos conceptos muy importantes y básicos:

-   **Una ruta absoluta** es aquella en la que se indican todos los directorios de la jerarquía desde el root hasta el archivo o directorio final, por ello inicia con `/`. Cuando utilizamos el comando `pwd` este nos muestra una **ruta absoluta**.
-   **Una ruta relativa** es una forma de especificar la ubicación de un archivo o directorio en relación con la ubicación del archivo que se está ejecutando actualmente o el directorio en el que se encuentra.

En Linux, el punto (".") y los dos puntos ("..") tienen un significado especial en las rutas de directorio.

El punto (".") representa el directorio actual. Por ejemplo, si estás en el directorio "/home/juancarlos" y quieres acceder al archivo "si.txt" que se encuentra en ese mismo directorio, la ruta completa sería "/home/juancarlos/si.txt", pero puedes usar la ruta relativa "./si.txt" para indicar que está en el directorio actual.

Los dos puntos ("..") representan el directorio padre. Por ejemplo, si estás en el directorio "/home/juancarlos/documentos" y quieres acceder al archivo "si.txt" que se encuentra en el directorio superior (padre) "/home/juancarlos", la ruta completa sería "/home/juancarlos/si.txt", pero puedes usar la ruta relativa "../si.txt" para indicar que está en el directorio padre.

```bash
# ruta absoluta
cat /home/bandit1/-
# rutas relativas
cat ./-
cat ~/-
cat `pwd`/-
cat $PWD/-
cat $HOME/-
```

![OverTheWire - Bandit1](https://i.ibb.co/LvTLNyn/StLGf6x.png)

## Level 2 → Level 3

La contraseña para el siguiente nivel se almacena en un archivo llamado **`spaces in this filename`** ubicado en el directorio de inicio.

### Resolución:

Entendamos la diferencia entre comillas simples y dobles.

Las comillas dobles (`"`) se usan para encerrar frases, generalmente se usan cuando tenemos que referirnos a un nombre de archivo o directorio que contiene espacios.

```bash
cat "spaces in this filename"
```

Lo que esté dentro de las comillas dobles se toma literalmente a excepción de los caracteres `$` (sustitución de variable), `` ` `` (sustitución de comando) y `\\` (carácter de escape).

Más ejemplos:

```bash
# sustitución de variable
echo "estoy en $PWD"
# sustitución de comando
echo "el archivo se llama: `ls`"
# carácter de escape
echo "el archivo se llama: \\$(ls)"
```

Las comillas simples (`'`) también se usan para encerrar frases, sin embargo, todo lo que está dentro de las comillas simples es tomado literalmente.

```bash
echo 'estoy en la ruta $PWD'
echo 'el archivo se llama: `ls`'
echo 'el archivo se llama: \\$(ls)'
```

Sabiendo esto podemos resolver este reto de diferentes formas:

```bash
cat "spaces in this filename"
cat 'spaces in this filename'
```

También se puede de una forma bastante curiosa, que es escapando los espacios:

```terminal
cat spaces\ in\ this\ filename
```

![OverTheWire - Bandit2](https://i.ibb.co/6gKXFZJ/StLGf6x.png)

## Level 3 → Level 4

La contraseña para el siguiente nivel se almacena en un archivo oculto en el directorio `inhere`.

### Resolución:

En Linux, los archivos y directorios se pueden ocultar si tiene un punto al principio del nombre. Con el comando `ls` podemos listar el contenido de un directorio. Este tiene un parámetro `-a` que sirve para listar los archivos y directorios ocultos.

```bash
ls inhere/ -a
cat inhere/.hidden
```

![OverTheWire - Bandit3](https://i.ibb.co/JHq4rDs/StLGf6x.png)

## Level 4 → Level 5

La contraseña para el siguiente nivel se almacena en el único archivo legible por humanos en el directorio `inhere`.

Consejo: si su terminal está desordenada, pruebe el comando "reset".

### Resolución:

Lo que se busca es que encuentres el único archivo que tenga sea legible para humanos (ASCII Text). 

Para averiguar el tipo de archivo de cada fichero se puede hacer usando el comando `file`.

```bash
# Primer método - fácil
file inhere/* | grep "ASCII"
cat inhere/-file07
# Segundo método - intermedio
file inhere/* | grep "ASCII" | awk '{print $1}' FS=':' | xargs cat
```

![OverTheWire - Bandit4](https://i.ibb.co/BCspSZC/StLGf6x.png)

## Level 5 → Level 6

La contraseña para el siguiente nivel se almacena en un archivo en algún lugar del directorio `inhere` y tiene todas las siguientes propiedades:

-   legible para el ser humano
-   1033 bytes de tamaño
-   no ejecutable

### Resolución:

Nos pide que encontremos, encontremos un archivo con ciertas características. Como hay muchas resoluciones que lo hacen con el comando `find`, lo haremos con el `du`.

```bash
du -ab inhere/ | grep "1033"
```

Lo que hace es mostrar el tamaño de todos los archivos y directorios en el directorio "inhere/" y sus subdirectorios, en bytes `-b` y de forma recursiva `-a`.  Luego con `grep`  busca en la salida del comando anterior las líneas que contienen el número "1033".

![OverTheWire - Bandit5](https://i.ibb.co/9snYPV0/StLGf6x.png)

> Recomiendo pasarle al final el comando `xargs` para que no se deforme nuestra terminal con la salida del fichero.
{: .prompt-warning}

Por otro lado, si quieres resolver este reto la forma más tradicional, puedes hacerlo con el comando `find` y con los siguientes parámetros y argumentos:

```bash
find inhere/ -type f -readable -size 1033c ! -executable -exec file {} \;
```

Este comando busca en el directorio "inhere/" y en sus subdirectorios `-type f` todos los archivos regulares que sean legibles para el usuario actual `-readable`, tengan un tamaño de 1033 bytes `-size 1033c` y no sean ejecutables `! -executable`. La opción `-c` en el tamaño especifica que se busca un tamaño exacto en bytes. 

![OverTheWire - Bandit5](https://i.ibb.co/WyX66CT/StLGf6x.png)

## Level 6 → Level 7

La contraseña para el siguiente nivel se almacena **en algún lugar del servidor** y tiene todas las siguientes propiedades:

-   propiedad del usuario bandit7
-   propiedad del grupo bandit6
-   33 bytes de tamaño

### Resolución:

Es casi lo mismo que el anterior reto usando `find`, pero con diferentes parametros:

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

La opción `-user` se usa para especificar el usuario propietario del archivo que se busca, mientras que la opción `-group` se usa para especificar el grupo propietario del archivo. 

Lo nuevo aqui es el "2>/dev/null". Esto se utiliza para redirigir cualquier mensaje de error hacia el "/dev/null", este es un dispositivo virtual que descarta todo lo que se escribe en él.. Esto se hace para evitar que se muestren mensajes de error que podrían aparecer al buscar archivos en ciertas ubicaciones del sistema de archivos para las cuales el usuario que ejecuta el comando no tiene permiso.

![OverTheWire - Bandit6](https://i.ibb.co/7kV4HtJ/StLGf6x.png)

## Level 7 → Level 8

La contraseña para el siguiente nivel se almacena en el archivo **data.txt** junto a la palabra **`millionth`.**

### Resolución:

Para esto existe un comando llamado `grep`, este lo que hace es buscar patrones dentro de un archivo o una cadena de texto. Es muy útil cuando necesitamos encontrar una palabra o frase específica dentro de un archivo o en la salida de un comando.

En este caso, podemos utilizar el comando `grep` para buscar la palabra "millionth" dentro del archivo "data.txt" y encontrar la contraseña correspondiente:

```bash
grep "millionth" ./data.txt
```

![OverTheWire - Bandit7](https://i.ibb.co/PjCZbpY/StLGf6x.png)

## Level 8 → Level 9

La contraseña para el siguiente nivel se almacena en el archivo data.txt y es la única línea de texto que aparece una sola vez.

### Resolución:

Podemos utilizar una combinación de los comandos `sort` y `uniq`. El comando `sort` se utiliza para ordenar las líneas de texto en orden alfabético, y el comando `uniq` para encontrar las líneas únicas en un archivo.

Sin embargo, para encontrar la única línea que aparece una sola vez en el archivo, debemos utilizar la opción `-u` de `uniq`, que mostrará solo las líneas que aparecen una sola vez en el archivo después de que se haya ordenado con `sort`:

```bash
sort data.txt | uniq -u
```

Este comando ordenará las líneas en el archivo "data.txt" en orden alfabético y luego mostrará solo la línea que aparece una sola vez.

![OverTheWire - Bandit8](https://i.ibb.co/fQ11QpW/StLGf6x.png)

## Level 9 → Level 10

La contraseña para el siguiente nivel se almacena en el archivo **data.txt** en una de las pocas cadenas legibles para el ser humano, precedida por varios caracteres "=".

### Resolución:

Para resolver este reto podemos utilizar el comando `strings`, que nos permite buscar cadenas legibles para el ser humano dentro de un archivo binario. En este caso, queremos buscar todas las cadenas que contengan los caracteres "=".

Una vez que tenemos las cadenas legibles, podemos utilizar el comando `grep` para filtrar sólo aquellas que contengan los caracteres "`===`":

```bash
strings data.txt  | grep "==="
```

Este comando busca todas las cadenas legibles dentro del archivo "data.txt" y filtra sólo aquellas que contengan los caracteres "`===`". 

![OverTheWire - Bandit9](https://i.ibb.co/P5NL1Nm/StLGf6x.png)

## Level 10 → Level 11

La contraseña para el siguiente nivel se almacena en el archivo **data.txt**, que contiene datos codificados en base64.

### Resolución:

Debemos decodificar el contenido en base64 para poder leer la contraseña.

Podemos decodificar el contenido utilizando el comando "base64" junto con la opción "-d" para indicar que queremos decodificar el contenido.

```bash
cat data.txt  | base64 -d
```

Este comando básicamente toma la salida del comando "cat" y la envía como entrada al comando "base64" para que decodifique el contenido. La opción "-d" se utiliza para decodificar la entrada que se le proporciona en formato base64.

![OverTheWire - Bandit10](https://i.ibb.co/7Vm4zNw/StLGf6x.png)

## Level 11 → Level 12

La contraseña para el siguiente nivel se almacena en el archivo **data.txt**, donde todas las letras minúsculas `(a-z)` y mayúsculas `(A-Z)` han sido rotadas en 13 posiciones.

### Resolución:

Este reto implica descifrar un archivo que ha sido codificado utilizando el cifrado ROT13. 

El cifrado ROT13 es un tipo de cifrado César que sustituye cada letra por la letra que se encuentra trece posiciones por delante en el alfabeto. Por ejemplo, la letra "A" se convierte en "N", la letra "B" en "O", y así sucesivamente hasta llegar a la letra "M", que se convierte en "Z".

![ROT13](https://upload.wikimedia.org/wikipedia/commons/thumb/3/33/ROT13_table_with_example.svg/1200px-ROT13_table_with_example.svg.png)

Para descifrar el archivo, podemos utilizar el comando "tr" en la terminal de Linux, que es una herramienta que se utiliza principalmente para realizar transformaciones de caracteres. 

```bash
cat data.txt | tr a-zA-Z n-za-mN-ZA-M
```

En este caso, la opción "a-zA-Z" especifica que todas las letras del alfabeto en minúsculas y mayúsculas deben ser reemplazadas, y la cadena "n-za-mN-ZA-M" indica que cada letra debe ser reemplazada por la letra que se encuentra trece posiciones por delante en el alfabeto. 

![OverTheWire - Bandit11](https://i.ibb.co/ydzDy4X/StLGf6x.png)

## Level 12 → Level 13

La contraseña para el siguiente nivel se almacena en el archivo **data.txt**, que es un volcado **hexadecimal** de un archivo que ha sido comprimido repetidamente. Para este nivel puede ser útil crear un directorio bajo `/tmp` en el que se pueda trabajar usando `mkdir`. Por ejemplo: `mkdir /tmp/myname123`. Luego copie el archivo de datos usando `cp`, y renómbrelo usando mv (¡lea las páginas de manual!)

### Resolución:

Este nivel consiste en descomprimir un archivo que ha sido comprimido repetidamente y se encuentra en formato hexadecimal. Para ello, haré uso de tuberías para descomprimir el archivo de manera repetida con ciertos comandos hasta obtener el archivo final.

Primero se usará el comando `xxd` con el parámetro `-r` para revertir el archivo hexadecimal a su formato original. Luego, con el comando `file -` identificaremos de qué tipo de archivo se trata. 

A continuación, se usan los siguientes comandos: `zcat` (gzip), `bzcat` (bzip2), `tar` (tar), que nos permitirá descomprimir archivos repetidamente.

```bash
xxd -r data.txt | file -
# Asi sucesivamente hasta dar con el ASCII text
xxd -r data.txt | zcat | bzcat | zcat | tar xO | tar xO | bzcat | tar xO | zcat
```

Al final, se utiliza el comando `zcat` para mostrar el contenido del archivo descomprimido y se obtiene la contraseña del siguiente nivel.

![OverTheWire - Bandit12](https://i.ibb.co/2P8FznQ/StLGf6x.png)

> Es importante mencionar que esta solución es un poco más vaga y menos precisa que la solución comun, pero puede resultar útil en situaciones en las que se desconoce la cantidad de veces que el archivo ha sido comprimido y en qué formato.
{:. prompt-warning}

## Level 13 → Level 14

La contraseña para el siguiente nivel se almacena en `/etc/bandit_pass/bandit14` y sólo puede ser leída por el usuario **bandit14**. Para este nivel, no obtienes la siguiente contraseña, pero obtienes una clave SSH privada que puede ser usada para ingresar al siguiente nivel.

> **Nota:** localhost es un nombre de host que se refiere a la máquina en la que estás trabajando.
{:. prompt-info}

### Resolución:

Lo único que debemos de hacer es conectarnos proporcionando la clave privada en vez de una contraseña tradicional.

```bash
ssh -i sshkey.private bandit14@localhost -p2220 -oStrictHostKeyChecking=no -oUserKnownHostsFile=/dev/null
```

El parámetro `-i` se utiliza para especificar la ruta a la clave privada, `-p` se utiliza para especificar el puerto al que se conecta y `-oStrictHostKeyChecking=no -oUserKnownHostsFile=/dev/null` se utiliza para omitir la verificación del host remoto.

![OverTheWire - Bandit13](https://s2.gifyu.com/images/18e641077ce31aed2.gif)

## Level 14 → Level 15

La contraseña del siguiente nivel se puede recuperar enviando la contraseña del nivel actual al puerto **`30000` en localhost**.

### Resolución:

Para solucionar este reto, podemos utilizar el comando `nc` (netcat) para enviar la contraseña del nivel actual al puerto 30000 en localhost. 

Netcat (también conocido como `nc`) es una herramienta de red que se utiliza para establecer conexiones TCP o UDP en un sistema operativo Unix o Unix-like. Permite crear, enviar y recibir paquetes de datos a través de la red. 

Si nos olvidamos o no lo apuntamos la contraseña del nivel actual donde estamos conectados, podemos visualizarlo de nuevo en el directorio `/etc/bandit_pass/` sumado al nombre de nuestro nivel (`/etc/bandit_pass/bandit<X>`).

```bash
cat /etc/bandit_pass/bandit14 | nc localhost 30000
```

Lo que estamos haciendo es imprimir la contraseña de nuestro nivel actual (bandit14) y lo estamos enviando al puerto 30000 en localhost.

![OverTheWire - Bandit14](https://i.ibb.co/p13LJB8/StLGf6x.png)

## Continuará...

### Autopwn.py

Este código de Python que te presento a continuación, es una implementación de automatización del proceso para obtener las contraseñas de los niveles 0 hasta el 25.

El objetivo de este código es permitir la ejecución automática de una serie de comandos almacenados en el archivo "commands.txt", para obtener las contraseñas de cada nivel y avanzar al siguiente nivel, hasta llegar al nivel 25.

La implementación se basa en la conexión SSH a través de la librería "paramiko" y el manejo de excepciones para garantizar la estabilidad del programa. Se utiliza la clase "SSH" para establecer la conexión SSH y ejecutar comandos en el servidor remoto.

Además, se ha implementado una función de manejo de señales para poder salir del programa de manera segura cuando sea necesario.

Repositorio: [J053/OverTheWire](https://github.com/J053/OverTheWire.git)

![Bandit - Autopwn.py](https://s2.gifyu.com/images/1df86db1509f4258b.gif)


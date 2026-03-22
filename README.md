# attacktive_directory
Máquina resuelta de *TryHackMe* en la que se trabaja la enumeración y *fingerprinting*, explotar el servicio *kerberos*, *crackeo* de contraseñas, realizar un *dump* de *hashes* y *pass the hash* para penetrar por medio del servicio SMB.
<div>
  <img src="https://img.shields.io/badge/-Kali-5e8ca8?style=for-the-badge&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/-Nmap-6933FF?style=for-the-badge&logo=nmap&logoColor=white" />
  <img src="https://img.shields.io/badge/-python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/-Kerbrute-005571?style=for-the-badge&logo=kerbrute&logoColor=white" />
  <img src="https://img.shields.io/badge/-john-F4B942?style=for-the-badge&logo=john&logoColor=white" />
  <img src="https://img.shields.io/badge/-smbclient-3B47C2?style=for-the-badge&logo=smbclient&logoColor=white" />
  <img src="https://img.shields.io/badge/-evilwinrm-BA0C2F?style=for-the-badge&logo=evil-winrm&logoColor=white" />
  <img src="https://img.shields.io/badge/-impacket-1E8E3E?style=for-the-badge&logo=impacket&logoColor=white" />
  <img src="https://img.shields.io/badge/-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" />
  <img src="https://img.shields.io/badge/-Netcat-F5455C?style=for-the-badge&logo=netcat&logoColor=white" />
</div>

## Objetivo

Explicar la realización del siguiente _Capture the flag_ perteneciente a la plataforma *TryHackMe*. Este desafío consiste en penetrar una máquina Windows con *Active Directory*. Para ello, se utilizarán distintos servicios presentes en la máqiuna como *kerberos* y *SMB* que nos permitirán conocer los *hashes* que a la postre nos den acceso a la máquina. Para conseguir las banderas también será necesrio realizar una escalada de privilegios.

## Que hemos aprendido?

- Realizar *fingerprinting* y enumeración de puertos.
- Enumerar usuarios del servicio *kerberos*.
- *Crackeo* de contraseñas.
- *Dump* de *hashes*.
- *Pass the hash*
- Escalada de privilegios.

## Herramientas utilizadas

- *Kali Linux*.
- Enumeración: *Nmap*, *kerbrute*, *Python*.
- Penetración: *Impacket-GetNPusers*, *Impacket-secretsdump*, *john the ripper*, *smbclient*, *Evil-WinRM*, *Bash*. 

## Steps

### Enumeración y fingerprinting

La máquina a vulnerar pertenece a la plataforma *TryHackMe*, la propia web te proporciona la IP de la máquina víctima. La conexión a esta se hace mediante una VPN que te proporciona THM, la cual asigna una nueva IP para que tu máquina interactúe con la máquina vulnerable.
El primer paso es lanzar **Nmap** sin *ping* (-Pn) y con los *scripts* por *default* de la herramienta (-sC) para que encuentre vulnerabilidades. Además, se quieren conocer las versiones que corren en cada puerto (-sV) y el sistema operativo (-O). Se indica que haga un escaneo sobre los 100 puertos más comúnes (-F), para evitar resolver nombres DNS se pasa el parámetro -n y con el objetivo de agilizar el escaneo se añade -T4.

<code>nmap -F -sV -O -sC -Pn -T4 -n 10.129.148.245</code>

<img width="1151" height="662" alt="Captura de pantalla 2026-03-14 130729" src="https://github.com/user-attachments/assets/bc794144-37eb-432b-b38d-3b173b6946c2" />
<img width="1162" height="634" alt="Captura de pantalla 2026-03-14 130835" src="https://github.com/user-attachments/assets/1a05d8f5-7623-42ec-b556-01e4db5efdfa" />

El comando devuelve varios puertos TCPs abiertos de los cuales se prestará especial atención a tres:  
- En el puerto 88 corre el servicio *Microsoft Windows Kerberos*, en un sistema *Windows*, servicio *SSH*.
- En el puerto 389 corre el servicio LDAP (Protocolo Ligero de Acceso a Directorios) encargado de manejar la forma en que las aplicaciones acceden y modifican la información en el *Active Directory*, por lo que se puede esperar que estamos ante un *Domain controller*.
- En el puerto 445 corre la versión 2.02 del servicio *SMB*.

Tambié se averigua que el nombre del dominio de NetBIOS es **THM-AD**, mientras que el nombre de dominio *DNS* es **spookysec.local**.

Para facilitar el uso de las siguientes herramientas se añade la IP de la víctima a nuestro DNS local para que haga la resolución DNS automáticamente.

<code>echo 10.129.148.245 spookysec.local >> /etc/hosts</code>

Lo primero será atacar el puerto 88 para enumerar los diferentes usuarios que hayan almacenados en el *Active Directory*. En este puerto corre el servicio, antes mencionado, *Kerberos*, el cual es un protocolo de autenticación de red que permite a usuarios y servicios demostrar su identidad de forma segura en una red, sin enviar contraseñas en texto plano. Se usa mucho en entornos empresariales, especialmente en *Microsoft Active Directory* y sistemas basados en *MIT Kerberos*. *Kerberos* funciona con tickets cifrados, en lugar de enviar la contraseña cada vez que accedes a un servicio, el sistema te da un ticket que prueba que ya te autenticaste. Así se evita que la contraseña viaje por la red. Como resúmen, *Active Directory* es el sistema de gestión y *kerberos* es el mecanismo de autenticación.

Pues bien, la herramienta a utilizar se llama **kerbrute** que permite enumerar usuarios y probar contraseñas en dominios de AD usando el protocolo *kerberos*. Se le pasa el parametro 'usernum' para la enumeración de usuarios, se informa el nombre del *Domain Controller* (--dc), el del dominio (-d), el número de hilos (-t) y una lista de usuairos con los que probar (users.txt).

<code>kerbrute userenum --dc spookysec.local -d spookysec.local -t 100 users.txt</code>

<img width="895" height="399" alt="Captura de pantalla 2026-03-15 211822" src="https://github.com/user-attachments/assets/ca970e51-a1ec-4f47-8ae0-576ff48c7bc1" />


### Vulnerabilidades explotadas

Con los usuarios devueltos, crearse una lista de usuarios válidos ('users_valid.txt') para encontrar el *ticket* del usuario correcto, es decir, aquel usuario que no necesita una autenticación válida antes de utilizar el protocolo *Kerberos*. Este método de ataque se conoce como 'ASREPRoasting' y para ello se utiliza la herramienta **Impacket**, la cual contiene varios programas de **Python***. En este caso se utiliza **GetBPUsers.py** desde la carpeta donde se almacena '/opt/impacket/examples'. Por medio de *Python3* se ejecuta el programa indicandole el dominio a tratar y la lista de usuarios con los que probar.

<code>python3 GetNPUsers.py spookysec.local/ -usersfile /root/users_valid.txt</code>

<img width="897" height="288" alt="Captura de pantalla 2026-03-15 213008" src="https://github.com/user-attachments/assets/710d7732-75b8-4c89-b267-294707fa7030" />

Como resultado se averigua que el usuario buscado era 'svc-admin' y obtenemos el *hash* de su contraseña. Este se guarda en un documento de texto ('hash'). Con la herramienta **John the ripper** y pasándole una 'wordlist' se puede descifrar la contraseña. La lista que se le pasa es la proporcionada por el propio reto de *Try Hack Me* para resolver esta máquina.

<code>john --wordlist=passwords.txt hash</code>

<img width="905" height="232" alt="Captura de pantalla 2026-03-15 213416" src="https://github.com/user-attachments/assets/b90dfec6-e21b-4282-995f-e4e9a1363cdc" />

La contraseña resulta ser 'management2005'.

Una vez conseguidas las credenciales, se dispone a ver cuales son los recursos del usuario via *SMB*. Este protocolo permite acceder a archivos e impresoras compartidas en red. Con la ayuda de la herramienta **smbclient** se entrará en los recursos compartidos del usuario 'svc-admin'. El comando -L listará estos recursos, también se indica el usuario cuyos recursos se quieren consultar y se informa del usuario (-U) y de la contraseña para conseguir el acceso.

<code>smbclient -L \\\\spookysec.local\\svc-admin -U svc-admin</code>

<img width="780" height="220" alt="Captura de pantalla 2026-03-15 214203" src="https://github.com/user-attachments/assets/883470ba-060a-4cc7-8096-2f1aff94cdba" />

De entre los recursos compartidos parece interesante el llamado 'backup'. Con la misma herramienta se puede consultar su contenido, donde se encuentra el documento backup_credentials.txt. Solo es necesario descargarlo con el comadno *get* para posteriormente leerlo.

<code>smbclient \\\\spookysec.local\\backup -U svc-admin</code>

<code>get backup_credentials.txt</code>

<code>cat backup_credentials.txt</code>

<img width="892" height="307" alt="Captura de pantalla 2026-03-15 214816" src="https://github.com/user-attachments/assets/f5fdccfe-80bd-424b-9274-b4ea02d47a80" />

Este documento contiene un string en base64 que al decodificarlo se obtiene un usuario y una contraseña. Este nuevo perfil debe almacenar más *hashes* de otros usuarios, pues es la cuenta de respaldo del *Active Directory*.

<code>echo YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw | base64 -d</code>

<code>backup@spookysec.local:backup2517860</code>

Para conseguir los *hashes* del resto de usuarios se utiliza otra herramienta de *Impacket*, en este caso **secretsdump.py**. Con el parámetro --just-dc se indica que el volcado de credenciales se hará sólo desde el *Domain Controller* utilizando el mecanismo de replicación de *Microsoft Active Directory*. Usa el método DRSUAPI (Directory Replication Service Remote Protocol). Las credenciales recuperadas en el anterior *hash* nos servirán para este propósito.

<code>secretsdump.py -just-dc backup@spookysec.local</code>

<img width="935" height="497" alt="Captura de pantalla 2026-03-16 172402" src="https://github.com/user-attachments/assets/d645713a-d11b-479a-b463-053fa25e1060" />

Ya sólo queda acceder al sistema con los usuarios (pedidos en este reto por parte de *Try Hack me*) 'scv-admin', 'backup' y 'Administrator' por medio de un ataque **pass the hash**. Este ataque consiste en usar directamente los *hashes* encontrados, sin necesidad de decifrarlos, para iniciar sesión con cualquiera de los usuarios. Esto se hace mediante la herramietna propuesta **Evil-WinRM** que nos dará acceso por medio de *Windows Remote Management*.
Para el usuario Administrator,por ejemplo, el comando sería el siguiente, donde se informan la IP (-i), el usuario (-u) y el *hash* (-H).

<code>evil-winrm -i spookysec.local -u Administrator -H 0e0363213e37b94221497260b0bcb4fc</code>

Una vez se tiene acceso al sistema con cada usuario, sólo es necesario buscar el archivo que contiene las banderas, las cuales se encontraban en el escritorio de cada uno de los usuarios.

<img width="706" height="737" alt="Captura de pantalla 2026-03-16 174254" src="https://github.com/user-attachments/assets/7f81d9cc-17a5-4734-ad40-34b9d1f3a67f" />

**flag Administrator: TryHackMe{4ctiveD1rectoryM4st3r}**

**flag svc-admin: TryHackMe{K3rb3r0s_Pr3_4uth}**

**flag backup: TryHackMe{B4ckM3UpSc0tty!}**

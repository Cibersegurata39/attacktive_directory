# attacktive_directory
Máquina resuelta de *TryHackMe* en la que se trabaja la enumeración y *fingerprinting*, explotar el servicio *kerberos*, *crackeo* de contraseñas, realizar un *dump* de *hashes* y *pass the hash* penetración por medio del servicio SMB
<div>
  <img src="https://img.shields.io/badge/-Kali-5e8ca8?style=for-the-badge&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/-Nmap-6933FF?style=for-the-badge&logo=nmap&logoColor=white" />
  <img src="https://img.shields.io/badge/-python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/-Kerbrute-005571?style=for-the-badge&logo=kerbrute&logoColor=white" />
  
  <img src="https://img.shields.io/badge/-php-777BB4?style=for-the-badge&logo=php&logoColor=white" />
  <img src="https://img.shields.io/badge/-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" />
  <img src="https://img.shields.io/badge/-Netcat-F5455C?style=for-the-badge&logo=netcat&logoColor=white" />
</div>

## Objetivo

Explicar la realización del siguiente _Capture the flag_ perteneciente a la plataforma *TryHackMe*. Este desafío consiste en penetrar una máquina Windows con *Active Directory*. Para ello, se utilizarán distintos servicios presentes en la máqiuna como *kerberos* y *SMB* que nos permitiran conocer lso *hashes* que a la postre nos den acceso ala máquina. Para conseguir la bandera también será necesrio realizar una escalada de privilegios.

## Que hemos aprendido?

- Realizar *fingerprinting* y enumeración de puertos.
- Enumerar usuarios del servicio *kerberos*.
- *Dump* de *hashes*.
- *Pass the hash*
- *Crackeo* de contraseñas.
- Escalada de privilegios.

## Herramientas utilizadas

- *Kali Linux*.
- Enumeración: *Nmap*, *kerbrute*, *Python*.
- Penetración: *Impacket-GetNPusers*, *Impacket-secretsdump*, *john the ripper*, *smbclient*, *Evil-WinRM*, *Bash*. 

## Steps

### Enumeración y fingerprinting

La máquina a vulnerar pertenece a la plataforma *TryHackMe*, la propia web te proporciona la IP de la máquina víctima. La conexión a esta se hace mediante una VPN que te proporciona THM y asigna una nueva IP para que tu máquina interactúe con la máquina vulnerable.
El primero paso es lanzar **Nmap** sin *ping* (-Pn) y con los *scripts* por *default* de la herramienta (-sC) para que encuentre vulnerabilidades. Además se quieren conocer las versiones que corren en cada puerto (-sV) y el sistema operativo (-O). Se indica que haga un escaneo sobre los 100 puertos más comúnes (-F). Para evitar resolver nombres DNS se pasa el parámetro -n y con el objetivo de agilizar el escaneo se añade -T4.

<code>nmap -F -sV -O -sC -Pn -t4 -n 10.129.148.245</code>

<img width="1151" height="662" alt="Captura de pantalla 2026-03-14 130729" src="https://github.com/user-attachments/assets/bc794144-37eb-432b-b38d-3b173b6946c2" />
<img width="1162" height="634" alt="Captura de pantalla 2026-03-14 130835" src="https://github.com/user-attachments/assets/1a05d8f5-7623-42ec-b556-01e4db5efdfa" />



El comando devuelve varios puertos TCPs abiertos de los cuales se prestará especial atención a tres:  
- En el puerto 88 corre el servicio *Microsoft Windows Kerberos*, en un sistema *Windows*, servicio *SSH*.
- En el puerto 389 corre 
- En el puerto 445 corre la versión *SMBv2.02*, en un sistema *Ubuntu*, servicio *HTTP*.




### Vulnerabilidades explotadas



**Flag: fleeb juice **

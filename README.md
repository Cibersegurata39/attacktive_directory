# attacktive_directory
Máquina resuelta de *TryHackMe* en la que se trabaja la enumeración y *fingerprinting*, *crackeo* de contraseñas, penetración por medio del servicio SMB
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

Explicar la realización del siguiente _Capture the flag_ perteneciente a la plataforma *TryHackMe*. Este desafío consiste en penetrar una máquina Windows con *Active Directory*. Para ello, se utilizarán distitnos servicios preentes en la máqiuna como *kerberos* y *SMB*. Para conseguir la bandera también será necesrio realizr una escalada de privilegios.

## Que hemos aprendido?

- Realizar *fingerprinting* y enumeración de puertos.
- Enumerar usuarios del servicio *kerberos*.
- 
- Poner en escucha los puertos de la máquina.
- Escalada de privilegios.

## Herramientas utilizadas

- *Kali Linux*.
- Enumeración: *Nmap*, *Dirsearch*, *Python*.
- Penetración: *Bash*, *PHP*, *Netcat*. 

## Steps

### Enumeración y fingerprinting

La máquina a vulnerar pertenece a la plataforma *TryHackMe*, la propia web te proporciona la IP de la máquina víctima. La conexión a esta se hace mediante una VPN que te proporciona THM y asigna una nueva IP para que tu máquina interactúe con la máquina vulnerable.
El primero paso es lanzar **Nmap** sin *ping* (-Pn) y con los *scripts* por *default* de la herramienta (-sC) para que encuentre vulnerabilidades. Además se quieren conocer las versiones que corren en cada puerto (-sV) y el sistema operativo (-O). No se indica, específicamente, que compruebe todos los puertos pues en este tipo de máquina no será necesario.

![Captura de pantalla 2025-04-04 131854](https://github.com/user-attachments/assets/8527c885-9cdd-499a-ba91-bebcf4b73f80)

El comando devuelve 2 puertos TCPs abiertos:  
- En el puerto 22 corre la versión *Openssh 8.2p1*, en un sistema *Ubuntu*, servicio *SSH*.  
- En el puerto 80 corre el servidor *Apache 2.4.41*, en un sistema *Ubuntu*, servicio *HTTP*.




### Vulnerabilidades explotadas



**Flag: fleeb juice **

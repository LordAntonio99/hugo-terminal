+++
title = "Curso completo de iptables"
date = "2022-02-03T15:41:27+01:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["firewall", "iptables", "redes", "ciberseguridad"]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
+++

# Contenidos
1. [Filtrado de paquetes](#filtrado-de-paquetes)
    1.1. [iptables](#iptables)
    1.2. [firewalls](#firewalls)
    1.3. [NAT](#nat)
    1.4. [SNAT](#snat)
    1.5. [DNAT](#dnat)


## Filtrado de paquetes
Se pueden realizar filtrados en base a diferentes capas del modelo TCP/IP. Sirven para poder controlar el tipo de tráfico que hay en nuestra red, permitir y denegar tráfico, y analizarlo.
### iptables
Puede trabajar con ip6tables, ebtables y arptables. Es el proyecto más utilizado y no se desarrollan ya nuevas funcionalidades.
El proyecto nftables esta desarrollado por los creadores de iptables, y es el sucesor de iptables, ip6tables, arptables y ebtables.
### Firewalls
Sistema de seguridad que inspecciona el tráfico de red, seleccionando el conforme a unas reglas establecidas.
El paquete de iptables proporciona algunas funcionalidades extra que no tienen los firewalls, como la NAT.
### NAT
Sirve principalmente para:
- Un equipo con dirección privada que quiere salir a internet (SNAT)
- Un equipo que quiere acceder a un servicio ubicado en una dirección privada (DNAT)
#### SNAT
Es la más comun para acceder a redes de Internet desde una red privada. Un dispositivo de NAT modifica la ip privada de origen por una ip pública. Puede utilizarse también entre redes privadas para no generar demasiadas reglas estáticas.
#### DNAT
Se utiliza para exponer un servicio que está en una ip privada. Un dispositivo NAT modifica la ip pública de una petición para un servicio, asignandole una ip privada de dentro del servidor.
## Esquema de red
![](/img/esquemaiptables.png)
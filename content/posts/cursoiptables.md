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
- [Filtrado de paquetes](#filtrado-de-paquetes)
    - [iptables](#iptables)
    - [firewalls](#firewalls)
    - [NAT](#nat)
    - [SNAT](#snat)
    - [DNAT](#dnat)


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
![Esquema de red](/hugo-terminal/img/esquemaiptables.png)
Puedes descargar la OVA para VirtualBox [aqui](https://drive.google.com/file/d/1cG1Vcl5aioI28A_fGbzV8-pcCXI5PRYv/view?usp=sharing)
## Tipos de tráfico
- Entrante: tráfico entrante con destino al equipo
- Salida: tráfico saliente con origen en el equipo
- Paso: tráfico que atraviesa el equipo, entra por una interfaz y sale por otra
## Tablas de iptables
- filter: cortafuegos, es la tabla por defecto.
- nat: para hacer NAT.
- mangle: marcar y modificar paquetes.
- raw: depurar seguimiento de conexión.
- security: usadas para MAC.
### Cadenas de filter
- INPUT: tráfico entrante
- OUTPUT: tráfico saliente
- FORWARD: tráfico de paso
### Cadenas de nat
- PREROUTING: relacionada con DNAT. Modifica la ip de destino.
- INPUT
- OUTPUT
- POSTROUTING: relacinada con SNAT. Modifica la ip de paquetes de salida.
### Tipo de tráfico y flujo
- Paquete del exterior con destino el equipo: PREROUTING de nat e INPUT de filter.
- Paquete originado en el equipo que sale: OUTPUT de nat, OUTPUT de filter y POSTROUTING de nat.
- Paquete que atraviesa el equipo: PREROUTING de nat, FORWARD de filter y POSTROUTING de nat.

## Sintaxis
### Políticas
```shell
iptables [-t TABLA] -P [CADENA] ACCEPT|DROP
```
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
### Listar reglas
```shell
iptables -L [CADENA] [-n] [-v] [--line-numbers]
iptables -S [CADENA] [-v]
```
### Añadir reglas
```shell
iptables -A [CADENA] [-p PROTOCOLO] [-s IP ORIGEN] [-d IP DESTINO] [-i INTERFAZ ENTRADA] [-o INTERFAZ SALIDA] [-j ACCEPT|DROP] #Inserta al final
iptables -I [CADENA] [-p PROTOCOLO] [-s IP ORIGEN] [-d IP DESTINO] [-i INTERFAZ ENTRADA] [-o INTERFAZ SALIDA] [-j ACCEPT|DROP] #Inserta al principio
```
### Borrar reglas
```shell
iptables -C [CADENA] [-p PROTOCOLO] [-s IP ORIGEN] [-d IP DESTINO] [-i INTERFAZ ENTRADA] [-o INTERFAZ SALIDA] [-j ACCEPT|DROP] #Comprueba si la regla existe
iptables -D [CADENA] [-p PROTOCOLO] [-s IP ORIGEN] [-d IP DESTINO] [-i INTERFAZ ENTRADA] [-o INTERFAZ SALIDA] [-j ACCEPT|DROP] #Borra una regla
```
### Limpiar
```shell
iptables -F [CADENA] # Borra todas las reglas
iptables -Z #Pone los contadores a 0
```

**Ejemplo de reglas para iptables personales**
```shell
# HTTP
iptables -A INPUT -i enp0s3 -p tcp --sport 80 -j ACCEPT
iptables -A OUTPUT -o enp0s3 -p tcp --dport 80 -j ACCEPT
# HTTPS
iptables -A INPUT -i enp0s3 -p tcp --sport 443 -j ACCEPT
iptables -A OUTPUT -o enp0s3 -p tcp --dport 443 -j ACCEPT
# DNS
iptables -A INPUT -i enp0s3 -p udp --sport 53 -j ACCEPT
iptables -A OUTPUT -o enp0s3 -p udp --dport 53 -j ACCEPT
```
## Configuración del firewall perimetral 1
```shell
# Activar en el router el reenvio de paquetes entre interfaces
echo 1 > /proc/sys/net/ipv4/ip_forward
# Borrar todas las reglas
iptables -F
# Politicas por defecto en DROP
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
# Permitir el ping por la NAT
iptables -A OUTPUT -o enp0s3 -p icmp -j ACCEPT
iptables -A INPUT -i enp0s3 -p icmp -j ACCEPT
# Permitir el ping del servidor a las demas maquinas y viceversa
iptables -A OUTPUT -o enp0s8 -s 192.168.100.0/24 -p icmp -j ACCEPT
iptables -A INPUT -i enp0s8 -s 192.168.100.0/24 -p icmp -j ACCEPT
iptables -A OUTPUT -o enp0s9 -s 192.168.200.0/24 -p icmp -j ACCEPT
iptables -A INPUT -i enp0s9 -s 192.168.200.0/24 -p icmp -j ACCEPT
# Permitir el ping entre las máquinas
iptables -A FORWARD -i enp0s8 -o enp0s9 -p icmp -J ACCEPT
iptables -A FORWARD -o enp0s8 -i enp0s9 -p icmp -J ACCEPT
# Redirigir el trafico DNS
iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.100.0/24 -p udp --dport 53 -j ACCEPT
iptables -A FORWARD -o enp0s8 -i enp0s3 -d 192.168.100.0/24 -p udp --sport 53 -j ACCEPT
# Aceptar trafico web
iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.100.0/24 -p tcp --dport 80 -j ACCEPT
iptables -A FORWARD -o enp0s8 -i enp0s3 -d 192.168.100.0/24 -p tcp --sport 80 -j ACCEPT
iptables -A FORWARD -i enp0s9 -o enp0s3 -s 192.168.200.0/24 -p tcp --dport 443 -j ACCEPT
iptables -A FORWARD -o enp0s9 -i enp0s3 -d 192.168.200.0/24 -p tcp --sport 443 -j ACCEPT
iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.100.0/24 -p tcp --dport 80 -j ACCEPT
iptables -A FORWARD -o enp0s8 -i enp0s3 -d 192.168.100.0/24 -p tcp --sport 80 -j ACCEPT
iptables -A FORWARD -i enp0s9 -o enp0s3 -s 192.168.200.0/24 -p tcp --dport 443 -j ACCEPT
iptables -A FORWARD -o enp0s9 -i enp0s3 -d 192.168.200.0/24 -p tcp --sport 443 -j ACCEPT
#SNAT
iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o enp0s3 -j MASQUERADE
#DNAT
iptables -t nat -A PREROUTING -i enp0s8 -p tcp --dport 80 -j DNAT --to 192.168.200.2
```


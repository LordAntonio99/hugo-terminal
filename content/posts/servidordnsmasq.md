+++
title = "Servidordnsmasq"
date = "2022-02-03T13:16:32+01:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["", ""]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
+++

## Instalación

Para instalar el servicio, utilizaremos el siguiente comando:

```shell
apt-get install dnsmasq
```

Una vez instalado el paquete, procederemos a levantar el servicio con el siguiente comando:

```shell
systemctl start dnsmasq
```

Con esto tenemos lista la parte de instalación y preparación del servicio.

## Servidor DNS Caché

A continuación configuraremos el servidor para que el servidor dnsmasq pueda ser un servidor caché DNS. Para ello tendremos que modificar el archivo "/etc/resolv.conf", donde especificaremos que servidores de DNS externos van a actuar en caso de que el propio servidor no tenga una respuesta. La configuración que utilizaremos será con los 2 servidores DNS de google, de forma que el archivo quede de la siguiente manera:

```bash
nameserver 8.8.8.8
nameserver 8.8.4.4
```

Una vez modificado este archivo, reiniciamos el servicio de la red con el comando:

```bash
systemctl restart networking
```

Para comprobar su funcionamiento, vamos a ejecutar un comando de "nslookup" con una dirección de internet para comprobar que se resuelve y que servidor lo hace:

```bash
nslookup www.google.es
Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:
Name:   www.google.es
Address: 142.250.184.163
Name:   www.google.es
Address: 2a00:1450:4003:80c::2003
```

Con esto la configuración del servidor está completada, ahora solo falta modificar el archivo "/etc/resolv.conf" del resto de los ordenadores de la red para que su servidor DNS sea el de dnsmasq. Para ello pondremos la siguiente configuración:

```shell
nameserver 10.0.0.5
```

La ip seleccionada es la del servidor.

A continuación podemos probar a resolver nombres desde cualquier otra máquina, dándonos el siguiente resultado:

```shell
nslookup www.iespabloserrano.com
Server:         10.0.0.5
Address:        10.0.0.5#53

Non-authoritative answer:
Name:   www.iespabloserrano.com
Address: 82.98.135.44
```

## Servidor DNS maestro
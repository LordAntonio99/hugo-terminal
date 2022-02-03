+++
title = "Trabajo de servidores DNS"
date = "2022-02-03T13:16:32+01:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["redes", "dns"]
keywords = ["", ""]
description = "Trabajo completo de DNS"
showFullContent = false
readingTime = true
+++

# Contenidos
- [Servidor de DNS en Linux](#servidor-de-dns-en-linux)
  * [Configuración inicial](#configuración-inicial)
  * [Instalación del servicio](#instalación-del-servicio)
  * [Punto 1](#punto-1)
    + [A. Creación del una zona para el dominio](#a-creación-del-una-zona-para-el-dominio)
    + [B. Crear una zona de resolución inversa](#b-crear-una-zona-de-resolución-inversa)
    + [C. Nombres FQHN](#c-nombres-fqhn)
    + [D. Servidor web y ftp](#d-servidor-web-y-ftp)
  * [Punto 2](#punto-2)
    + [A. Configuración de las zonas](#a-configuración-de-las-zonas)
    + [B. Reiniciar los servidores](#b-reiniciar-los-servidores)
    + [C. Modificar el numero de registro](#c-modificar-el-numero-de-registro)
    + [D. Transferencias completas:](#d-transferencias-completas-)
    + [E. Consultas con dig](#e-consultas-con-dig)
    + [F. Configura un cliente para que utilice los 2 servidores](#f-configura-un-cliente-para-que-utilice-los-2-servidores)
  * [Punto 3](#punto-3)
    + [Instalación](#instalación)
    + [Servidor DNS Caché](#servidor-dns-caché)
    + [Servidor DNS maestro](#servidor-dns-maestro)
  * [Punto 4](#punto-4)
    + [Configuración DHCP en DNSMASQ](#configuración-dhcp-en-dnsmasq)

# Servidor de DNS en Linux

## Configuración inicial

| PC     | Interfaz | Red  | Dirección    |
| ------ | -------- | ---- | ------------ |
| PC01   | enp0s3   | NAT  | 10.0.2.15/24 |
| PC01   | enp0s8   | red2 | 10.0.0.2/24  |
| PC02   | enp0s3   | NAT  | 10.0.2.15/24 |
| PC02   | enp0s8   | red1 | 10.0.1.2/24  |
| Router | enp0s3   | NAT  | 10.0.2.15/24 |
| Router | enp0s8   | red1 | 10.0.1.1/24  |
| Router | enp0s9   | red2 | 10.0.0.1/24  |
| DNS    | enp0s3   | NAT  | 10.0.2.15/24 |
| DNS    | enp0s8   | red2 | 10.0.0.3/24  |

## Instalación del servicio

Lo primero que haremos será crear un dominio, para lo que utilizaremos el paquete de bind9, que además funcionará como servidor DNS cuando se configure. 

Para instalar dicho paquete y sus todos sus paquetes sugeridos según la documentación oficial, ejecutaremos el siguiente comando:

```shell
sudo apt install bind9 bind9-doc resolvconf python-ply-doc
```

Una vez instalado, iremos a la carpeta de este paquete para ver sus archivos de configuración y podremos ver todos los archivos que nos ha generado:

```shell
cd /etc/bind
ls -l
total 48
-rw-r--r-- 1 root root 1991 Oct 25 07:29 bind.keys
-rw-r--r-- 1 root root  237 Oct 25 07:29 db.0
-rw-r--r-- 1 root root  271 Oct 25 07:29 db.127
-rw-r--r-- 1 root root  237 Oct 25 07:29 db.255
-rw-r--r-- 1 root root  353 Oct 25 07:29 db.empty
-rw-r--r-- 1 root root  270 Oct 25 07:29 db.local
-rw-r--r-- 1 root bind  463 Oct 25 07:29 named.conf
-rw-r--r-- 1 root bind  498 Oct 25 07:29 named.conf.default-zones
-rw-r--r-- 1 root bind  165 Oct 25 07:29 named.conf.local
-rw-r--r-- 1 root bind  846 Oct 25 07:29 named.conf.options
-rw-r----- 1 bind bind  100 Jan 31 06:13 rndc.key
-rw-r--r-- 1 root root 1317 Oct 25 07:29 zones.rfc1918
```

## Punto 1

### A. Creación del una zona para el dominio

Lo primero que haremos, será configurar los forwarders de bind9 para que utilicen servidores DNS públicos y puedan ser funcionales para el dominio.

Para esto lo primero que tenemos que hacer será configurar el archivo "named.conf.options", no sin antes crear una copia de seguridad por si acaso hay algún tipo de fallo:

```shell
cp named.conf.options named.conf.options.copia
```

A continuación configuraremos los servidores DNS de Google en el archivo original. Lo único que haremos será descomentar la sección de forwarders y escribir los que nosotros queramos:

```html
options {
        directory "/var/cache/bind";

        forwarders {
                8.8.8.8;
                1.1.1.1;
        };

        dnssec-validation auto;

        listen-on-v6 { any; };
};
```

A continuación configuraremos bind9 para las resoluciones locales, para lo que tendremos que modificar el archivo "named.conf.local", que al igual que con el anterior, se le realizará la correspondiente copia por si acaso hay errores:

```shell
cp named.conf.local named.conf.local.copia
```

A continuación, procederemos a crear una zona con el dominio que queramos, en este caso "asir2.com":

```html
zone "asir2.com" {
        type master;
        file "/etc/bind/db.asir2.com";
};
```

Como hemos especificado en el anterior código, será un servidor DNS maestro, y tendrá su configuración en el archivo "db.asir2.com"

```shell
cp db.local db.asir2.com
```

A continuación modificaremos dicho archivo para ajustarlo a la configuración que queramos:

```shell
;
; BIND data file for local asir2.com
;
$TTL    604800
@       IN      SOA     asir2.com. root.asir2.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      asir2.com.
@       IN      A       10.0.0.3
@       IN      AAAA    ::1
ns      IN      A       10.0.0.3
pc01    IN      A       10.0.0.2
router  IN      A       10.0.0.1
pc02    IN      A       10.0.1.2
```

En este archivo, habremos modificado la línea que trae por defecto el nombre de servidor de localhost por "asir2.com" para que de esta forma quede asegurado nuestro dominio.

Una vez realizadas las configuraciones, procederemos a reiniciar el servicio para que todos los cambios se apliquen.

```shell
systemctl restart bind9
```

Si ahora probamos a utilizar el comando host buscando cualquier nombre de la red veremos como nos aparece su ip tal y como se ha especificado en el anterior archivo.

### B. Crear una zona de resolución inversa

Para crear la zona de resolución inversa, añadiremos una nueva al archivo "named.conf.local" con los siguientes datos:

```html
zone "0.10.in-addr.arpa" {
        type master;
        file "/etc/bind/db.asir2inv.com";
};
```

Lo siguiente que haremos será crear la nueva base de datos para guardar las diferentes direcciones ip:

```shell
cp db.127 db.asir2inv.com;
```

Con la nueva base de datos creada a partir del archivo "db.127", el cual tiene la estructura de la resolución inversa, modificaremos los diferentes parámetros de forma que quede así:

```html
  GNU nano 5.4                                                    db.asir2inv.com                                                             
;
; BIND reverse data file for asir2inv.com
;
$TTL    604800
@       IN      SOA     ns.asir2inv.com. root.asir2inv.com. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.
3.0     IN      PTR     ns.asir2inv.com.
2.0     IN      PTR     pc01.asir2inv.com.
1.0     IN      PTR     router.asir2inv.com.
2.1     IN      PTR     pc02.asir2inv.com.
```

### C. Nombres FQHN

Los nombres asignados para todas las interfaces son:

| PC     | Interfaz | Red  | Dirección   | FQDN             |
| ------ | -------- | ---- | ----------- | ---------------- |
| PC01   | enp0s8   | red2 | 10.0.0.2/24 | pc01.asir2.com   |
| PC02   | enp0s8   | red1 | 10.0.1.2/24 | pc02.asir2.com   |
| Router | enp0s8   | red1 | 10.0.1.1/24 | router.asir2.com |
| Router | enp0s9   | red2 | 10.0.0.1/24 | router.asir2.com |
| DNS    | enp0s8   | red2 | 10.0.0.3/24 | ns.asir2.com     |

Los registros de estos son:

```html
ns      IN      A       10.0.0.3
pc01    IN      A       10.0.0.2
router  IN      A       10.0.0.1
router  IN      A       10.0.1.1
pc02    IN      A       10.0.1.2
```

Para la zona de resolución inversa es la siguiente:

| PC     | Interfaz | Red  | Dirección             | FQDN                |
| ------ | -------- | ---- | --------------------- | ------------------- |
| PC01   | enp0s8   | red2 | 2.0.0.10.in-addr.arpa | pc01.asir2inv.com   |
| PC02   | enp0s8   | red1 | 2.1.0.10.in-addr.arpa | pc02.asir2inv.com   |
| Router | enp0s8   | red1 | 1.1.0.10.in-addr.arpa | router.asir2inv.com |
| Router | enp0s9   | red2 | 1.0.0.10.in-addr.arpa | router.asir2inv.com |
| DNS    | enp0s8   | red2 | 3.0.0.10.in-addr.arpa | ns.asir2inv.com     |

Los registros de esta zona son los siguientes:

```html
3.0     IN      PTR     ns.asir2inv.com.
2.0     IN      PTR     pc01.asir2inv.com.
1.0     IN      PTR     router.asir2inv.com.
1.1     IN      PTR     router.asir2inv.com.
2.1     IN      PTR     pc02.asir2inv.com.
```

### D. Servidor web y ftp

Se agregan los nuevos registros al servidor DNS normal:

```html
;
; BIND data file for local asir2.com
;
$TTL    604800
@       IN      SOA     asir2.com. root.asir2.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      asir2.com.
@       IN      A       10.0.0.3
@       IN      AAAA    ::1
ns      IN      A       10.0.0.3
pc01    IN      A       10.0.0.2
router  IN      A       10.0.0.1
router  IN      A       10.0.1.1
pc02    IN      A       10.0.1.2
web     IN      A       10.0.0.4
ftp     IN      A       10.0.0.5
```

Y al servidor de DNS inverso:

```html
;
; BIND reverse data file for asir2inv.com
;
$TTL    604800
@       IN      SOA     ns.asir2inv.com. root.asir2inv.com. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.
3.0     IN      PTR     ns.asir2inv.com.
2.0     IN      PTR     pc01.asir2inv.com.
1.0     IN      PTR     router.asir2inv.com.
1.1     IN      PTR     router.asir2inv.com.
2.1     IN      PTR     pc02.asir2inv.com.
4.0     IN      PTR     web.asir2inv.com.
5.0     IN      PTR     ftp.asir2inv.com.
```

## Punto 2

### A. Configuración de las zonas

```html
zone "asir2.com" {
        type slave;
        masters {10.0.0.3;};
        file "/etc/bind/db.asir2.com";
};

zone "0.10.in-addr.arpa" {
        type slave;
        masters {10.0.0.3;};
        file "/etc/bind/db.asir2inv.com";
};
```

### B. Reiniciar los servidores

Servidor maestro:

```html
root@dns:/etc/bind# systemctl status bind9
● named.service - BIND Domain Name Server
     Loaded: loaded (/lib/systemd/system/named.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-02-02 13:43:22 EST; 4s ago
       Docs: man:named(8)
   Main PID: 1665 (named)
      Tasks: 4 (limit: 1117)
     Memory: 12.5M
        CPU: 23ms
     CGroup: /system.slice/named.service
             └─1665 /usr/sbin/named -f -u bind

Feb 02 13:43:23 dns named[1665]: FORMERR resolving './NS/IN': 202.12.27.33#53
Feb 02 13:43:23 dns named[1665]: DNS format error from 192.33.4.12#53 resolving ./NS for <unknown>: non-improving referral
Feb 02 13:43:23 dns named[1665]: FORMERR resolving './NS/IN': 192.33.4.12#53
Feb 02 13:43:23 dns named[1665]: DNS format error from 192.112.36.4#53 resolving ./NS for <unknown>: non-improving referral
Feb 02 13:43:23 dns named[1665]: FORMERR resolving './NS/IN': 192.112.36.4#53
Feb 02 13:43:23 dns named[1665]: DNS format error from 192.5.5.241#53 resolving ./NS for <unknown>: non-improving referral
Feb 02 13:43:23 dns named[1665]: FORMERR resolving './NS/IN': 192.5.5.241#53
Feb 02 13:43:23 dns named[1665]: DNS format error from 199.7.83.42#53 resolving ./NS for <unknown>: non-improving referral
Feb 02 13:43:23 dns named[1665]: FORMERR resolving './NS/IN': 199.7.83.42#53
Feb 02 13:43:23 dns named[1665]: resolver priming query complete
```

Esclavo:

```html
root@dnsslave:/etc/bind# systemctl status bind9
● named.service - BIND Domain Name Server
     Loaded: loaded (/lib/systemd/system/named.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-02-02 13:48:54 EST; 1min 7s ago
       Docs: man:named(8)
   Main PID: 1867 (named)
      Tasks: 5 (limit: 1117)
     Memory: 18.7M
        CPU: 75ms
     CGroup: /system.slice/named.service
             └─1867 /usr/sbin/named -f -u bind

Feb 02 13:49:52 dnsslave named[1867]: automatic empty zone: EMPTY.AS112.ARPA
Feb 02 13:49:52 dnsslave named[1867]: automatic empty zone: HOME.ARPA
Feb 02 13:49:52 dnsslave named[1867]: configuring command channel from '/etc/bind/rndc.key'
Feb 02 13:49:52 dnsslave named[1867]: configuring command channel from '/etc/bind/rndc.key'
Feb 02 13:49:52 dnsslave named[1867]: reloading configuration succeeded
Feb 02 13:49:52 dnsslave named[1867]: scheduled loading new zones
Feb 02 13:49:52 dnsslave named[1867]: managed-keys-zone: Unable to fetch DNSKEY set '.': operation canceled
Feb 02 13:49:52 dnsslave named[1867]: resolver priming query complete
Feb 02 13:49:52 dnsslave named[1867]: any newly configured zones are now loaded
Feb 02 13:49:52 dnsslave named[1867]: running
```

### C. Modificar el numero de registro

En los 2 se ha puesto en el serial el número 7.

```html
@       IN      SOA     asir2.com. root.asir2.com. (
                              7         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
```

La salida del log muestra lo siguiente:

```html
root@dnsslave:/etc/bind# systemctl status bind9
● named.service - BIND Domain Name Server
     Loaded: loaded (/lib/systemd/system/named.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-02-02 13:55:37 EST; 1s ago
       Docs: man:named(8)
   Main PID: 2021 (named)
      Tasks: 4 (limit: 1117)
     Memory: 12.3M
        CPU: 26ms
     CGroup: /system.slice/named.service
             └─2021 /usr/sbin/named -f -u bind

Feb 02 13:55:37 dnsslave named[2021]: DNS format error from 193.0.14.129#53 resolving ./NS for <unknown>: non-improving referral
Feb 02 13:55:37 dnsslave named[2021]: FORMERR resolving './NS/IN': 193.0.14.129#53
Feb 02 13:55:37 dnsslave named[2021]: resolver priming query complete
Feb 02 13:55:37 dnsslave named[2021]: zone 0.10.in-addr.arpa/IN: Transfer started.
Feb 02 13:55:37 dnsslave named[2021]: transfer of '0.10.in-addr.arpa/IN' from 10.0.0.3#53: connected using 10.0.0.4#49701
Feb 02 13:55:37 dnsslave named[2021]: zone 0.10.in-addr.arpa/IN: transferred serial 7
Feb 02 13:55:37 dnsslave named[2021]: transfer of '0.10.in-addr.arpa/IN' from 10.0.0.3#53: Transfer status: success
Feb 02 13:55:37 dnsslave named[2021]: transfer of '0.10.in-addr.arpa/IN' from 10.0.0.3#53: Transfer completed: 1 messages, 10 records, 318 by>Feb 02 13:55:37 dnsslave named[2021]: zone 0.10.in-addr.arpa/IN: sending notifies (serial 7)
```

### D. Transferencias completas:

```html
Feb 02 13:55:37 dnsslave named[2021]: zone 0.10.in-addr.arpa/IN: Transfer started.
Feb 02 13:55:37 dnsslave named[2021]: transfer of '0.10.in-addr.arpa/IN' from 10.0.0.3#53: connected using 10.0.0.4#49701
Feb 02 13:55:37 dnsslave named[2021]: zone 0.10.in-addr.arpa/IN: transferred serial 7
Feb 02 13:55:37 dnsslave named[2021]: transfer of '0.10.in-addr.arpa/IN' from 10.0.0.3#53: Transfer status: success
Feb 02 13:55:37 dnsslave named[2021]: transfer of '0.10.in-addr.arpa/IN' from 10.0.0.3#53: Transfer completed: 1 messages, 10 records, 318 by>Feb 02 13:55:37 dnsslave named[2021]: zone 0.10.in-addr.arpa/IN: sending notifies (serial 7)

```

### E. Consultas con dig

Consulta desde pc01:

```html
root@pc01:/home/alu# dig @10.0.0.4 ns.asir2.com

; <<>> DiG 9.16.22-Debian <<>> @10.0.0.4 ns.asir2.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64110
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 2b1a042f6d8709ad0100000061fad3eaf7478968c4ee03ef (good)
;; QUESTION SECTION:
;ns.asir2.com.                  IN      A

;; ANSWER SECTION:
ns.asir2.com.           604800  IN      A       10.0.0.3

;; Query time: 0 msec
;; SERVER: 10.0.0.4#53(10.0.0.4)
;; WHEN: Wed Feb 02 13:56:42 EST 2022
;; MSG SIZE  rcvd: 85
```

Consulta desde pc02:

```html
root@pc02:/home/alu# dig @10.0.0.3 pc01.asir2.com

; <<>> DiG 9.16.22-Debian <<>> @10.0.0.3 pc01.asir2.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62022
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 2343e31b1c2b68660100000061fad436c65a3e43b4168f94 (good)
;; QUESTION SECTION:
;pc01.asir2.com.                        IN      A

;; ANSWER SECTION:
pc01.asir2.com.         604800  IN      A       10.0.0.2

;; Query time: 0 msec
;; SERVER: 10.0.0.3#53(10.0.0.3)
;; WHEN: Wed Feb 02 13:57:59 EST 2022
;; MSG SIZE  rcvd: 87
```

Como se puede ver, ambos servidores, el 10.0.0.3 y el 10.0.0.4 pueden resolver sin problemas.

### F. Configura un cliente para que utilice los 2 servidores

Modificaremos el archivo "/etc/resolv.conf" del pc01.

```html
nameserver 10.0.0.3
nameserver 10.0.0.4
```

## Punto 3
### Instalación

Para instalar el servicio, utilizaremos el siguiente comando:

```shell
apt-get install dnsmasq
```

Una vez instalado el paquete, procederemos a levantar el servicio con el siguiente comando:

```shell
systemctl start dnsmasq
```

Con esto tenemos lista la parte de instalación y preparación del servicio.

### Servidor DNS Caché

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

### Servidor DNS maestro
Para realizar la configuración como un servidor maestro, vamos a tener que escribir los diferentes hosts dentro del archivo "/etc/hosts", en el que escribiremos las siguientes entradas:
```shell
10.0.0.5        dns.asir2.com
10.0.0.2        pc01.asir2.com
10.0.0.1        router.asir2.com
10.0.1.1        router.asir2.com
10.0.1.2        pc02.asir2.com
```
Una vez guardado el archivo con los cambios, simplemente tendremos que reiniciar el servicio con el comando:
```shell
systemctl restart dnsmasq
```
Si ahora realizamos una consulta desde cualquier ordenador de la red, esto es lo que obtenemos:
```shell
nslookup pc01.asir2.com
Server:         10.0.0.5
Address:        10.0.0.5#53

Name:   pc01.asir2.com
Address: 10.0.0.2


nslookup router.asir2.com
Server:         10.0.0.5
Address:        10.0.0.5#53

Name:   router.asir2.com
Address: 10.0.1.1
Name:   router.asir2.com
Address: 10.0.0.1
```

## Punto 4
### Configuración DHCP en DNSMASQ
Para poder utilizar nuestro servidor DNS dinámico como un servidor DHCP, lo que tenemos que hacer es configura varios archivos, siendo el primero el "/etc/dnsmasq.conf" para establecer nuestro rango de direcciones ip para el servicio DHCP.
Lo único que tendremos que hacer será agregar la siguiente línea al archivo indicado:
```sql
dhcp-range=10.0.0.10,10.0.0.100,24h
```
Una vez configurado, reiniciamos el servicio y probamos a conectar un cliente, en este caso el pc01 que teníamos anteriormente en la misma red. Modificaremos su interfaz de red local para que obtenga dhcp, y tras el reinicio del servicio obtenemos el siguiente resultado:
```shell
enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
link/ether 08:00:27:c0:c8:27 brd ff:ff:ff:ff:ff:ff
inet 10.0.0.68/24 brd 10.0.0.255 scope global secondary dynamic enp0s8
    valid_lft 86381sec preferred_lft 86381sec
inet6 fe80::a00:27ff:fec0:c827/64 scope link 
valid_lft forever preferred_lft forever
```
Como podemos observar, se nos ha concedido la ip 10.0.0.68, por un periodo de 86400 segundos, es decir, 24 horas tal y como se especificón en la configuración del servidor.
Si accedemos al archivo "/var/lib/dhcp/dhclient.enp0s8.leases" nos mostrará todas las concesiones que se han realizado por el servidor dns, en este caso nos muestra lo siguiente:
```shell
cat /var/lib/dhcp/dhclient.enp0s8.leases 
default-duid "\000\001\000\001)\216\235\345\010\000'\300\310'";
lease {
  interface "enp0s8";
  fixed-address 10.0.0.68;
  option subnet-mask 255.255.255.0;
  option routers 10.0.0.5;
  option dhcp-lease-time 86400;
  option dhcp-message-type 5;
  option domain-name-servers 10.0.0.5;
  option dhcp-server-identifier 10.0.0.5;
  option dhcp-renewal-time 43200;
  option broadcast-address 10.0.0.255;
  option dhcp-rebinding-time 75600;
  option host-name "pc01";
  renew 5 2022/02/04 00:26:45;
  rebind 5 2022/02/04 11:07:29;
  expire 5 2022/02/04 14:07:29;
}
```
Si realizamos una ejecución del comando "nslookup" con el mismo nombre que tiene el nuevo ordenador, nos devolverá lo siguiente:
```shell
nslookup pc01.asir2.com 10.0.0.5
Server:         10.0.0.5
Address:        10.0.0.5#53

Name:   pc01.asir2.com
Address: 10.0.0.68
```
Y si ejecutamos consultas desde el pc01, esto es lo que obtenemos:
```shell
nslookup router.asir2.com
Server:         10.0.0.5
Address:        10.0.0.5#53

Name:   router.asir2.com
Address: 10.0.1.1
Name:   router.asir2.com
Address: 10.0.0.1
```
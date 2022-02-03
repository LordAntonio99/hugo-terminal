+++
title = "Servidores DHCP Teoría"
date = "2022-02-03T13:43:56+01:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["redes", "dhcp"]
keywords = ["", ""]
description = "Teoría completa del examen de DHCP para Servicios y Redes"
showFullContent = false
readingTime = true
+++

# Servidores DHCP

## Introducción

DHCP es un protocolo de configuración dinámica de hosts, con el que asignamos Ips dentro de un rango que establecemos.

Su principal función es que los dispositivos de una red obtengan su configuración de internet de forma automática.

Los hosts que proporcionan servicios de red no deberían usar esta configuración, ya que dependen de ella y pueden desconectarse o causar errores.

## Componentes de un servidor DHCP

Sus componentes son:

- Servidor DHCP: asigna la configuración a los clientes.

- Clientes DHCP: realizan las peticiones al servidor y configuran sus parámetros con los que el servidor les devuelve.

- Protocolo DHCP: conjunto de normas y reglas de diálogo entre el servidor y el cliente.

- Agentes de retransmisión DHCP: escuchan las peticiones de los clientes y las retransmiten a los servidores.

## Tipos de asignación

Dentro de la asignación de Ips de DHCP, podemos configurar diferentes tipos:

- Manual o estática: se trata de direcciones concretas para máquinas las cuales van a obtener siempre la misma configuración, puesto que estarán reservadas.

- Dinámica: asigna una Ip aleatoria dentro del rango durante un tiempo establecido (lease time)

- Automática: se asigna una dirección Ip al cliente hasta que este se desconecta y la libera, con un tiempo ilimitado.

El rango de direcciones es el intervalo consecutivo de estas, las cuales son válidas y están disponibles para ser asignadas.

De la misma manera que podemos definir una Ip para que se asigne de forma estática, también podemos excluirlas para que no se puedan conceder a nuevas máquinas. Estas suelen ser reservadas para servidores, routers...

El tiempo de concesión o lease time es el tiempo que un cliente DHCP mantiene sus datos de configuración otorgados por el servidor. Una vez vencido este tiempo, puede solicitar su renovación o una desconexión.

## Servidores DHCP

Permiten asignar la configuración de red al resto de máquinas presentes en la red cuando estos inician sus interfaces de red. Por defecto, el puerto de escucha es el 67 mediante UDP.

Los parámetros que estos servidores configuran son:

- Dirección Ip

- Máscara de subred

- Puerta de enlace

- Servidores DNS

- Nombre DNS

Algunos de los ejemplos de servidores DHCP son:

- ISC DHCP: servidor para Linux

- Servidor DHCP de Microsoft: principalmente para Windows Server

- Servidor DHCP integrados en routers: en ocasiones un router particular puede incluir este servicio.

## Clientes DHCP

Son los que realizan peticiones al servidor DHCP y configuran sus parámetros con los que reciben del propio servidor. Para estas tareas, utilizan el puerto 68 mediante UDP. Estos clientes vienen integrados por defecto en los sistemas operativos.

## Tipos de opciones

Los parámetros que el servidor envía a los clientes se pueden establecer a diferentes niveles:

- Opciones de servidor: se envían a todos los clientes.

- Opciones de ámbito: se envían a todos los clientes del ámbito y sobrescriben a los del servidor.

- Opciones de clase: se envían a los clientes de pendiendo de a que clase pertenecen.

- Opciones de equipo: se define para un equipo concreto mediante una reserva. Este tipo de configuración sobrescribe a todas las demás.

## Protocolo DHCP

Determina el conjunto de normas y reglas en base a las cuales dialogan os clientes y servidores.

El formato de un mensaje DHCP tiene una parte fija que aparece en todos los mensajes, aunque irá cambiando conforme se obtengan opciones específicas.

## Funcionamiento

Los pasos que sigue el servicio DHCP son los siguientes:

1. Un cliente DHCP envía una solicitud en forma de broadcast a través de la red para solicitar una configuración.

2. Todos los servidores alcanzados por la solicitud responden al cliente.

3. El cliente acepta una de estas, haciéndoselo saber al servidor elegido.

4. El servidor le otorga la información requerida.

5. Esta información se mantiene asociada al cliente mientras éste no desactive su interfaz o no expire el plazo de concesión.

6. Cuando expira el plazo, el cliente puede pedir una renovación.

### Obtención de una concesión

Sigue los siguientes pasos:

1. Descubrimiento DHCP ( DHCPDISCOVER ): el cliente manda la señal de broadcast desde el puerto 68 a todos los puertos 67, que serán donde los servidores escuchen.

2. Oferta DHCP ( DHCPOFFER ): los servidores responden al puerto 68 del cliente, enviándole una ip y algunos parámetros de configuración. Durante este paso, la dirección ip enviada se reserva.

3. Solicitud DHCP ( DHCPREQUEST ): el cliente recibe una o más ofertas de servidores y elige la que mejor funcione y envía un mensaje DHCPREQUEST poniendo como destino al servidor elegido. En caso de no recibir la respuesta con DHCPOFFER, reenvía un nuevo mensaje.

4. Reconocimiento DHCP ( DHCPACK ): Si el mensaje DHCPREQUEST no contiene su dirección, el servidor la considera rechazada, en caso de que sea correcta y la Ip siga disponible, el servidor enviará un mensaje DHCPACK, en caso de no estarlo, se enviará un mensaje DHCPNACK.

Los clientes, normalmente intentan renovar su concesión:

- Cuando inician su interfaz, para asegurarse de poder usar la dirección Ip anteriormente otorgada, y si no pueden, solicitar una nueva.

- Antes de que finalice el periodo de concesión, para garantizar que la información de configuración quede actualizada.

- Manualmente desde el cliente en cualquier otro momento

El cliente puede elegir abandonar su alquiler de una dirección enviando el mensaje DHCPRELEASE al servidor.

Se pueden actualizar los parámetros de configuración cuando el cliente contacta con el servidor para cualquier acción de todas las anteriores.

## Mensajes DHCP

Tenemos principalmente 8 mensajes diferentes:

1. DHCPDISCOVER: mensaje que envía el cliente a la red para solicitar unirse a un servidor.

2. DHCPOFFER: mensaje que envía el servidor como respuesta al cliente que la ha solicitado.

3. DHCPREQUEST: mensaje que envía el cliente como respuesta a la oferta que le ha realizado un servidor.

4. DHCPACK: mensaje de confirmación y cierre desde el servidor hacia el cliente indicándole los parámetros definitivos.

5. DHCPNACK: mensaje que informa desde el servidor al cliente de que la Ip solicitada no es válida, o la configuración incluye algún error.

6. DHCPDECLINE: el cliente informa al servidor de que la dirección está en uso.

7. DHCPRELEASE: el cliente informa al servidor de que ha finalizado su uso con la Ip.

8. DHCPINFORM: el cliente consulta al servidor la configuración local.

## Configuración de varios servidores

Un servidor DHCP debe estar en la misma red física que los clientes. Si se dispone de varias redes para conectar, tenemos diferentes opciones:

1. Configurar un servidor DHCP en cada subred: esta opción supone un aumento del trabajo administrativo y del equipo necesario, ya que habrá que ubicar un servidor DHCP en cada subred individual.

2. Configurar un servidor DHCP desde una ubicación centralizada a varias subredes: en este caso se pueden contemplar varias opciones 
   
   1. Conectar el servidor directamente a las diferentes redes
   
   2. Que los enrutadores tengan capacidad de retransmitir mensajes DHCP entre redes
   
   3. Instalar un agente de retransmisión DHCP en algún equipo y configurarlo para escuchar los mensajes y redirigirlos a un servidor DHCP específico.

## Agente de retransmisión DHCP

Se trata de un equipo o enrutador configurado para escuchar difusiones DHCP procedentes de clientes, y a continuación, retransmitir dichos mensajes a los servidores.

Los tipos de agentes de retransmisión DHCP pueden ser:

- Integrados en routers

- Servidor que funcione como agente de retransmisión

Para que un servidor funcione como agente de retransmisión, el funcionamiento debe ser el siguiente.

1. El cliente difunde un mensaje DHCPDISCOVER

2. El agente de retransmisión de la subred del cliente reenvía el mensaje al servidor mediante unidifusión.

3. El servidor emplea unidifusión para enviar el mensaje de DHCPOFFER al agente.

4. El agente de retransmisión difunde el DHCPOFFER a la subred del cliente.

5. El cliente difunde el mensaje DHCPREQUEST

6. El agente de retransmisión recoge el mensaje y se lo envía al servidor mediante unidifusión.

7. El servidor genera el mensaje DHCPACK, que se le envía de nuevo al agente.

8. Finalmente el agente envía este último mensaje a la subred del cliente.

El servidor DHCP dará la configuración dará una configuración de red adecuada al segmento de red al que está conectado el equipo que efectúa la petición.

# Protocolo de failover en DHCP

Cuando 2 servidores DHCP se encuentran en la misma red, ambos mantienen una base de datos con sus concesiones y sus estados. Para evitar que la misma dirección Ip se asigne a varios equipos, la solución es que estos trabajen con distintos rangos de direcciones.

Si ambos servidores desean trabajar con el mismo rango de direcciones, es necesario que sus bases de datos estén sincronizadas. Para esto se utiliza el protocolo Failover entre 2 servidores DHCP.

Este protocolo también se puede utilizar para realizar un balanceo de carga, de manera que el trabajo se reparta entre los servidores primario y secundario.
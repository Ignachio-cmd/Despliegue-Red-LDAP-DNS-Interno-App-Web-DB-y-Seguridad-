# Despliegue-Red-LDAP-DNS-Interno-App-Web-DB-y-Seguridad-

- ¿Que vamos a hacer?:

Para la práctica deberán configurarse los siguientes elementos.

-Un servidor LDAP que servirá para la autentificación de clientes.

-Un cliente Linux que hará login con usuarios creados en LDAP

-Un servicio web (puede ser un servidor web o cualquier otro que necesite de una base de datos para funcionar).

-Un servidor de BBDD al que se conectará el servicio web anteriormente creado.

-Un PC-Router que habilitará el acceso a internet de todos los equipos.

-Servidor DNS interno que asigne nombres a todos los servicios

Restricciones:

-Todos los equipos tendrán una única interfaz de red de tipo interno.

-El PC-ROUTER tendrá una interfaz extra de tipo puente (o NAT) que hará de entrada/salida hacia internet (configuración de iptables).

-El servidor de BBDD únicamente aceptará peticiones del servicio web (configurar firewall para ello).

-El servidor DNS interno, sólo resolverá nombres dentro de la red interna, usará Bind9.

- ¿Que debemos tener al final?:

1.-Topología de la red (IP y nombre de cada equipo)

2.-Listado de ficheros de configuración modificados.

3.-Listado de órdenes empleadas en cada configuración.

4.-Entrevista: Aplicar cambios en la topología y crear usuarios en el LDAP.


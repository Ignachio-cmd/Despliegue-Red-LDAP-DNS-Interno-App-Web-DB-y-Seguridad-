# Despliegue-Red-LDAP-DNS-Interno-App-Web-DB-y-Seguridad-

1.-Topología de la red (IP y nombre de cada equipo):

![Topologia_de_la_red](imgs/pcrouter/TopologiaDeLaRed.png)

2.-¿Que he hecho en el pcrouter?:

- Configurar pcrouter:
  pcrouter va a tener dos redes, una interna y otra nat, esto nos va servir para que las demas maquinas tengas acceso a internet a traves de esta.
  
  <p><p>
- Configuramos su netplan:
  
  ![netplan](imgs/pcrouter/netplanpcrouter.png)
  Una vez terminado de preparar el fichero tendremos que usar el comando:
  
  ``sudo nano netplan apply``
  
  <p><p>
- Ahora vamos a configurar las iptables:
  Son puros comandos de consola
  
  ``sudo sysctl -w net.ipv4.ip_forward=1 ``
  ``sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE``
  ``sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT``
  ``sudo iptables -A FORWARD -I enp0s3 -O enp0s8 -M State -STATE RELATED,ESTABLISHED -j ACCEPT``
  
  Una vez puestos esos comandos vamos a explicarlos un poco:

* **net.ipv4.ip_forward=1** --> Nos permite que el ordenador actúe como un intermediario basicamente.
  
  ``sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE``
  
  ¿que hace esta linea?: Basicamente esta línea hace que todos los ordenadores de nuestra red local que quieran salir a internet se "disfracen" usando la dirección IP del pcrouter para salir por la conexión principal.
  
  <p>
* **-t nat** --> Indica a iptables que vamos a trabajar con la tabla ''nat''
* **-A POSTROUTING** : Añade (-A) una regla a la cadena "POSTROUTING". Esto significa que la regla se aplicará justo después de que el sistema haya decidido por dónde va a salir el paquete de datos.
* **-o enp0s3** : Esta regla solo se aplica al tráfico que sale (-o de output) por la interfaz de red enp0s3.
* **-j MASQUERADE** : Cuando los paquetes de la red local salen a Internet a través de enp0s3, este comando es el que cambia la dirección IP de origen de los paquetes por la dirección IP de enp0s3.
  
  ``sudo iptables -A FORWARD -i enp0se8 -o enp0se3 -j ACCEPT``
  
  ¿Que hace esta linea?: Esta linea basicamente es como abrir una puerta para que lo que llegue desde enp0s8 pueda pasar a la enp0s3.
  
  <p>
* **-A FORWARD**: Es para indicar que esta regla se aplica al tráfico que está siendo reenviado.
* **-j ACCEPT**: Indica que el tráfico que cumpla con estas condiciones debe ser permitido pasar.
  
  ``sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT``
  
  ¿Que hace esta linea?: Es la que nos permite que las respuestas que nos da internet puedan volver a nuestra red interna.
* **-m state**: Le dice a iptables que use el módulo de seguimiento de conexiones.
* **--state RELATED,ESTABLISHED**: Significa que solo se permitirá el tráfico que esté relacionado con una conexión que ya se inició desde tu red local.

Con eso ya tendriamos las iptables configuradas pero tenemos que instalar una cosita para que cuando reinicemos el pc las normas puestas no se borren y esto seria:

- ``sudo apt-get install iptables-persistent``
  
  Y para aplicarlo:

* ``+sudo netfilter-persistent save``

Si quisieramos reedirigir la informacion que nos llega desde un puerto a otro seria asi:

``sudo iptables -t nat -A OUTPUT -p tcp --dport 80 -j REDIRECT --to-port 8080``

* Ahora pasemos a instalar LDAP:
  ``sudo apt-get install slapd ldap-utils``
  ``sudo dpkg-reconfigure slapd``
  Tras estos dos comandos nos saltara un instalador para Ldap donde deberemos poner lo siguiente:
  * Nombre del dominio.
  * El nombre de la organizacion.
  * Elegimos una contraseña de admin.
  * Cuando nos diga que si queremos quitar la DB le decimos que no.
  * Y cuando nos diga de mover la antigua le diremos que si.

Y con eso ya tendriamos ldap en nuestra maquina server.

* Ahora pasemos a preparar el DNS:

1. Instalamos bind9:
   ``sudo apt-get install bind9``
2. Vamos a configurar el siguiente fichero donde vamos a decir donde se encuentran los ficheros de zona y zona inversa:
   ``sudo nano /etc/bind/named.conf.local``










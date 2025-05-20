# Despliegue-Red-LDAP-DNS-Interno-App-Web-DB-y-Seguridad-

Topología de la red (IP y nombre de cada equipo):

![Topologia_de_la_red](imgs/pcrouter/TopologiaDeLaRed.png)

# PCROUTER

### ¿Que he hecho en el pcrouter?:

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
  Pero este comando se borra al reiniciar la maquina, si quieres hacerlo permanente tendremos que ir a:
  
  ``sudo nano sysctl.conf``
  
  y descomentar la linea que es igual que el comando anteriormente dado.
  
  ![ipv4](imgs/pcrouter/ipv4.png)
  
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
  
  ``sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -m state --state RELATED,ESTABLISHED -j ACCEPT``
  
  ¿Que hace esta linea?: Es la que nos permite que las respuestas que nos da internet puedan volver a nuestra red interna.
* **-m state**: Le dice a iptables que use el módulo de seguimiento de conexiones.
* **--state RELATED,ESTABLISHED**: Significa que solo se permitirá el tráfico que esté relacionado con una conexión que ya se inició desde tu red local.

Con eso ya tendriamos las iptables configuradas pero tenemos que instalar una cosita para que cuando reinicemos el pc las normas puestas no se borren y esto seria:

- ``sudo apt-get install iptables-persistent``
  
  Y para aplicarlo:

* ``+sudo netfilter-persistent save``

Si quisieramos reedirigir la informacion que nos llega desde un puerto a otro seria asi:

``sudo iptables -t nat -A OUTPUT -p tcp --dport 80 -j REDIRECT --to-port 8080``

---

# Ahora pasemos a instalar LDAP:

``sudo apt-get install slapd ldap-utils``

``sudo dpkg-reconfigure slapd``

Tras estos dos comandos nos saltara un instalador para Ldap donde deberemos poner lo siguiente:

* Nombre del dominio.
* El nombre de la organizacion.
* Elegimos una contraseña de admin.
* Cuando nos diga que si queremos quitar la DB le decimos que no.
* Y cuando nos diga de mover la antigua le diremos que si.

Y con eso ya tendriamos ldap en nuestra maquina server.

### Vamos con el usuario:

El paso 1 y 2 se pueden omitir si ya los tienes creados.

1. Creamos una unidad organizativa:
   
   ``sudo nano ou_people.ldif``
   
   y lo añadimos:
   
   ``ldapadd -x -D cn=admin,dc=dominya,dc=com -W -f ou_people.ldif``
   
   ![oudeldap](imgs/pcrouter/ou.png)
   
   <p>
2. Creamos un grupo:
   
   ``sudo nano buengrupo.ldif``
   
   ![grupodellap](imgs/pcrouter/grupo.png)
   
   y lo añadimos:
   
   ``ldapadd -x -D cn=admin,dc=dominya,dc=com -W -f buengrupo.ldif``
   
   <p>
3. Creamos el usuario:
   
   ``miguelUser.ldif``
   
   ![usuarioldap](imgs/pcrouter/user.png)
   
   y lo añadimos:
   
   ``ldapadd -x -D "cn=admin,dc=dominya,dc=com" -W -f javiUser.ldif``

# Ahora pasemos a preparar el DNS:

1. Instalamos bind9:
   
   ``sudo apt-get install bind9``
2. Vamos a configurar el siguiente fichero donde vamos a decir donde se encuentran los ficheros de zona y zona inversa:
   
   ``sudo nano /etc/bind/named.conf.local``
   
   ![Rutas de los ficheros zonas](imgs/pcrouter/rutasFicherosZOna.png)
   
   <p>
3. Pasemos a configurar el fichero zona y zona inversa:
   
   * Primero copiamos el fichero de zona y de zona inversa de ejemplo que tenemos para poder tener una buena estructura:
     
     ``sudo cp /etc/bind/db.local /etc/bind/db.dominya.com``
     
     ``sudo cp /etc/bind/db.127 /etc/bind/db.192.168.22 ``
     
     <p>
   * Ahora editamos el fichero zona:
     
     ``sudo nano /etc/bind/db.dominya.com``
     
     ![ficheroZona](imgs/pcrouter/ficheroZona.png)
   * Ahora editamos el fichero zona inversa:
     
     ``sudo nano /etc/bind/db.192.168.22``
     
     ![ficheroZona](imgs/pcrouter/ficheroZonaInversa.png)
     
     Podemos comprobar si tenemos bien el fichero a traves de:
     
     ``sudo named-checkzone 22.168.192.in-addr.arpa /etc/bind/db.192.168.22``
     
     Una vez ya lo tengamos todo solo tendremos que reiniciar el servicio de bind9:
     
     ``sudo systemctl restart bind9``

# PCCLIENT:

### ¿Que he hecho en el pcclient?:

1. Configurar el netplan para darle una ip y poder tener salida internet.
   
   ![netplan](imgs/pcclient/netplanC.png)

👀️  ``sudo netplan apply`` no te olvides.

<p>
2. Instalamos ldap-client:

``sudo apt-get install libnss-ldap libpam-ldap ldap-utils nscd``

Y durante la instalacion:

* URI del servidor LDAP: ldap://192.168.22.1
* DN de búsqueda: dc=dominya,dc=com
* Versión LDAP: 3
* Hacer que la base de datos local sea escribible: No
* ¿Exigir inicio de sesión para realizar acciones administrativas?: No
* ¿Permitir que el administrador LDAP se comporte como administrador local?: Sí
* ¿Permitir acceso sin contraseña?: No
  <p><p>

3. Ahora tenemos que configurar el fichero ``/etc/nsswitch.conf``:
   ¿Para que nos sirve este archivo? pues es fundamental para decirle al sistema Linux cómo debe buscar y resolver información sobre diferentes tipos de datos
   
   ![nssitch](imgs/pcclient/nss.png)
   
   <p>
4. Ahora vamos a aplicar una pequeña opcion poniendo el siguiente comando:
   
   ``sudo pam-auth-update``
   
   Este comando nos dara distintas opciones pero a nosotros la que nos interesa es para que se cree automaticamnete el directorio del usuario ldap.
   
   ![auth](imgs/pcclient/auth.png)
   
   <p>
5. Por ultimo inicamos sesion:
   
   ``su - usuario``

# PCDB:

### ¿Que he hecho en pcdb?:

1. Configurar el netplan:
   
   ![NetPlanDB](imgs/pcdb/netplanDB.png)
   
   👀️  ``sudo netplan apply`` no te olvides.
   
   <p>
2. Instalar mysql:
   
   ``sudo apt install mysql-server -y``
   
   ``sudo mysql_secure_installation``
   
   El secure installation es para ejecutar un asistente interactivo para hacer que la instalación de MySQL sea más segura, configurando contraseñas, eliminando usuarios y bases de datos de prueba, y ajustando permisos por defecto.
   
   <p>
3. Ahora es el turno de bloquear a pcclient para que no pueda acceder a la base de datos y que solo pcweb pueda acceder a ella. Para ello escribiremos los siguientes comandos:
   
   * ``sudo ufw allow from 192.168.22.3 to any port 3306``
   * ``sudo ufw deny from 192.168.22.2 to any port 3306``
   * ``sudo ufw enable``






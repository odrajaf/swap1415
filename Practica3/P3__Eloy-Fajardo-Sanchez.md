
#Practica 3

[defaultNginx]:./defaultNginx.png
[curlRespuesta]:./curlRespuesta.png
[configHaproxy]:./configHaproxy.png
[curlHAproxy]:./curlHAproxy.png


##Nginx

Como en la practica anterior seguimos los pasos para tener una máquina virtural operativa.

**1º Instalamos Ubuntu server** 12.04.5 marcando solo la opción de OPENSSH, ***NO INSTALAMOS LAMP***

**2º Configuramos la red interna para virtual box** mediante un editor de textos en modo superusuario 
`sudo vim /etc/network/interfaces` añadimos las siguientes lineas

    auto eth1
    iface eth1 inet static
    address 192.168.1.103
    gateway 192.168.1.1
    netmask 255.255.255.0
    network 192.168.1.0
    broadcast 192.168.1.255

Por tanto nuestra dirección será **192.168.1.103**

**3º Pasamos a instalar nginx**, podemos hacerlo instalandolo de repositorios ejecutando

    sudo apt-get install nginx

Se instalarán las dependecias necesarias y nginx se instalará en el puerto 80 de la máquina.
    
Una vez hechos estos tres podemos empezar a configurar nuestro balanceador.

4**º Configuración del fichero `/etc/nginx/conf.d/default.conf`**

Podemos modificarlo con cualquier editor nosotros usaremos vim

    sudo vim /etc/nginx/conf.d/default.conf
<br>
Una vez editando el fichero, al principio pondremos las dos máquinas servidoras web de la práctica anterior.

    upstream apaches {
        server 192.168.1.100;
        server 192.168.1.101;
    } 
En server pondremos la configuración del balanceador que será la siguente:

    server{
        listen 80;
        server_name localhost;
        
        access_log /var/log/nginx/localhost.access.log;
        error_log /var/log/nginx/localhost.error.log;
        root /var/www/;
        location / {
            proxy_pass http://apaches;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
        }
    }
<br>
En la  siguiente imagen podemos apreciar como quedaría finalmente la configuración

![alt text][defaultNginx]

Por último reiniciamos el servidor nginx para que tenga en cuenta los cambios ejecutando

    sudo service nginx restart
    

**5º Comprobaciones**

Si ahora desde cualquiera de las máquinas dentro de la red interna de las maquinas virtuales realizamos un curl al balanceador  (192.168.1.103) nos debería mostrar respuesta ya que realiza la petición al los servidores web y la devuelve.

![alt text][curlRespuesta]
	
<br>
##HAproxy
Se podría instalar ambos valanceadores en el misma máquina pero para evitar tener problemas con el puerto 80 hemos preferido instalar HAproxy en una máquina aparte.

En caso de querer instalar HAproxy en la misma máquina nos saltariamos los dos primeros pasos y deberíamos detener nginx primero ejecutando
    
    sudo service nginx stop
    
Para cerciorarnos que el servicio se ha parado podemos ejecutar el siguien te comando buscando en las tareas ejecutandose

    ps -e | grep nginx
Si el comando no muestra nada es que el servicio  de nginx está parado y podemos comenzar con HAproxy sin problemas.
    

**1º Instalamos Ubuntu server** 12.04.5, al igual que antes marcamos para instalar OPENSSH, ***NO INSTALAMOS LAMP***

**2º Configuramos la red interna para virtual box** En este caso le pondremos una dirección direfente **192.168.1.102**
`sudo vim /etc/network/interfaces` añadimos las siguientes lineas

    auto eth1
    iface eth1 inet static
    address 192.168.1.102
    gateway 192.168.1.1
    netmask 255.255.255.0
    network 192.168.1.0
    broadcast 192.168.1.255
    
**3º Instalamos HAproxy**, podemos hacerlo instalandolo de repositorios ejecutando

    sudo apt-get install haproxy

Al igual que nginx se instalará en el puerto 80 por lo que solo podremos usar uno a la vez en ese puerto.
    
Una vez hechos estos tres podemos empezar a configurar nuestro balanceador.
    
**4º Editamos el archivo** `/etc/haproxy/haproxy.cfg`
 Le asignaremos la siguiente configuración
 
    global
        daemon
        maxconn 1000
    
    defaults
        mode http
        contimeout 4000
        clitimeout 42000
        srvtimeout 43000
  
    frontend http-in
        bind *:80
        default_backend servers
        
    backend servers
        server    m1 192.168.1.100:80 maxconn 64
        server    m2 192.168.1.101:80 maxconn 64
   

En backend servers ponemos las IPs de las máquinas servidoras de páginas web y el maxconn global a 1000 ya que para probar benchmark como ab o siege ya que normalmente el numero de peticiones suele ser mil.

Si ejecutamos el comando 
    
    sudo /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg
   
Podemos ver como queda el resultado y como ejecutamos el comando para iniciar el servicio de HAproxy.

![alt text][configHaproxy]

<br><br>


**5º Comprobación**

Al igual que antes comprobaremos que el balanceador está funcionando correctamente con el comando curl, deberemos tener al menos ejecutandose las dos máquinas de servicio web y el balanceador

![alt text][curlHAproxy]

Ejecutamos varias veces el comando curl para ver que balancea correctamente a ambos servidores web.

#Practica 4

[caso1]:./Caso1.png
[caso2]:./Caso2.png
[caso3]:./Caso3.png
[curl1]:./curlservidor1.png
[curl2]:./curl2.png
[curl3]:./curl3.png
[abUna]:./abPrueba.png
[salidaAb]:./salidaAb.png
[abHaproxy]:./abHaproxy.png
[abNginx]:./abNginx.png
[graficaAb]:./graficaAb.png
[timeTakenAb]:./timeTakenAb.png
[failRequestAb]:./failRequestAb.png
[requestPerSAb]:./requestPerSAb.png
[siegecaso1]:./siegecaso1.png
[siegecaso1b]:./siegecaso1b.png
[siegecaso2]:./siegecaso2.png
[siegecaso3]:./siegecaso3.png
[graficaSiege]:./graficaSiege.png
[disponibilidadSiege]:./disponibilidadSiege.png
[TemTransSiege]:./TemTransSiege.png
[TiemRespuesta]:./TiemRespuesta.png
[tasaTransmision]:./tasaTransmision.png


Para esta práctica usaremos la herramienta **apache benchmark** (ab)
y **siege** en tres casos diferentes de servir peticiones web.

En el **primer caso** solo se harán peticiones a un servidor web, teniendo que suplir respuesta *una sola máquina*, como se muestra en la imagen.
![alt text][caso1]


En el **segundo caso** la maquina testeadora que hará uso de las herramientas mencionadas hará peticiones a un balanceaador *HAproxy* y balanceará las peticiones a los dos servidores web, en la siguente imagen vemos claramente a que nos referimos.
![alt text][caso2]

<br>
El **tercer caso** es similar al anterior ya que sigue la misma filosofía pero el balanceador será *Nginx*
![alt text][caso3]

###Primer Caso Apache Benchmark
Una vez explicados en que casos haremos las pruebas pasaremos a realizar las pruebas con apache benchmark. Para ello encenderemos el servidor web 1 y la maquina testeadora.

Primero comprobaremos con el comanado curl que el servidor web está activo, teniendo en cuenta que el script que hay en el servidor es el siguente.

    <?php

    $tiempo_inicio = microtime(true);
    
    for ($i=0; $i<1200000; $i++){
     $a = $i * $i;
     $b = $a - $i;
     $c = $a / $b;
     $d = 1 / $c;
    }
    
    $tiempo_fin = microtime(true);
    
    echo "Tiempo empleado: " . round($tiempo_fin - $tiempo_inicio, 4) ; 
    
    ?>
<br><br>
Como podemos apreciar el servidor responde correctamente
![alt text][curl1]

<br><br>
Para realizar las pruebas con apache benchmark usaremos el siguente comando donde -n será el numero de peticiones totales y -c el numero de peticiones concurrentes enviadas a la vez. Redirigimos la salida para posteriormente comprobar los datos.

    ab -n 1000 -c 20 http://dirección/index.php >archivoSalida.txt
<br>
Primero mediante prueba y error determinamos cual era el rango de concurrencia para apreciar si el servidor puede caer, nosotros determinamos que un buen rango de concurrencia para las pruebas con ab es de 20 a 225.

Para el primer caso el comando para ejecutar el test sería 

    ab -n 1000 -c 20 http://192.168.1.100/index.php >abuna1000c20.txt
<br>
![alt text][abUna]

<br>
Si hacemos un cat al archivo de salida generado veremos los resultados obtenidos, *marcados en rojo los que usaremos*.
![alt text][salidaAb]
<br><br>

###Segundo Caso Apache Benchmark (HAproxy)
Primero levantaremos todas las máquinas y comprobaremos con un curl que tanto los dos servidores como el balanceador con HAproxy dan respuesta correctamente.

![alt text][curl2]

Ejecutaremos ab teniendo en cuenta que debemos hacerle las peticiones al balanceador con dirección *192.168.1.102* que a su ver la repartirá en los servidores web.

    ab -n 1000 -c 20 http://192.168.1.102/index.php > abHaproxy1000c20.txt
<br>
![alt text][abHaproxy]
<br><br>

###Tercer Caso Apache Benchmark (Nginx)
Ahora como balanceador utilizaremos nginx, al igual que antes comprobaremos que todas las máquinas están reciviendo respuesta correctamente mediante curl.

![alt text][curl3]

Para Nginx deberemos ejecutar los test de ab poniendo la dirección del balanceador Nginx *192.168.1.103*

    ab -n 1000 -c 20 http://192.168.1.103/index.php > abNginx1000c20.txt
<br>
![alt text][abNginx]
<br><br>

Una vez que hemos realizado todos los test para cada caso con el rango de concurrencia de 20 a 225, podemos crear una tabla con los datos y ver como reacciona nuestra granja web con las diferentes peticiones.

![alt text][graficaAb]
<br><br>
Con cada columna hemos realizado una gráfica para ver de forma más visual los datos, primero analizaremos el tiempo tomado por test en cada caso.

![alt text][timeTakenAb]

En la gráfica podemos apreciar que cuando se usan balanceadores (las lineas verdes) el tiempo que se tarda enviar las 1000 peticiones se reduce dando una respuesta más rapida en general.

En la siguente gráfica apreciaremos las respuestas no respondidas por el servidor en alguno de los tres casos, esto siempre nos interesa que sea lo más bajo posible mejorando las disponibilidad del sitio web.

![alt text][failRequestAb]

Como podemos ver por debajo de las 100 peticiones concurrentes en todos los casos el sistema responde bastante bien, aquí como curiosidad HAproxy consigue perores resultados que una sola máquina, esto se debe a que se han realizado las pruebas en una máquina virtual y en este ambito no se pueden realizar con precisión los Benchmark. 

Aun en un entorno virtualizado sin retardos, nginx consigue responder a más número de peticiones concurrentes ya que utiliza un balanceador con dos máquinas servidoras web.

<br>
En la siguente gráfica veremos las respuestas por segundo de cada caso.

![alt text][requestPerSAb]

Como podemos ver Nginx tiene casi el doble de respuestar por segundo que una sola máquina, hecho que se refleja en la 1º gráfica, en cambio HAproxy aumenta el numero de respuestas en función de las peticiones concurrentes llegadas. Otra vez podemos asegurar que usar un balanceador mejora las prestaciones de nuestra granja Web.

###Primer Caso con Siege
Al igual que antes realizaremos primero una comprobación a ver si ambas máquinas están activas y dando respuesta mediante curl

    curl 192.168.1.100
<br>
Ahora describiremos las opciones que usaremos con la herramienta siege

    siege -b -t60S -c 70 -v http://192.168.1.100
Con la opción -b ejecutará sin pausa las peticiones pudiendo así obtener buenos datos, la obción -t indica el tiempo del test, -c el numero de peticiones concurrencias, -v modo verbose y finalmente la dirección a la que queremos testear.

El rango de concurrencia que usaremos con siege es de **70** a **115** **peticiones**.

En la siguiente captura mostramos como sería una ejecución del test para el primer caso con *una sola máquina servidora*, como inconveniente no podremos redirigir la salida como con ab y deberemos de ver los datos al final de la ejecución del test.

![alt text][siegecaso1]

El final de la ejecución sería, *marcando en rojo los datos que luego analizaremos*

![alt text][siegecaso1b]
<br><br>

###Segundo Caso con Siege (HAproxy)
Volveremos a cerciorarnos que tanto el balanceador HAproxy como el servidor web 1 y el servidor web dos están activos.

       curl 192.168.1.100
       curl 192.168.1.101
       curl 192.168.1.102
Con siege ahora la dirección a testear es la dirección del balanceador (192.168.1.102) siendo el comando a usar(para cada concurrencia)
    
    siege -b -t60S -c 70 -v http://192.168.1.102
    
![alt text][siegecaso2]
<br><br>


###Tercer Caso con Siege (Nginx)
Como anteriormente hacemos el curl correspondiente al balanceador, en este caso Nginx y a los dos servidores web

       curl 192.168.1.100
       curl 192.168.1.101
       curl 192.168.1.103
       
Ahora ejecutaremos siege para cada una de las concurrencias a testear a la dirección del balanceador nginx
    
    siege -b -t60S -c 70 -v http://192.168.1.103
    
![alt text][siegecaso3]
<br><br>

Por ultimo analizaremos los datos reunidos de todas las ejecucines de siege para los diferentes casos y concurrencias, en la siguiente tabla se muestran todos los datos obtenidos en las ejeciones de los test.

![alt text][graficaSiege]

Comenzaremos analizando la **disponibilidad**, en este caso hemos obtenido resultados dispares.

![alt text][disponibilidadSiege]

Curiosamente una sola máquina no puede cubrir más de las 70 peticiones concurrentes mientras que con Nginx se cubrian incluso con niveles de concurrencia mucho más altos. 

HAproxy muestra unos datos muy por debajo que una sola máquina lo que nos hizo comprobar que balanceaba correctamente, como resultado vimos que se debía a la limitación de conexiones del archivo de configuración `/etc/haproxy/haproxy.cfg` concretamente el parametro **maxconn**. Con maxconn 1000 resolvía peticiones correctamente ya que es el limite de ab y aproximadamente las peticiones totales que usamos con siege.
<br><br>

El **tiempo transcurrido** es más bien una referencia para los demás parametros por lo que su gráfica no nos da demasiados datos, además de que todos tiempos de referencia se hicienron en 60s

![alt text][TemTransSiege]
<br><br>

En el **tiempo de respuesta** se puede apreciar cuando servidor está sirviendo bien o no una página.

![alt text][TiemRespuesta]

En esta grafica apreciamos que cuando una sola máquina sirve tiene un tiempo de respuesta 22~ (línea amarilla) y aumenta un poco mientras el servidor responde correctamente, cuando la linea amarilla cae, el servidor apenas sirve correctamente las peticiones aunque estas se tarden menos en servirse en forma de error.

Nginx (líena Roja) muestra como conforme aumenta la carga aunque las sirva todas las peticiones, tarda más en servirlas a más carga.
<br><br>

Con el siguiente parametro **Tasa de Transmisión** veremos claramente la ventaja entre una sola máquina servidora y una granja web.

![alt text][tasaTransmision]

En este caso los balanceadores tienen practicamente el doble de tasa de transmisión que una sola máquina, además de mantenerse cuando la carga de peticiones es mayor.

###Conclusión
Después de todas las pruebas con ab como con siege comprobamos que tener una granja web incluso en un entorno sintetico en el que hemos realizado las pruebas usar una granja web en lugar de un solo servidor nos supone una mejora de prestaciones y una posible escalabilidad más sencilla de cara al futuro.


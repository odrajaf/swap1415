[capturaNmap]:./imagenesActividades/capturaNmap.png
#Ejercicio T6.2:
##Comprobar qué puertos tienen abiertos nuestras máquinas, su estado, y qué programa o demonio lo ocupa.

Esta tarea se puede realizar facilmente mediante nmap, para instalarlo en ubuntu podríamos el comando:

    sudo apt-get install nmap
    
Para ver los puertos de nuestra máquina, si están abiertos y que demonio los ocupa ejecutamos

    nmap localhost
<br>
![alt text][capturaNmap]

Como vemos nos muestra los puertos abiertos y el servicio funcionando en ellos.
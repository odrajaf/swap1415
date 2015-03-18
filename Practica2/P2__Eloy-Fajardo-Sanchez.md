
#Practica 2

[cpClavePublica]:https://github.com/odrajaf/swap1415/blob/master/Practica2/copiando%20clave%20publica.png
[sshClavePub]:https://github.com/odrajaf/swap1415/blob/master/Practica2/ssh%20mediante%20clave%20publica.png
[crontab]:https://github.com/odrajaf/swap1415/blob/master/Practica2/crontab.png
[resultado]:https://github.com/odrajaf/swap1415/blob/master/Practica2/contrab%20con%20rsync.png

1º Instalamos Ubuntu server 12.04.5 marcando las opciones de LAMP y OPENSSH, a esta instalación la llamaremos ubuntu01

2º En ubuntu01 mediante un editor de textos en modo superusuario 
`sudo vim /etc/network/interfaces` añadimos las siguientes lineas

    auto eth1
    iface eth1 inet static
    address 192.168.1.100
    gateway 192.168.1.1
    netmask 255.255.255.0
    network 192.168.1.0
    broadcast 192.168.1.255


3º Creamos otra máquina virtual e instalamos Ubuntu server 12.04.5 con exactamente las mismas opciones de antes, a la que llamaremos ubuntu02

4º Modificamos también el archivo /etc/network/interfaces pero cambiando la dirección ip de ubuntu02

    auto eth1
    iface eth1 inet static
    address 192.168.1.101
    gateway 192.168.1.1
    netmask 255.255.255.0
    network 192.168.1.0
    broadcast 192.168.1.255


5º Antes de usar rsync es conveniente para evitar problemas de permisos hacer en ambas maquinas<br/>`sudo chown fajardo:fajardo -R /var/www`

para instalar rsync usamos `sudo apt-get install rsync`
		

6º Preferimos por usar un usuario normal en lugar de root para el ssh, 
   generamos la clave publica en ubuntu02<br/>`ssh-keygen -t dsa`

copiamos la clave publica de ubuntu02 en ubuntu01<br/>
`ssh-copy-id -i ~/.ssh/id_dsa.pub fajardo@192.168.1.100`
	
![alt text][cpClavePublica]

probamos a conectarnos por ssh sin que nos pida clave<br/>
	`ssh 192.168.1.100 -l fajardo`
	
![alt text][sshClavePub]

7º Modificamos el archivo crontab de ubuntu02 con `sudo vim /etc/crontab`
añadimos la línea<br/>`* * * * *	fajardo	rsync -avz -e ssh fajardo@192.168.1.100:/var/www/* /var/www`

esto actualizará el contenido de las carpetas cada minuto.

![alt text][crontab]

En caso de modificar algún archivo de /var/www/ en ubuntu01 se modificará en ubuntu02 cada minuto

![alt text][resultado]
	



# Raspberry SSH Setup

1. ####  Descargar el raspberry imager de [aquí](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#2-prepare-the-sd-card).
1. #### Seleccionar el OS, en este caso Ubuntu 22.04.1, y la microSD donde se va a instalar.
1. #### Abrir las opciones avanzadas y configurar el ssh.
1. #### Meter la microSD en la raspberry, iniciar y hacer las configuración inicial.
1. #### Iniciar sesión, abrir un terminal y escribir `sudo apt update`, `sudo apt upgrade` y `sudo reboot` para buscar e instalar las últimas actualizaciones.
1. #### `sudo systemctl set-default multi-user.target` para quitar la interfaz gráfica por defecto y `sudo systemctl isolate multi-user.target`.
1. #### ‘ssh-keygen’ en el servidor y cliente para generar las claves.
1. #### Abrir el puerto 22 y darle una dns dinámica para poder conectarse fuera de red local.

#
#
#

# Raspberry OpenVPN
##### Pasos sacados de [aquí](https://ubuntu.com/server/docs/service-openvpn)

1. #### Instala openvpn con `sudo apt install openvpn easy-rsa`.

1. #### `sudo make-cadir /etc/openvpn/easy-rsa` para crear el directorio.

1. #### Usa `sudo su` y `cd  /etc/openvpn/easy-rsa` para ir a la carpeta creada como root y usa `./easyrsa init-pki` y `./easyrsa build-ca`.

1. #### Usa `cp /pki/ca.crt /home/user/`.

1. #### `./easyrsa gen-req myservername nopass` Para generar la key del server y `./easyrsa gen-dh` para generar los parámetros.

1. #### Para crear los certificados usa `./easyrsa gen-req myservername nopass` y `./easyrsa sign-req server myservername`.

1. #### Ahora se copian todas las claves y certificados a /etc/openvpn/ con `cp pki/dh.pem pki/ca.crt pki/issued/myservername.crt pki/private/myservername.key /etc/openvpn/`.

1. #### Usa `cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/myserver.conf` para tener el template de configuración del server y asegurate que los certificados y keys apuntan a los archivos que corresponden:
    * ca ca.crt
    * cert myservername.crt
    * key myservername.key
    * dh dh2048.pem

       
1. #### Edita `/etc/sysctl.conf`  y activa la linea `net.ipv4.ip_forward=1` y usa `sysctl -p /etc/sysctl.conf` para aplicar los cambios.

1. #### Para crear un cliente se usa `./easyrsa gen-req myclient1 nopass` y `./easyrsa sign-req client myclient1`.

1. #### Una vez creado el cliente se usa `cp /pki/issued/myclient1.crt /home/user/` y `cp pki/private/myclient1.key /home/user/`.

1. #### Usa `cd etc/openvpn`, `sudo openvpn --genkey secret ta.key` y `cp ta.key /home/user/`.

1. #### Para tener el archivo de configuración del cliente usa `sudo cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /home/user/`.

1. #### Usa `chown user:user  /home/user/myclient1.crt`, `chown user:user /home/user/myclient1.key`, `chown user:user /home/user/ca.crt`, `chown user:user  /home/user/ta.key` y `chown user:user  /home/user/client.conf`.

1. #### Ahora desde otro ordenador usa 
    * `scp user@ip:/home/user/client.conf (path al fichero donde vas a guardar los archivos para los clientes sin paréntesis)`
    * `scp user@ip:/home/user/ca.crt (path al fichero donde vas a guardar los archivos para los clientes sin paréntesis)`
    * `scp user@ip:/home/user/ta.key (path al fichero donde vas a guardar los archivos para los clientes sin paréntesis)`
    * `scp user@ip:/home/user/myclient1.key (path al fichero donde vas a guardar los archivos para los clientes sin paréntesis)`
    * `scp user@ip:/home/user/myclient1.crt (path al fichero donde vas a guardar los archivos para los clientes sin paréntesis)`

1. #### Copia y edita el archivo client.conf, asegurate que los certificados y keys apuntan a los archivos que corresponden:
    * ca ca.crt
    * cert myclient1.crt
    * key myclient1.key
    * tls-auth ta.key 1

1. #### Asegurate de que el keyword client está puesto y cambia la ip por la de la vpn:
    * client
    * remote ip 1194

1. #### Guarda el archivo con el nombre del cliente y cambia la extensión a .ovpn

# Configuración del server de My Kasa

| Componente| Configuración |
|----------:|---------------|
|Equipo|**[HP EliteDesk 800 G2 Desktop Mini](https://www.amazon.es/gp/product/B081TB2SZ1)**|
|CPU|**Intel CORE I5 6500**|
|Velocidad|**3,6 GHz**|
|RAM|**8 Gb DDR4 SDRAM**|
|HD|**256 Gb SSD**|
|Sistema operativo|**Debian 11**|
||![My Kasa](https://m.media-amazon.com/images/I/41Ndmxy0BpL._AC_SL1002_.jpg)|

# Procedimiento de instalación

<details>
<summary>CONSIDERACIONES PREVIAS A LA INSTALACIÓN</summary>

He decidido seguir las siguientes premisas en la instalación para mantenerla normalizada y ordenadita:

- La IP del server es la 192.168.1.254, y el rango de IP's disponibles para DHCP  192.168.1.11/253
- El dominio local es *.home*
- El dominio público es *.mykasa.mooo.com* (facilitado por [FreeDNS](http://freedns.afraid.org/subdomain/))
- Para cada servicio instalado, se crea una subcarpeta en la capeta */docker* (en la raiz del server)
- Para cada servicio instalado, se crea una regla de backup en **Duplicati**, que creará copias de seguridad en periódicas según cada caso
> [!NOTE]
> Ya iré poniendo cosillas para completar la guia...
    
</details>

<details>
<summary>CONFIGURACION DEL SERVER</summary>

### INSTALAR NET-TOOLS
   (Necesario para disponer p.e. de *ifconfig*)
1. Instalar el paquete:
```plain
sudo apt-get install net-tools
```
2. Crear enlace simbólico para que esté siempre disponible:
```plain
sudo ln -s /sbin/ifconfig /usr/bin/ifconfig
```
### CONEXIÓN SSH CERTIFICADA

1. Instalar SSH en servidor:
```plain 
sudo apt install ssh
```
2. Generar clave SSH con PuTTYgen
3. Guardar claves pública y privada en el cliente en la ruta *%USERPROFILE%\.ssh*
4. Copiar key en el servidor en el archivo *$HOME/.ssh/authorized_keys*
5. Activar directivas SSH en el servidor (editar el archivo */etc/ssh/sshd_config*):

*RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile                    %h/.ssh/authorized_keys
PassworAuthentication no (esto es para deshabilitar el acceso al servidor con contraseña)*

6. Revisar permisos SSH:
```plain
https://eltallerdelbit.com/server-refused-our-key/#solucion_permisos
```
8. Reiniciar servicio SSH:
```plain 
sudo systemctl restart ssh
```

### ACTIVAR/DESACTIVAR ENTORNO GRÁFICO

1. Averiguar qué entorno gráfico tenemos:
```plain 
ls /usr/bin/*session
```
2. Deshabilitarlo (p.e. Gnome)
```plain
sudo systemctl disable gdm3
```
3. Arrancarlo (p.e. Gnome)
```plain 
sudo systemctl start gdm3
```

### INSTALAR SAMBA

1. Actualizar Linux
```plain sudo apt update
sudo apt upgrade
```
2. Instalar Samba Server
```plain
sudo apt install samba -y
```
3. Habilitar Samba en inicio del sistema
```plain
sudo systemctl enable smbd
```
4. Reiniciar Samba y comprobar status
```plain
sudo systemctl restart smbd
```
```plain
sudo systemctl status smbd
```
5. Añadir al archivo */etc/samba/smb.conf* los volúmenes a compartir (pueden ser varios), por ejemplo:

*[COMIC]*

*comment = Comic*

*path = /COMIC*

*writeable = yes*

*browsable = yes*

*public = no*

*force create mode = 774      # permisos por defecto cuando se crean ficheros*

*force directory mode = 774   # permisos por defecto cuando se crean carpetas*

*valid users = kike, root*

También habrá que modificar la sección *[homes]*, si es que queremos cambiar los permisos de la carpeta de usuario (accesible dependiendo del usuario que se conecte)

6. Declarar usuario/os en Samba
```plain
sudo smbpasswd -a kike
```
```plain
sudo smbpasswd -a root
```
7. Reiniciar Samba
```plain
sudo systemctl restart smbd
```
8. Validar smb.conf
```plain
testparm
```

### DESHABILITAR MODO HIBERNACIÓN
(Necesario para que el server esté funcionando 24/7)

1. Deshabilitar
```plain
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```
2. Para ver los efectos:
```plain
sudo systemctl status [targets]
```
3. Por si en algún momento queremos volver a habilitarlo:
```plain
sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

### ESTABLECER IP ESTÁTICA

1. Editar archivo de interfaces:
```plain 
sudo nano /etc/network/interfaces
``` 
2. Actualizarlo según el siguiente contenido:
   
*# The primary network interface*

*allow-hotplug eno1*

*iface eno1 inet static*

*address 192.168.1.254*

*netmask 255.255.255.0*

*network 192.168.1.0*

*gateway 192.168.1.1*

3. Aplicar cambios:
```plain
sudo ifdown eno1 (instalar ifupdown2)
```
```plain
sudo ifup eno1
```
### CONFIGURAR SERVIDOR DNS

1. Editar el archivo resolv.conf:
```plain
sudo nano /etc/resolv.conf
```
```
nameserver: 127.0.0.1
nameserver 192.168.1.254
nameserver 8.8.8.8
nameserver 8.8.4.4
```
2. Aplicar cambios:
```plain
sudo ifdown eno1 (instalar ifupdown2)
```
```plain 
sudo ifup eno1
```

### MONTAR UNIDAD NTFS

1. Instalar ntfs-3g (Necesario para montar unidades Windows ntfs)
```plain
sudo apt-get install ntfs-3g
```
2. Ver nombre dispositivos
```plain 
sudo fdisk -l
```
3. Montar unidad en carpeta
```plain 
sudo mount /dev/nombredispositivo carpetaenlaquemontarlo
```

### MONTAR UNIDAD EXFAT

1. Instalar ntfs-3g (Necesario para montar unidades Windows ntfs)
```plain 
sudo apt-get install fuse exfat-fuse exfat-utils
```
2. Ver nombre dispositivos
```plain 
sudo fdisk -l
```
3. Montar unidad en carpeta
```plain 
sudo mount -t exfat /dev/nombredispositivo carpetaenlaquemontarlo
```

### MONTAR UNIDAD NTFS AUTOMATICAMENTE (Y QUE ESTÉ DISPONBLE AL REINICIAR)

(ver [Montar particiones al iniciar Linux](https://vivaelsoftwarelibre.com/montar-particiones-al-iniciar-linux-automaticamente/)

1. Ver nombre dispositivos
```plain 
sudo fdisk -l
```
```plain 
dh -h
```
2. Ver UUID del dispositivos
```plain 
sudo blkid
```
3. Editar fstab
```plain 
sudo nano /etc/fstab
```
```plain 
UUID=iddeldispositivo carpetaenlaquemontarlo ntfs-3g defaults 0 0
```
</details>

<details>
<summary>INSTALAR DOCKER SWARM</summary>

Instalo con el método del script, que es el más sencillo.

Para más información, ver [Install Docker Engine on Debian](https://docs.docker.com/engine/install/debian/).

1. Actualizar Linux:
```plain 
sudo apt update
```
```plain 
sudo apt upgrade
```
5. Instalar paquetes para permitir que apt use un repositorio sobre HTTPS:
```plain 
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
```
6. Descargar el script de instalación proporcionado por Docker y ejecutarlo:
```plain 
curl -fsSL https://get.docker.com -o get-docker.sh
```
```plain 
sh get-docker.sh
```
7. Crear el grupo docker, agregar nuestro usuario y activar los cambios en el grupo:
```plain 
sudo groupadd docker
```
```plain
sudo usermod -aG docker ${USER}
```
```plain 
newgrp docker
```
8. Crear el directorio docker y darle permisos (es posible que se haya creado la carpeta automáticamente, pero que los permisos no sean los correctos...):
```plain 
sudo mkdir /docker
```
```plain
sudo chown root:docker /docker
```
```plain
sudo chmod 774 /docker
```
9. Configurar para que el servicio Docker se active automáticamente al reiniciar el equipo:
```plain 
sudo systemctl enable docker.service
```
```plain
sudo systemctl enable containerd.service
```
10. REINICIAR el equipo
    
11. Activar Docker swarm:
```plain
docker swarm init
```
</details>

<details>
<summary>INSTALAR DOCKER COMPOSE</summary>

1. Instalar el paquete:
```plain 
sudo apt-get install docker-compose
```
</details>

<details>
<summary>INSTALAR SERVICIOS DE GESTIÓN DE RED</summary>

Los servicios que queremos instalar son los siguientes:

  - **[Traefik](https://doc.traefik.io/traefik/)**: Proxy inverso y balanceador de carga
  - **[Portainer](https://www.portainer.io)**: Interfaz Web para manejo de Docker
  - **Whoami**: De momento no sé si me es util o no...

1. Crear las ruta de las configuraciones Docker en el server:
```plain 
mkdir -p /docker/{portainer/{config},traefik/{config,letsencrypt}}
```
2.  Descargar el archivo de configuración de Traefik [red/traefik.yaml](red/traefik.yaml) en */docker/traefik/config*
```plain 
wget https://raw.githubusercontent.com/EnriqueBesuman/Docker-Swarm-Config/master/gestion-red/traefik.yaml
```
3. Crear el archivo *acme.json* para almacenar certificados y *traefik.log* para los log y darles los permisos adecuados (imprescindible permiso 600 a acme.json para evitar error en el despliegue):
```plain
touch /docker/traefik/{traefik.log,letsencrypt/acme.json}
```
```plain
chmod 600 /docker/traefik/letsencrypt/acme.json
```
4. Crear una red para que todos los servicios Docker que necesiten de Traefik, se conecten a ella.
> [!NOTE]
> Tenemos pendiente hacer pruebas con la red privada...
```plain 
docker network create --driver=overlay --attachable traefik-network
```
5. Descargar y ejecutar el stack (al que llamaremos *gestion-red*) de instalación [red/docker-compose.yaml](red/docker-compose-gestion.yaml). A partir de aquí, usaremos Portainer para desplegar:

```plain
docker stack deploy -c docker-compose.yaml gestion-red
```
6. Incluir DNS de nuestros servicios en el fichero host del server (El fichero host en Linux está en */etc/hosts/*)
```plain
> [!NOTE]
> ¿Ponemos la DNS pública para servicios abiertos, o mantenemos DNS independientes para la web y para la intranet?
192.168.1.254	portainer.home
192.168.1.254	traefik.home
192.168.1.254	whoami.home
192.168.1.254	pihole.home
```
7. Si todo ha ido bien, los servicios estarán disponibles en las siguientes URL:
- Traefik
```plain
https://traefik.home/local/dashboard/
```
- Portainer
```plain
https://portainer.home/
```
- Pihole
```plain
https://pihole.home/admin
```
- Whoami
```plain
https://whoami.home
```
</details>

<details>
<summary>INSTALAR SERVICIOS DE GESTIÓN DE DNS</summary>

> [!IMPORTANT]
> Mi gestor de DCHP sigue siendo el router, no Pi-Hole... :poop: 

Los servicios que queremos instalar son los siguientes:

  - **[Pi-Hole](https://pi-hole.net/)**: Servidor de DNS y DHCP, y bloqueador de anuncios

1. Crear en Portainer un secret pihole_pwd para la contraseña de Pihole
   
3. Crear en Portainer un nuevo stack con el yaml [dns/docker-compose-pihole.yaml](dns/docker-compose-pihole.yaml) y desplegarlo
</details>

<details>
<summary>INSTALAR SERVICIOS DE DOMÓTICA</summary>
    
> [!CAUTION]
> En construcción...:-)
    
Los servicios que queremos instalar son los siguientes:

  - **[Home Assistant](https://doc.traefik.io/traefik/)**: Plataforma de software libre de gestión de domótica por antonomasia
  - **[mqtt]()**: I
  - **[Zigbee2mqtt]()**:
  - **[Frigate]()**:
  - **[nut-upsd]()**:
  - **[Visual Code]()**: 

1. Crear una red overlay llamada traefix. Todos los servicios Docker que necesiten de Traefik, deberán conectarse a esta red.
```plain 
docker network create --driver=overlay --attachable traefik
```
</details>

# Procedimientos de mantenimiento

<details>
<summary>ACTUALIZACIÓN DE SERVICIOS</summary>
    
> [!CAUTION]    
> En construcción...:-)

https://blog.foureight84.com/swarm-your-pihole/

</details>



<!-- Esto es un comentario -->

> [!NOTE]
> Highlights information that users should take into account, even when skimming.

> [!TIP]
> Optional information to help a user be more successful.

> [!IMPORTANT]
> Crucial information necessary for users to succeed.

> [!WARNING]
> Critical content demanding immediate user attention due to potential risks.

> [!CAUTION]
> Negative potential consequences of an action.



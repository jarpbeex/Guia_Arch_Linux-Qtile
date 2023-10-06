# Guía de instalación de Arch Linux con Qtile

## Configuración teclado y conectar internet (Cable o WIFI)
#### Configurar la distribución de teclado
- Para cargar nuestra distribución de teclado (Español)
	`loadkeys es`
#### Conectar a internet por cable
- Hacer una prueba de conexión:
	`ping google.com`
	Para salir: `CTRL + C` 
#### Conectar a internet por WIFI
- Para mostrar las conexiones WIFI:
	`ip link`
- Determinamos cual es nuestro adaptador WIFI para utilizar el siguiente comando:
	`ip link set nombre_adaptador up` con esto ya estaría levantado.
- Ahora escaneamos nuestras redes wifi con el comando:
	`iwlist nombre_adaptador scan | more`
- Una vez tengamos nuestro nombre de la wifi:
	- para conectarnos a una red sin clave:
		`iwconfig nombre_adaptador essid nombre_wifi`
	- conectarnos con contraseña WEP
		`iwconfig nombre_adaptador essid nombre_wifi key s:clave_wifi`
	- Conectarnos con contraseña WPA (Opción 1: probado)
		- comandos que use:
			`ip link set wlan0 up`
			`iwlist wlan0 scan | more`
			`wpa_passphrase nombre_red contraseña_red > /etc/wpa_supplicant/wifiConfig`
			`cat /etc/wpa_supplicant/wifiConfig`
			`wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wifiConfig -D wext`
			`dhclient wlan0`
			`ping 8.8.8.8`
	- conectarnos con contraseña WPA (Opción 2)
		- Introducimos nuestra contraseña:
			`wpa_passphrase nombre_red clave_wifi > /etc/WIFIConfiguration`
		- Revisamos que se haya guardado
			`cat /etc/WIFIConfiguration`
		- Ahora usando el fichero nos conectaremos al Wifi
			`wpa_supplicant -B -i nombre_adaptador -D wext -c /etc/WIFIConfiguration`
		- Para que quede nuestra configuración:
			`dhclient`
			y con esto ya estaría conectado al wifi.
## Particionar el disco (UEFI)
### UEFI - Crear particiones
- Verificar si es UEFI o BIOS
	Para verificar si es UEFI si da algún resultado es que si: `ls /sys/firmware/efi/`
- Crear 4 particiones usando el comando `cfdisk` y le damos a `GPT` por que es para UEFI:
	`dev/sda1` -> 512M para EFI system Tipo: EFI System
	`dev/sda2` -> este agarrara la mayoria de la memoria Tipo: Linux filesystem
	`dev/sda3` -> Esta agarrara un poco menos que la anterior Tipo: Linux filesystem
	`dev/sda4` -> Este seria la memoria de intercambio se le asigna el que queramos Tipo: Linux swap
	por ultimo darle a escribir antes de salir.
### Formatear las particiones
- Formateo del /boot
	`mkfs.vfat -F32 /dev/sda1`
	y con esto ya estaria formateada
- Formateo de las particions `/` y el `/home` con formato ext4
	- Para el `/`
		`mkfs.ext4 /dev/sda2`
	- Para el `/home`
		`mkfs.ext4 /dev/sda3`
- Formateo del área de intercambio swap
	`mkswap /dev/sda4`
	y una vez ejecutado el comando se utiliza: `swapon`
y ya tendríamos nuestras particiones creadas
### UEFI - Montar particiones
- Ahora montamos nuestra raíz: **(Directorio: /)**
	`mount /dev/sda2 /mnt`
- Seguimos con la partición ``boot``: **(Directorio: /mnt/boot/efi)**
	- necesitamos crear dos directorios dentro de nuestra carpeta temporal mnt:
		`mkdir /mnt/boot`
		`mkdir /mnt/boot/efi`
	- y montamos nuestra partición `boot`:
		`mount /dev/sda1 /mnt/boot/efi`
- Ahora queda montar el `/home` **(Directorio: /mnt/home)**
	`mkdir /mnt/home`
	`mount /dev/sda3 /mnt/home`
- Y con esto ya tendríamos todas las particiones montadas.
## Particionar el disco (BIOS)
[Crear, Formatear y Montar en BIOS](https://youtu.be/JcdxLgcmUkA)
## Instalación de paquetes base
- Para instalar los paquetes base:
	`pacstrap /mnt linux linux-firmware base nano grub networkmanager dhcpcd` y en caso de que estemos en UEFI añadimos: `efibootmgr` y le damos a instalar y esperamos que se instalen
## Instalar paquetes para WIFI
- Para instalar los paquetes WIFI: 
	`pacstrap /mnt netctl wpa_supplicant dialog` enter y esperar que se instalen.
## Generar fstab
- Generar el fichero de particiones fstab, para guardar nuestras particiones.
	`genfstab /mnt >> /mnt/etc/fstab`
- Para visualizarlo:
	`cat /mnt/etc/fstab`
## Configuración: Zona horaria, reloj y distribución de teclado
- Ya tendríamos nuestro sistema, asi que entraríamos a el con:
	`arch-chroot /mnt`
	y ahora estaríamos como root en nuestro sistema, ya podríamos configurarlo.
### Nombre del sistema
`echo nombre_del_sistema > /etc/hostname` . Y este seria el nombre de nuestro equipo
### Zona horaria
- Buscar nuestra zona horaria:
	`timedatectl list-timezones`
- Buscamos nuestra zona horaria y establecemos la nuestra:
	`ln -sf /usr/share/zoneinfo/America/Panama /etc/localtime`
### Configurar nuestro idioma
- Buscamos el archivo:
	`nano /etc/locale.gen`
- Dentro de nano buscamos nuestro idioma y quitamos el `#` para que ya no este comentado y guardamos `CTRL + O` Y Enter. y nos salimos.
- Después para que el sistema lo reconozca usamos el comando: `locale-gen`
### Reloj
- Es facil ya que hemos configurado nuestra zona horaria.
- Asi que cargamos con el comando: `hwclock -w`
- Si ejecutamos `date` veríamos si se configuro.
### Distribución de teclado
- Ahora toca dejar permanente el teclado en español, a diferencia de cargarlo con `loadkeys` ahora quedaría permanente por lo que `loadkeys` es solo temporal, cuando se haga un reinicio se va.
- Asi que ejecutamos el comando `echo KEYMAP=es > /etc/vconsole.conf` y enter.
- y luego: `echo LANG=es_PA.UTF8 > /etc/locale.conf` y ya estaria.
## Instalación GRUB en UEFI
- Para instalar utilizamos el siguiente comando:
	`grub-install --efi-directory=/boot/efi --bootloader -id='Arch Linux' --target=x86_64-efi`

	prueba
	`grub-install /dev/nvme0n1`
	
## Instalación GRUB en BIOS
[Instalación GRUB en BIOS](https://youtu.be/JcdxLgcmUkA)
## Configuración de GRUB
- ejecutamos el siguiente comando para configurar el grub:
	`grub-mkconfig -o /boot/grub/grub.cfg`
## Clave de ROOT y creación de nuestro usuario
### Configurar la contraseña del root
- ejecutamos el comando: 
	`passwd` le damos `enter` e introducimos la contraseña.
### Configuramos nuestro usuario
- Para esto ejecutamos:
	`useradd -m nombre_usuario`
### Configurar nuestra clave de usuario
- Ejecutamos:
	`passwd nombre_usuario` le damos `enter` e introducimos la contraseña.
### Reinicio
- Ahora toca reiniciar el sistema, para esto:
	- Salimos del root con:
		`exit`
	- Desmontamos nuestras particiones:
		`umount -R /mnt`
	- Y ahora hacemos el reinicio con:
		`reboot`
- Y podríamos ver como nos pide nuestro usuario y contraseña para al entrar.
## Activando y configurando internet (Cable o WIFI)
- Nos conectamos como root usando el comando `su` e introducimos la contraseña del root.
### Habilitamos nuestro servicio de red
- Ejecutamos:
	`systemctl start NetworkManager.service`
- y ahora lo habilitamos con:
	`systemctl enable NetworkManager.service`
### Conectarnos por cable
- Si ejecutamos `ping google.com` veríamos que tenemos conexión a internet.
### Conectarlo por WIFI
- Para ver nuestros adaptadores WIFI, ejecutamos:
	`ip link` 
- Activamos nuestro adaptador WIFI:
	`ip link set nombre_adaptador up`
- Ahora nos conectamos usando:
	`nmcli dev wifi connect nombre_wifi password clave_wifi`
	y estaríamos conectados por wifi.
## Drivers Gráficos
- ejecutamos: `lspci | grep VGA`
- Controlador genérico ahora ejecutamos: `pacman -S xf86-video-vesa`
- Para NVIDIA
	`pacman -S xf86-video-nouveau`
- Para Intel
	`pacman -S xf86-video-intel intel-ucode`
## Instalación XORG y MESA
- Ejecutamos e instalamos:
	`pacman -S xorg-server xorg-xinit mesa mesa-demos`
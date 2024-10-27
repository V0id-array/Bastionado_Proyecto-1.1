# Proyecto bastionado

![imagen.png](Proyecto%20bastionado%2012ac935dfacf804cbdbbfe57644bb998/imagen.png)

## Introducción
En este proyecto se explorarán opciones del sistema UEFI/BIOS y del sistema de arranque en Linux para mejorar la seguridad del sistema. Se elaborará una guía paso a paso de como podemos aplicar estos ajustes y así minimizar la superficie de ataque de nuesto equipo.

## Configuración de BIOS/UEFI

### Entorno:

En esta practica he trabajado con la BIOS/UEFI de un portatil Lenovo yoga 720 con procesador intel de 7ª generación. 

![imagen.png](Proyecto%20bastionado%2012ac935dfacf804cbdbbfe57644bb998/imagen%201.png)

### **Contraseña de administración de la BIOS:**

En la opción de **Security<”Set administrative password”** se puede añadir una contraseña para el acceso a la BIOS/UEFI. 

Es importante anotar esta contraseña ya que la única manera de reiniciarla es restaurando la configuración de fabrica de la placa base, lo cual requiere desmota el portátil y quitar la pila de la BIOS o hacer un puente un unos pines dedicados para ello. 

![imagen.png](Proyecto%20bastionado%2012ac935dfacf804cbdbbfe57644bb998/imagen%202.png)

![imagen.png](Proyecto%20bastionado%2012ac935dfacf804cbdbbfe57644bb998/imagen%203.png)

Para quitarla, en la misma opción de “**set administrative password**” nos permite cambiarla. Nos pide la contraseña actual y la nueva contraseña. En nueva contraseña se deja el campo en blanco. 

### **Contraseña de usuario de la BIOS y Contraseña de arranque del dispositivo**

Se realiza el mismo proceso que el paso anterior, esta vez se selecciona **Security<”Set user password”** (2) y se habilita **“Power on Password”** (1) 

![imagen.png](Proyecto%20bastionado%2012ac935dfacf804cbdbbfe57644bb998/imagen%204.png)

Esto hará que nos pida la contraseña cada vez que arranquemos el equipo para hacer cualquier acción con el. Ver imagen del apartado anterior.

Para quitarla es el mismo procedimiento que el caso anterior.

### Arranques externos y Orden de arranque

Para este apartado se ha deshabilitado el arranque por PXE(3) (imágenes enviadas por un servidor) y por dispositivos USB para evitar que nadie pueda intentar arrancar software e inspeccionar o modificar el contenido de nuestros discos. En caso de que el dispositivo tuviese lector de disco también sería conveniente desactivar el arranque por este medio. Por ultimo he mantenido el fastboot (1) deshabilitado ya que es una característica para sistemas Windows.

![Figura2 BIOS/UEFI de lenovo yoga 720 apartado boot](Proyecto%20bastionado%2012ac935dfacf804cbdbbfe57644bb998/imagen%205.png)

Figura2 BIOS/UEFI de lenovo yoga 720 apartado boot

Por último sería recomendable eliminar los discos que no tienen sistema operativo del orden de arranque, si no fuese posible ponerlas de ultima opción. Esto reduce la posible superficie de ataque. En este equipo solo se pude ver un dispositivo que es el del único disco duro disponible. 

 En otros equipos se puede ver el orden de arranque de la siguiente forma:

![image.png](Proyecto%20bastionado%2012ac935dfacf804cbdbbfe57644bb998/image.png)

### **Secure Boot**

El Secure Boot impide la ejecución en el arranque de software no firmado. Esto puede causar problemas con sistemas Linux, ya que el sistema de firmas lo gestionan principalmente Microsoft y los fabricantes. En mi investigación, encontré el siguiente dato interesante:

*Las versiones modernas de Ubuntu, a partir de Ubuntu 12.04.2 LTS y 12.10, normalmente arrancan e instalan sin problemas en la mayoría de los PC con Secure Boot activado. Esto se debe a que el cargador de arranque EFI de primera etapa de Ubuntu está firmado por Microsoft.*

En la BIOS/UEFI de esta máquina, la opción de Secure Boot se encuentra en la sección de **Security > "Secure Boot"**. También es importante arrancar el sistema en modo **UEFI**.

Desde allí se puede habilitar:

![imagen.png](Proyecto%20bastionado%2012ac935dfacf804cbdbbfe57644bb998/imagen%206.png)

## Otras opciones:

**-Intel PTT** [https://www.intel.la/content/www/xl/es/support/articles/000007452/intel-nuc.html](https://www.intel.la/content/www/xl/es/support/articles/000007452/intel-nuc.html)

Intel Platform Trust Technology PTT es una solución de seguridad que proporciona funcionalidades similares a las del Trusted Platform Module (TPM) pero sin necesidad de un chip físico dedicado. PTT se integra en el firmware del sistema, permitiendo la gestión de credenciales y claves criptográficas de manera segura. Es especialmente útil para funciones como el **cifrado de disco completo y la autenticación segura**

**-Intel SGX** [https://www.intel.la/content/www/xl/es/support/articles/000058952/software/intel-security-products.html](https://www.intel.la/content/www/xl/es/support/articles/000058952/software/intel-security-products.html)

Intel Software Guard Extensions (SGX) es una tecnología de 
seguridad que permite a los desarrolladores proteger datos y códigos 
sensibles mediante la creación de **enclaves**
seguros en la memoria. Estos enclaves son áreas de memoria que están 
aisladas y protegidas, lo que impide que cualquier código fuerael 
enclave acceda a su contenido, incluso si el sistema operativo está 
comprometido

AMD ofrece una alternativa a SGX con su suite de seguridad llamada **Infinity Guard**,
que incluye características como la protección de memoria y el cifrado 
de datos en reposo. Aunque no tiene una funcionalidad equivalente exacta
a SGX, Infinity Guard se centra en la seguridad del hardware y 
proporciona capacidades robustas para proteger datos en entornos de 
virtualización

![imagen.png](Proyecto%20bastionado%2012ac935dfacf804cbdbbfe57644bb998/imagen%207.png)

# Hardening del arranque de Linux y GRUB

## Entorno:

Se ha utilizado una máquina virtual en VirtualBox con Debian 12.

## **Ocultación del arranque**

Para esto se ha editado el archivo /etc/default/grub

`GRUB_TIMEOUT_STYLE=menu`  a `GRUB_TIMEOUT_STYLE=hidden`  y 

`GRUB_TIMEOUT=5` a `GRUB_TIMEOUT=0` 

![image.png](Proyecto%20bastionado%2012ac935dfacf804cbdbbfe57644bb998/image%201.png)

Para aplicar los cambios:

```bash
$ sudo update-grub
```

Tras estos cambios, al reiniciar no debería aparecer el menú del GRUB sino que directamente cargaría el sistema operativo.

## **Contraseña de arranque**

fuente: [https://linuxconfig.org/set-boot-password-with-grub](https://linuxconfig.org/set-boot-password-with-grub)

El primer paso es generar un hash de la contraseña utilizando un comando específico. Esto evita que la contraseña sea visible si alguien llegara a acceder al archivo de configuración de GRUB. 

![image.png](Proyecto%20bastionado%2012ac935dfacf804cbdbbfe57644bb998/image%202.png)

Para introducirla, se abre el siguiente archivo de configuración de GRUB:

```bash
$ sudo nano /etc/grub.d/00_header
```

Y se añaden las siguientes lineas al final, cambiar los campos `USUARIO` por el nombre de usuario que se desee e `INSERT-HASH` por el creado en el paso anterior.

```bash
cat << EOF
set superusers="USUARIO"
password_pbkdf2 USUARIO INSERT-HASH
EOF
```

para guardar cambios en el editor nano, pulsar “Control” + “o” y salir con “Control” + “x”

![image.png](Proyecto%20bastionado%2012ac935dfacf804cbdbbfe57644bb998/image%203.png)

Para aplicar los cambios se tiene que ejecutar un `update-grub` y probarse reiniciando el equipo
 

```bash
$ sudo update-grub
$ sudo reboot 
```

Resultado:

![image.png](Proyecto%20bastionado%2012ac935dfacf804cbdbbfe57644bb998/a3b7af2c-74f2-441b-a4bd-75149e1ca731.png)

## **Copia de seguridad**

fuente: [https://wiki.archlinux.org/title/GRUB/Tips_and_tricks](https://wiki.archlinux.org/title/GRUB/Tips_and_tricks)

Para esta practica los he copiado simplemente con el comando `cp`

Los archivos de configuración del GRUB se pueden copiar con los siguientes comandos (editar los PATH a los convenientes) :

```bash
sudo cp /etc/default/grub /path/to/backup/
sudo cp -r /etc/grub.d /path/to/backup/
sudo cp /boot/grub/grub.cfg /path/to/backup/
sudo cp /boot/grub/custom.cfg /path/to/backup
```

Para restaurar la configuración se tendría que copiar los archivos desde donde se hayan guardado y sustituir los que tenga el GRUB actualmente:

```bash
sudo cp /path/to/backup/ /etc/default/grub
sudo cp -r /path/to/backup/ /etc/grub.d 
sudo cp /path/to/backup/ /boot/grub/grub.cfg
sudo cp /path/to/backup /boot/grub/custom.cfg
```

Por último aplicar los cambios actualizando el GRUB con: 

```bash
$ sudo update-grub
```

## **Otras opciones de seguridad**

fuente: [https://soloconlinux.org.es/securizando-grub/](https://soloconlinux.org.es/securizando-grub/)

Otras medidas de seguridad para proteger el arranque pueden incluir encriptar las particiones del disco y modificar los parámetros del kernel. A continuación se explica de manera básica ambos procesos.

### **Encriptar los discos duros con LUKS**

Es necesario tener instalado cryptsetup. Se puede instalar con el siguiente comando en sistemas Debian/Ubuntu:

```bash
sudo apt install cryptsetup
```

Se requiere una partición vacía para la encriptación. Por ejemplo, se asume que la partición a encriptar es /dev/sdb2.

Para encriptar la partición, se ejecuta el siguiente comando:

```bash
sudo cryptsetup luksFormat /dev/sdb2
```

Se debe introducir una contraseña para proteger la partición.

Para acceder a la partición encriptada, se utiliza el siguiente comando:

```bash
sudo cryptsetup luksOpen /dev/sdb2 nombreParticion
```

Después de abrir la partición, es necesario formatearla con un sistema de archivos. Para formatearla como ext4:

```bash
sudo mkfs.ext4 /dev/mapper/nombreParticion
```

Se crea un punto de montaje y se monta la partición:

```bash
mkdir /mnt/particionsecreta
sudo mount /dev/mapper/nombreParticion /mnt/particionsecreta
```

El estado de la partición encriptada se puede verificar con el siguiente comando:

```bash
sudo cryptsetup -v status nombreParticion
```

Para desmontar y cerrar el volumen:

```bash
sudo umount /mnt/particionsecreta
sudo cryptsetup luksClose nombreParticion
```

Para montar la partición automáticamente al inicio, es necesario editar los archivos /etc/crypttab y /etc/fstab.

```bash
sudo nano /etc/crypttab
```

Se agrega la siguiente línea:

```
nombreParticion /dev/sdb2 none luks
```

```bash
sudo nano /etc/fstab
```

Se agrega la siguiente línea:

```
/dev/mapper/nombreParticion /mnt/particionsecreta ext4 defaults 0 2
```

Con esta configuración, se solicitará la contraseña para desbloquear la partición en cada inicio del sistema.

## Activar opciones del kernel al arranque:

Estas se pueden activar desde los parámetros de GRUB en el siguiente fichero `/etc/default/grub`

`GRUB_CMDLINE_LINUX_DEFAULT="quiet splash kaslr"`

Estas son algunas de las opciones que pueden mejorar las seguridad del sistema en el aranque:

- **CONFIG_DEBUG_SG=y**, **CONFIG_DEBUG_NOTIFIERS=y**, **CONFIG_DEBUG_CREDENTIALS=y**, **CONFIG_DEBUG_LIST=y**, **CONFIG_BUG_ON_DATA_CORRUPTION=y**, **CONFIG_SCHED_STACK_END_CHECK=y**: Estas configuraciones activan la validación de estructuras frecuentemente atacadas, como tablas SG, cadenas de llamadas de notificación, gestión de credenciales y manipulación de listas enlazadas.
- **CONFIG_DEBUG_VIRTUAL=y**: Activa verificaciones de integridad en el código que convierte direcciones virtuales a físicas, ayudando a prevenir ciertas corrupciones de datos.
- **CONFIG_INTEL_IOMMU_DEFAULT_ON=y**: Activa el IOMMU por defecto para proteger contra ataques DMA (Acceso Directo a Memoria).
- **CONFIG_GCC_PLUGIN_STRUCTLEAK=y**: Inicializa automáticamente las variables de pila a cero, mitigando vulnerabilidades relacionadas con memoria no inicializada. Utiliza la opción más robusta de STRUCTLEAK (BYREF_ALL).
- **CONFIG_GCC_PLUGIN_RANDSTRUCT=y**: Aleatoriza la disposición de las estructuras sensibles del kernel. No se debilita esta función con la opción RANDSTRUCT_PERFORMANCE.
- **CONFIG_GCC_PLUGIN_LATENT_ENTROPY=y**: Permite recopilar más entropía durante el arranque.
- **CONFIG_RESET_ATTACK_MITIGATION=y**: Previene ataques de arranque en frío sobrescribiendo la memoria con ceros al apagarse. Requiere soporte del firmware.
- **CONFIG_CRYPTO_JITTERENTROPY=y**, **CONFIG_HW_RANDOM_INTEL=y**, **CONFIG_HW_RANDOM_AMD=y**, etc.: Integran Jitterentropy y varios generadores de números aleatorios basados en hardware (HWRNG) para mejorar la entropía.
- **CONFIG_MODULE_SIG_ALL=y**, **CONFIG_MODULE_SIG_HASH="sha512"**: Firma todos los módulos del kernel durante la compilación usando SHA512, la función hash más robusta disponible. La verificación de firmas aún no se aplica.
- **CONFIG_PANIC_ON_OOPS=y**: Provoca que el kernel entre en pánico ante errores "oops", frustrando ciertos tipos de exploits. Solo se habilita para el kernel endurecido (hardened-vm-kernel) debido a posibles problemas en algunas máquinas host. Se recomienda que los hosts activen esta característica mediante el paquete `security-misc`.
- **CONFIG_MAGIC_SYSRQ_DEFAULT_ENABLE=0x84**: Limita la tecla SysRq para permitir únicamente las funciones SAK (Secure Attention Key) y apagados.

Para aplicar los cambios:

```bash
$ sudo update-grub
```

En el enlace de la fuente se pueden encontrar más opciones con las que para customizar el kernel o descargarnos. [https://www.kicksecure.com/wiki/Hardened-kernel](https://www.kicksecure.com/wiki/Hardened-kernel) 

También podemos descargarnos la configuración del proyecto [hardened-kernel](https://www.kicksecure.com/wiki/Download)

## Conclusión 
El hardening en Linux es un proceso continuo que requiere atención y adaptación a medida que surgen nuevas amenazas. Implementar las mejores prácticas descritas no solo ayuda a proteger el sistema contra ataques, sino que también contribuye al cumplimiento de normativas y estándares de seguridad. Al adoptar un enfoque proactivo hacia la seguridad, los administradores pueden garantizar que sus sistemas Linux operen en un entorno seguro y resiliente. 
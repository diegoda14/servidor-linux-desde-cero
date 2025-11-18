Título: Proyecto personal – Servidor Linux desde cero en VirtualBox

Descripción general
He montado y documentado un servidor Linux minimalista (Ubuntu Server) dentro de una máquina virtual en VirtualBox, simulando el entorno de un servidor real conectado a la red local. El objetivo del proyecto es practicar y demostrar habilidades de administración de sistemas

El servidor se ha configurado con:

	Acceso remoto seguro por SSH con claves.

	Gestión correcta de usuarios, grupos y permisos.

	Protección mediante firewall y fail2ban.

	Gestión de servicios con systemd.

	Sistema de copias de seguridad automáticas con rsync.

Puede reproducirse fácilmente en cualquier entorno similar.

Paso 1 – Instalación del servidor en VirtualBox

	Creé una máquina virtual en VirtualBox con una distribución Linux de servidor (Ubuntu Server ) usando una instalación mínima, sin entorno gráfico. La red de la máquina está configurada en modo “Bridge” para que el servidor reciba una IP propia dentro de la red local mediante DHCP, como si fuera un equipo físico más.

	Durante la instalación se configuró un usuario inicial, el hostname del servidor y los paquetes básicos necesarios para trabajar por terminal. El resultado es un servidor limpio, ligero y preparado para ser administrado remotamente.

Paso 2 – Usuarios, grupos y sudoers

	Para separar correctamente privilegios y no trabajar directamente como root, creé un usuario administrativo adicional y un grupo específico para administradores. Este grupo es el que tiene permisos para utilizar sudo.

	En lugar de modificar el fichero principal de sudo, utilicé la ruta recomendada /etc/sudoers.d/ para definir las reglas de sudo de forma más segura y modular. Con esta configuración, cualquier usuario añadido a ese grupo puede usar sudo, y el resto de usuarios del sistema quedan sin privilegios de administración.

	También se revisaron los permisos del fichero de configuración de sudo para asegurarse de que nadie sin privilegios pueda modificarlo.

Paso 3 – SSH seguro (claves, endurecimiento y fail2ban)

	Para el acceso remoto configuré SSH siguiendo buenas prácticas:

	Generé un par de claves SSH en el equipo cliente (PC real).

	Registré solo la clave pública en el servidor, de forma que únicamente ese cliente puede autenticarse como el usuario autorizado.

	Endurecí la configuración de SSH desactivando:
	
		El acceso directo con el usuario root.

		La autenticación por contraseña.

	Con esta configuración, el acceso solo es posible usando claves SSH, lo que reduce mucho el riesgo de ataques por fuerza bruta.

	Además, instalé y configuré fail2ban para que monitorice los intentos de acceso por SSH y bloquee automáticamente las direcciones IP que fallen repetidas veces al intentar autenticarse. Esto añade una capa extra de protección frente a bots y ataques automatizados.

Paso 4 – Firewall con UFW

	Activé y configuré el firewall del sistema utilizando UFW (Uncomplicated Firewall), que es una capa sencilla sobre iptables.

	La política por defecto del servidor es:

		Bloquear todas las conexiones entrantes.

		Permitir todas las conexiones salientes.

		Después, habilité únicamente el puerto de SSH que se está utilizando, de manera que el servidor solo expone al exterior el servicio estrictamente necesario. Todo lo demás queda cerrado por defecto. De esta forma, se aplica el principio de “mínima superficie de ataque”.

Paso 5 – Logs y journalctl

	Para el diagnóstico y la monitorización, revisé los mecanismos de logging del sistema:

	Ficheros tradicionales en /var/log, como los de autenticación y sistema general.

	El sistema de logs de systemd a través de journalctl, que permite:

		Ver todos los logs del sistema.

		Filtrar por servicio (por ejemplo, SSH o fail2ban).

		Ver entradas en tiempo real.

		Limitar el tamaño del journal para evitar que ocupe demasiado espacio.

	Con esto, el servidor queda preparado para investigar problemas de inicio, errores de servicios, intentos de acceso fallidos y cualquier incidente de seguridad o estabilidad.

Paso 6 – Servicios con systemd

	Aprendí a gestionar servicios con systemd, que es el sistema de inicio estándar en distribuciones modernas:

		Iniciar, detener, reiniciar y consultar el estado de servicios.

		Configurar servicios para que se ejecuten automáticamente al arrancar el sistema.

	Además, creé un servicio propio de ejemplo, asociado a un pequeño script que escribe en un log. Esto sirve para entender cómo definir unidades de systemd, cómo recargar la configuración y cómo revisar sus logs específicos. Es un ejercicio práctico que replica lo que se haría al desplegar un servicio personalizado o una aplicación en producción.

Paso 7 – Script de backup con rsync y automatización periódica

	Para añadir una funcionalidad útil y realista, implementé un sistema de copias de seguridad:

		Creé un directorio protegido en el servidor para almacenar los backups.

		Desarrollé un script que realiza una copia de seguridad del directorio de usuarios (por ejemplo, /home) hacia esa carpeta de backup utilizando rsync.

El script incluye marcas de tiempo en las carpetas de destino.

	Mantiene un log propio con la hora de inicio, fin y resultado de cada ejecución.

	Configuré una automatización para que el backup se ejecute de forma periódica una vez al día, utilizando un timer de systemd (alternativa moderna a cron), de modo que:

		El backup se lanza automáticamente a una hora determinada.

		Si el equipo ha estado apagado, el timer puede recuperar y ejecutar la tarea al arrancar.

	De este modo, el servidor dispone de un sistema de copia de seguridad automatizado y trazable, muy similar a lo que se encontraría en un entorno de producción sencillo.

Verificación y pruebas

Al finalizar la configuración, verifiqué que:

	Solo es posible acceder al servidor por SSH usando claves; no se aceptan contraseñas y el usuario root no puede iniciar sesión remotamente.

	El firewall está activo, con políticas restrictivas y únicamente los puertos necesarios abiertos.

	Fail2ban está vigilando el servicio SSH y reacciona ante intentos de acceso fallidos repetidos.

	Los logs del sistema y de los servicios son accesibles y permiten identificar intentos de intrusión, problemas de servicios y errores de configuración.

	El servicio de backup se ejecuta correctamente, crea directorios de copia con fecha y deja constancia en un log específico.


FIN

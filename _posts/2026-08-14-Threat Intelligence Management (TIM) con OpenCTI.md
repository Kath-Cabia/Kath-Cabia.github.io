---
title: "Lab.05 - Laboratorio — Levantar tu TIM"
date: 2026-08-14
categories: [lab, setup]
tags: [OpenCTI, kali]
---

# **LABORATORIO — Levantar el Threat Intelligence Management (TIM) con OpenCTI**

## **Lab: Detección y Explotación**

# PASO 1: Acondicionar la VM Kali v.1
## 1.1. RAM y CPU (GUI de VirtualBox)
Con la máquina virtual (VM) apagada, se accedió a ***Configuración → Sistema*** y se establecieron los siguientes recursos:

- Placa base → Memoria base: **8192 MB (8 GB)**
- Procesador: **4 CPUs**

![T16.1](/assets/TIM/T16.1.png)

## 1.2. Segundo disco de 25 GB para Docker
En ***Configuración → Almacenamiento***, dentro del Controlador SATA, se agregó un segundo disco virtual con las siguientes características:

- Tipo de archivo: **VDI**
- Almacenamiento: **Reservado dinámicamente**
- Tamaño: **25 GB**

Ruta de configuración: 
***Agregar disco duro → Crear → VDI → Reservado dinámicamente → 25 GB.***

![T16.2](/assets/TIM/T16.2.png)
![T16.3](/assets/TIM/T16.3.png)
![T16.4](/assets/TIM/T16.4.png)

**NOTA:** 
- Verificación crítica dentro de Kali antes de formatear. 
- Arranca Kali y corre **lsblk**. 
- El disco a formatear es el que NO tiene particiones ni punto de montaje (el nuevo, vacío). 
- Las letras **sda/sdb** pueden salir intercambiadas, nunca formatees el disco que tiene / montado.

## 1.3 - Identificar el nuevo disco
Para identificar el disco agregado y verificar las versiones instaladas de ***Docker*** y 
***Docker Compose***, ejecutar:
Ingresar:

    > lsblk
    > docker --version          # Docker 26.x o superior
    > docker compose version    # Compose v2.x

Nota:
- Verificar siempre con lsblk antes de realizar cualquier operación de formateo.
- El disco que se va a formatear debe ser el que no tenga particiones ni punto de montaje.
- Nunca formatear el disco que tiene / montado, ya que corresponde al sistema operativo.

![T1](/assets/TIM/T1.png)
![T2](/assets/TIM/T2.png)

# 1.4. Formatear SOLO el disco nuevo 
- Identificar correctamente el disco nuevo antes de realizar el formateo. 
- Reemplazar **sdX** por el nombre correspondiente al dispositivo (por ejemplo, sda, sdb o sdc).
- En este caso, el disco nuevo corresponde a **sdb**.

Ingresar:

    > sudo mkfs.ext4 /dev/sdb

![T3](/assets/TIM/T3.png)

## 1.5. Detener Docker antes de mover su directorio y
##      Mover datos previos y montar el disco nuevo en /var/lib/docker

Antes de mover el directorio de datos de Docker, se debe **detener** el servicio para evitar que se estén utilizando archivos durante el proceso.

Ejecutar las siguientes líneas de código, reemplazando **sdX** por el disco correspondiente. 
En este caso, **sdX corresponde a sdb**:

Ingresar:

    > sudo systemctl stop docker docker.socket
    > sudo mv /var/lib/docker /var/lib/docker.old 2>/dev/null; sudo mkdir /var/lib/docker
    > UUID=$(sudo blkid -s UUID -o value /dev/sdX)
    > echo "UUID=$UUID /var/lib/docker ext4 defaults 0 2" | sudo tee -a /etc/fstab
    > sudo systemctl daemon-reload
    > sudo mount /var/lib/docker

![T4](/assets/TIM/T4.png)

 **Nota:** Verificar que el UUID obtenido corresponde al disco nuevo antes de continuar. 
 La configuración en /etc/fstab permite que el disco se monte automáticamente en /var/lib/docker al iniciar el sistema.


## 1.6. Reiniciar Docker y confirmar
Una vez montado el nuevo disco, iniciar nuevamente el servicio de Docker y verificar que el directorio de almacenamiento y el espacio disponible sean correctos:

Ingresar:

    sudo systemctl start docker
    docker info | grep "Docker Root Dir"     # Debe mostrar /var/lib/docker
    df -h /var/lib/docker                    # Debe mostrar aproximadamente 25 GB

La primera línea permite confirmar que Docker utiliza /var/lib/docker como directorio raíz de almacenamiento, mientras que la segunda verifica que el nuevo disco de aproximadamente 
25 GB se encuentra correctamente montado.

![T5](/assets/TIM/T5.png)

# PASO 2: Levantar el TIM

La contraseña se encuentra almacenada en texto plano dentro del archivo .env, por lo que puede recuperarse mediante una búsqueda en dicho archivo.
En un entorno de producción, las credenciales deben gestionarse mediante un gestor de secretos, evitando almacenarlas directamente en archivos de configuración.
Por ello, durante este procedimiento, se recomienda tratar la contraseña como una credencial que no debe volver a consultarse ni exponerse innecesariamente.

# 2.1. Clonar el laboratorio en Kali
Para obtener el código del laboratorio en la máquina virtual Kali, clonar el repositorio mediante:

    > git clone https://github.com/ollerenacastro/untels-tim-lab.git

![T6](/assets/TIM/T6.png)

# 2.2. Generar credenciales (.env con passwords aleatorios)
Ingresar al directorio del laboratorio y ejecutar el script de configuración para generar las credenciales de forma automática:

    > cd untels-tim-lab
    > ./scripts/setup-env.sh

 **NOTA:** Registrar las credenciales generadas y almacenarlas de forma segura.
 Si es necesario recuperar la contraseña de OpenCTI, puede consultarse con:

    > grep OPENCTI_ADMIN_PASSWORD .env

 **Importante:** Evitar exponer o compartir las credenciales contenidas en el archivo .env.

![T7](/assets/TIM/T7.png)

# 2.3. Levantar todos los servicios
Para construir las imágenes y levantar todos los servicios definidos en el archivo ***docker-compose.yml***.
Ejecutar:
    
    > docker compose up -d --build

El parámetro ***-d*** permite ejecutar los servicios en segundo plano, mientras que ***--build*** fuerza la construcción de las imágenes antes de iniciar los contenedores.

![T9](/assets/TIM/T9.png)
![T9.1](/assets/TIM/T9.1.png)
![T9.2](/assets/TIM/T9.2.png)
![T9.3](/assets/TIM/T9.3.png)

# 2.4. Esperar a que OpenCTI importe MITRE ATT&CK (~5–15 min)

Una vez iniciados los servicios, esperar a que OpenCTI complete la importación de los objetos de MITRE ATT&CK. 
Este proceso puede tardar aproximadamente **entre 5 y 15 minutos**, dependiendo de los recursos asignados a la máquina virtual.
Para verificar el progreso, ejecutar:

    > ./scripts/verify-platform.sh   #Termina cuando hay 100+ objetos ATT&CK importados.

El script finaliza cuando se han importado 100 o más objetos de MITRE ATT&CK, lo que permite confirmar que la plataforma está correctamente inicializada.

![T10](/assets/TIM/T10.png)

# 2.4. Acceder a OpenCTI
Cuando el script diga “Platform ready”, abre en el navegador de Kali:
Login con **admin@tim.local** y la **password** que guardaste en el paso 2.

    http://localhost:8080

Iniciar sesión utilizando:
- Usuario: admin@tim.local
- Contraseña: la registrada durante el **PASO 2.**

Esto permitirá acceder a la interfaz web de OpenCTI y verificar que la plataforma se encuentra operativa.

![T11](/assets/TIM/T11.png)
![T11.1](/assets/TIM/T11.1.png)

2.6. Retomar el laboratorio

**NOTA:** Si la máquina virtual fue suspendida o apagada, al volver a iniciarla no es necesario levantar los contenedores manualmente. 
Ejecutar:

    ```bash
    ./scripts/restart-lab.sh
    ```

Este script reinicia los servicios en el orden correcto. 
Esto es importante porque, después de reanudar la VM, **OpenCTI** puede iniciar antes que Elasticsearch y dejar el acceso por el **puerto 8080** sin funcionar correctamente.
El script realiza un reinicio limpio de los contenedores y espera a que los servicios necesarios estén disponibles.
Los datos no se pierden, ya que están almacenados en los volúmenes de Docker. 
El script no elimina estos volúmenes.

## COMANDOS ADICIONALES 
A continuación, se presentan algunos comandos útiles para administrar y verificar el laboratorio TIM.

### Reiniciar el laboratorio
Úsalo al iniciar una clase o después de suspender/apagar la VM:

    ```bash
    ./scripts/restart-lab.sh
    ```

### Ver el estado de los servicios
Permite comprobar si los contenedores están funcionando correctamente:

    ```bash
    docker compose ps
    ```

### Ver los logs de un servicio
Por ejemplo, para revisar el conector de MITRE ATT&CK:

    ```bash
    docker compose logs -f connector-mitre
    ```

### Consultar los IOCs obtenidos por el feed-orchestrator

    ```bash
    curl -s http://localhost:8001/feeds/status | python3 -m json.tool
    ```

### Exportar los IOCs en formato STIX 2.1
El siguiente comando muestra la cantidad de objetos STIX exportados:

    ```bash
    curl -s http://localhost:8001/feeds/export/stix | python3 -c "import sys,json; print(len(json.load(sys.stdin)['objects']), 'objetos STIX')"
    ```

### Apagar el TIM conservando los datos

    ```bash
    docker compose down
    ```

> Los volúmenes y los datos almacenados se mantienen.

### Apagar y eliminar todos los datos

    ```bash
    docker compose down -v
    ```

> **ADVERTENCIA:** 
Este comando elimina los volúmenes de Docker y, por tanto, **borra los datos del laboratorio**. 
Utilizarlo solo si se desea comenzar nuevamente desde cero.

# **Conectores (ingesta de inteligencia)**
## **A. Conector de AlienVault OTX**

### Paso 1 — Obtener la clave 
Para configurar el conector de AlienVault OTX, primero es necesario obtener la clave de API.

1. Regístrate gratuitamente en AlienVault OTX y confirma tu correo electrónico.
2. Inicia sesión y, en la parte superior derecha, abre el menú de usuario y selecciona Settings.
3. Busca la sección OTX Key y copia la clave mostrada.
4. Abre el archivo .env:

    ```bash
    nano .env
    ```

Registra la clave en la variable correspondiente, sin comillas ni espacios.
**Importante:**
La clave es una credencial privada. **NO** la compartas ni la publiques en el repositorio.

![T12](/assets/TIM/T12.png)

**NOTA:** El archivo .env contiene credenciales y está excluido del control de versiones. No debe subirse a ningún repositorio ni compartirse.
Además de la clave de OTX, este archivo contiene la contraseña de administrador de OpenCTI, por lo que debe tratarse como información confidencial.
Para editar el archivo .env, ingresar al directorio del laboratorio y ejecutar:

    cd ~/untels-tim-lab
    nano .env

![T12.1](/assets/TIM/T12.1.png)

## **Activar el feed del orquestador**
Por qué ***up -d*** **reinicia** el proceso dentro del mismo contenedor, que conserva las variables de entorno con las que fue creado. 
Es la diferencia entre reiniciar un programa y volver a lanzarlo con parámetros nuevos.

Ingresar:
```bash
    docker compose up -d feed-orchestrator
    #Se espera unos minutos para verificar con:
    curl -s http://127.0.0.1:8001/feeds/status | python3 -m json.tool 
```

![T12.2](/assets/TIM/T12.2.png)
![T12.3](/assets/TIM/T12.3.png)

### Paso 2 — Conector de MALWAREBAZAAR & THREATFOX**
Las otras dos fuentes **MalwareBazaar y ThreatFox** (ambos de ***abuse.ch***) comparten una única clave obtenida en 
***https://auth.abuse.ch**:

- Ingresar en el terminal **nano .env** y pegar lo siguiente incluyendo la clave generada.

```bash
        nano .env
        docker compose up -d feed-orchestrator

        MALWAREBAZAAR_AUTH_KEY=tu_clave_de_abusech
        THREATFOX_AUTH_KEY=tu_clave_de_abusech
        curl -s http://127.0.0.1:8001/feeds/status | python3 -m json.tool
```
Sale ERROR en la ingestación de los 2 conectores.

![T13.1](/assets/TIM/T13.1.png)
![T13.2](/assets/TIM/T13.2.png)








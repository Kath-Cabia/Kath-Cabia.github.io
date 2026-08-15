---
title: "Lab.05 - Laboratorio — Levantar tu TIM"
date: 2026-08-14
categories: [lab, setup]
tags: [OpenCTI, kali]
---

# **LABORATORIO — Levantar el Threat Intelligence Management (TIM) con OpenCTI**

## **Lab: Detección y Explotación**

# PASO 1: Acondicionar la VM Kali v.1
## 1.1 — RAM y CPU (GUI de VirtualBox)
Con la VM apagada → Configuración → Sistema:

- Placa base → Memoria base: 8192 MB
- Procesador: 4 CPUs

![T16.1](/assets/TIM/T16.1.png)

## 1.2 — Segundo disco de 25 GB para Docker
En la GUI: Configuración → Almacenamiento → Controlador SATA → 
Agregar disco duro → Crear → VDI → Reservado dinámicamente → 25 GB.

![T16.2](/assets/TIM/T16.2.png)
![T16.3](/assets/TIM/T16.3.png)
![T16.4](/assets/TIM/T16.4.png)

NOTA: 
- Verificación crítica dentro de Kali antes de formatear. 
- Arranca Kali y corre **lsblk**. 
- El disco a formatear es el que NO tiene particiones ni punto de montaje (el nuevo, vacío). 
- Las letras sda/sdb pueden salir intercambiadas — nunca formatees el disco que tiene / montado.

## 1.3 - Identificar el nuevo disco
Ingresar:

    lsblk
    docker --version          # Docker 26.x o superior
    docker compose version    # Compose v2.x

NOTA: 
- Verifica siempre con lsblk antes de formatear un disco. 
- El disco a formatear es el que no tiene particiones ni punto de montaje. 
- Nunca formatees el disco que tiene / montado.

![T1](/assets/TIM/T1.png)
![T2](/assets/TIM/T2.png)

# 1.4. Formatear SOLO el disco nuevo 
Reemplaza sdX por el correcto (p.ej. sda, sdb, sdc)
Ingresar:

    sudo mkfs.ext4 /dev/sdb
![T3](/assets/TIM/T3.png)

## 1.5. Detener Docker antes de mover su directorio y
##      Mover datos previos y montar el disco nuevo en /var/lib/docker
Ingresar:

    sudo systemctl stop docker docker.socket
    sudo mv /var/lib/docker /var/lib/docker.old 2>/dev/null; sudo mkdir /var/lib/docker
    UUID=$(sudo blkid -s UUID -o value /dev/sdX)
    echo "UUID=$UUID /var/lib/docker ext4 defaults 0 2" | sudo tee -a /etc/fstab
    sudo systemctl daemon-reload
    sudo mount /var/lib/docker

![T4](/assets/TIM/T4.png)

## 1.6. Reiniciar Docker y confirmar
Ingresar:

    sudo systemctl start docker
    docker info | grep "Docker Root Dir"     # debe decir /var/lib/docker
    df -h /var/lib/docker                     # debe mostrar ~25G 

![T5](/assets/TIM/T5.png)

# PASO 2: Levantar el TIM

IMPORTANTE: 
Sobre la **contraseña**. Vive en texto plano en tu .env, así que recuperarla es un grep. 
Apúntala igual: en cualquier despliegue real las credenciales están en un gestor de secretos y ese grep no existe. 
Acostúmbrate a tratarla como si no pudieras volver a leerla.

# 2.1. Clonar el laboratorio dentro de Kali

    git clone https://github.com/ollerenacastro/untels-tim-lab.git

![T6](/assets/TIM/T6.png)

# 2.2. Generar credenciales (.env con passwords aleatorios)
    
    cd untels-tim-lab
    ./scripts/setup-env.sh

NOTA: APÚNTALA. Si la pierdes: grep OPENCTI_ADMIN_PASSWORD .env

![T7](/assets/TIM/T7.png)

# 2.3. Levantar todo con un solo comando
    
    docker compose up -d --build

![T9](/assets/TIM/T9.png)
![T9.1](/assets/TIM/T9.1.png)
![T9.2](/assets/TIM/T9.2.png)
![T9.3](/assets/TIM/T9.3.png)

# 2.4. Esperar a que OpenCTI importe MITRE ATT&CK (~5–15 min)
   
    ./scripts/verify-platform.sh   #Termina cuando hay 100+ objetos ATT&CK importados.

![T10](/assets/TIM/T10.png)

Cuando el script diga “Platform ready”, abre en el navegador de Kali:
Login con **admin@tim.local** y la **password** que guardaste en el paso 2.

    http://localhost:8080

![T11](/assets/TIM/T11.png)
![T11.1](/assets/TIM/T11.1.png)
    NOTA: Al retomar el lab (tras suspender o apagar la VM): 
    Usa ./scripts/restart-lab.sh. 

    Al reanudar la VM los contenedores vuelven sin respetar el orden de arranque: 
        OpenCTI sube antes que Elasticsearch, crashea y toma una IP nueva → el puerto 8080 del host queda "muerto" 
        (aunque docker compose ps diga healthy). restart-lab.sh hace down + up limpio y reordena por healthcheck. 
        Los datos persisten (viven en volúmenes; down sin -v no los borra).

## COMANDOS ADICIONALES 

    # Reinicio LIMPIO — úsalo al empezar clase o tras suspender/apagar la VM
    ./scripts/restart-lab.sh

    # Ver estado de todos los servicios
    docker compose ps

    # Ver logs de un servicio (p.ej. el conector de ATT&CK)
    docker compose logs -f connector-mitre

    # Cuántos IOCs vivos trajo el feed-orchestrator
    curl -s http://localhost:8001/feeds/status | python3 -m json.tool

    # Exportar todos los IOCs como bundle STIX 2.1
    curl -s http://localhost:8001/feeds/export/stix | python3 -c "import sys,json; print(len(json.load(sys.stdin)['objects']), 'objetos STIX')"

    # Apagar el TIM (conserva los datos)
    docker compose down

    # Apagar y BORRAR todos los datos (empezar de cero)
    docker compose down -v

# **Conectores (ingesta de inteligencia)**
## **A. Conector de AlienVault OTX**

### Paso 1 — Obtener la clave (común a ambas vías)
1. Regístrate gratis en **https://otx.alienvault.com** y confirma el correo.
2. Inicia sesión, abre el menú de tu usuario (arriba a la derecha) → Settings.
3. En la sección **OTX Key** copia la clave (una cadena hexadecimal larga).
   Regístrala en **nano .env**, sin comillas ni espacios:

![T12](/assets/TIM/T12.png)

NOTA: ***nano .env*** contiene credenciales y está excluido del control de versiones. 
No lo subas a ningún repositorio ni lo compartas: además de esta clave guarda la contraseña de administrador de tu OpenCTI.
    cd ~/untels-tim-lab    #ingresar a la terminal de tim-lab
    nano .env

![T12.1](/assets/TIM/T12.1.png)

## **Activar el feed del orquestador**
Por qué ***up -d*** **reinicia** el proceso dentro del mismo contenedor, 
que conserva las variables de entorno con las que fue creado. 
Es la diferencia entre reiniciar un programa y volver a lanzarlo con parámetros nuevos.

Ingresar:

    docker compose up -d feed-orchestrator
    #Se espera unos minutos para verificar con:
    curl -s http://127.0.0.1:8001/feeds/status | python3 -m json.tool 

![T12.2](/assets/TIM/T12.2.png)
![T12.3](/assets/TIM/T12.3.png)

## **B. Conector de MALWAREBAZAAR & THREATFOX**
Las otras dos fuentes
MalwareBazaar y ThreatFox (ambos de abuse.ch) comparten una única clave obtenida en 
https://auth.abuse.ch:

- Ingresar en el terminal **nano .env** y pegar lo siguiente incluyendo la clave generada.

    nano .env
    docker compose up -d feed-orchestrator

    MALWAREBAZAAR_AUTH_KEY=tu_clave_de_abusech
    THREATFOX_AUTH_KEY=tu_clave_de_abusech
    curl -s http://127.0.0.1:8001/feeds/status | python3 -m json.tool

![T13.1](/assets/TIM/T13.1.png)
![T13.2](/assets/TIM/T13.2.png)

Sale ERROR en la ingestación de los 2 conectores.






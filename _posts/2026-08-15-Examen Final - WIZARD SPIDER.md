---
title: "Examen Final — [Wizard Spider]"
date: 2026-08-15
categories: [Examen, Threat Intelligence]
tags: [examen-final]
---
# **ACTO 1 — LA INTELIGENCIA DIRIGE (OpenCTI del curso)**

Ingresar al ***OpenCTI*** del servidor del curso con las credenciales temporales y elige un grupo APT para analizar.

![1](/assets/ACT1/1.png)

## **IMPORTANTE:**
En base a la asignación:
APT asignado: Wizard Spider	
Técnica ATT&CK: T1210 Exploitation of Remote Services

Para armar el perfil, se busca en **OpenCTI o en MITRE ATT&CK**. :
El ATP asignado **se obtuvo de OpenCTI** y se obtuvieron los siguientes datos:

- **Intrusion Set:** WIZARD SPIDER
- **Identidad:** G0102 
- **Author:** The MITRE CORPORATION
- **Entity aliases:**

    - New alias
    - UNC1878
    - TEMP.MixMaster
    - Grim Spider
    - FIN12
    - GOLD BLACKBURN
    - ITG23
    - Periwinkle Tempest
    - DEV-0193
    - Pistachio Tempest
    - DEV-0237

**Motivación:** 
Wizard Spider is a Russia-based financially motivated threat group originally known for the creation and deployment of TrickBot since at least 2016. Wizard Spider possesses a diverse arsenal of tools and has conducted ransomware campaigns against a variety of organizations, ranging from major corporations to hospitals.(Citation: CrowdStrike Ryuk January 2019)(Citation: DHS/CISA Ransomware Targeting Healthcare October 2020)(Citation: CrowdStrike Wizard Spider October 2020)

![2.1](/assets/ACT1/2.1.png)
![2.1](/assets/ACT1/2.2.png)

- **Indicators/IoCs:** No se encontraron indicadores directamente asociados a Wizard Spider en la instancia de **OpenCTI** consultada ni en **MITRE ATT&CK**

![2.3](/assets/ACT1/2.3.png)

- **Malware vinculado** (TrickBot, Ryuk, Conti...).

![2.4](/assets/ACT1/2.4.png)


Se seleccionaron 7 tácticas en Attack Patterns.
![3.1](/assets/ACT1/3.1.png)
![3.2](/assets/ACT1/3.2.png)

## **1. External ID: T1071**
- ***Táctica:*** Command and Control
- ***Técnica:*** Application Layer Protocol
- ***Nivel de inteligencia:*** Táctico
- ***Vértice del modelo Diamante:*** Capability
- ***Justificación:*** Describe cómo el atacante utiliza protocolos de aplicación para establecer o mantener C2. 
Aporta información sobre la infraestructura empleada para comunicarse con el objetivo.

![4.1](/assets/ACT1/4.1.png)

## **2. External ID: T1585**
- ***Táctica:*** Resource development
- ***Técnica:*** Establish Accounts
- ***Nivel de inteligencia:*** Capability
- ***Vértice del modelo Diamante:*** Adversario
- ***Justificación:*** Describe una actividad realizada por el grupo para preparar recursos antes de una operación. 
Permite conocer cómo actúa y se prepara el adversario.

![4.2](/assets/ACT1/4.2.png)

## **3. External ID: T1053**
- ***Táctica:*** Execution (Ejecución) / Persistence (Persistencia) / Privilege Escalation (Escalada de privilegios)
- ***Técnica:*** Scheduled Task/Job
- ***Nivel de inteligencia:*** Táctica
- ***Vértice del modelo Diamante:*** Capability
- ***Justificación:*** Describe una técnica concreta utilizada para ejecutar acciones, mantener persistencia o escalar privilegios. 
Representa el comportamiento técnico del atacante.

![4.3](/assets/ACT1/4.3.png)

## **4. External ID: T1133**
- ***Táctica:*** Persistence (Persistencia) / Initial Access (Acceso Inicial)
Técnica: External Remote Services
Nivel de inteligencia: Táctica
Vértice del modelo Diamante: Capability
Justificación: Describe el uso de servicios remotos externos para acceder al entorno objetivo. Se relaciona con los medios técnicos utilizados para obtener acceso

![4.4](/assets/ACT1/4.4.png)

## **5. External ID: T1005**
- ***Táctica:*** Collection
- ***Técnica:*** Data from Local System
- ***Nivel de inteligencia:*** Táctica
- ***Vértice del modelo Diamante:*** Capability
- ***Justificación:*** Describe la recopilación de información almacenada en el sistema afectado. 
Permite identificar qué información del objetivo puede ser buscada o recolectada.

![4.5](/assets/ACT1/4.5.png)

## **6. External ID: T1041**
- ***Táctica:*** Exfiltration
- ***Técnica:*** Exfiltration Over C2 Channel
- ***Nivel de inteligencia:*** Táctica
- ***Vértice del modelo Diamante:*** Capability
- ***Justificación:*** Describe la extracción de información mediante el canal de C2. 
Se relaciona con la infraestructura utilizada para transportar los datos robados.

![4.6](/assets/ACT1/4.6.png)

## **7. External ID: T1685**
- ***Táctica:*** Defense impairment
- ***Técnica:*** Disable or Modify Tools
- ***Nivel de inteligencia:*** Táctica
- ***Vértice del modelo Diamante:*** Capability
- ***Justificación:*** Describe una acción concreta del atacante para deshabilitar o modificar herramientas de seguridad y dificultar su detección.

![4.7](/assets/ACT1/4.7.png)

## **8. External ID: T1210**
- ***Táctica:*** Lateral movement
- ***Técnica:*** Exploitation of Remote Services 
- ***Nivel de inteligencia:*** Táctica
- ***Vértice del modelo Diamante:*** Capability
- ***Justificación:*** Describe la explotación de servicios remotos para realizar movimiento lateral. 
La acción corresponde al adversario, mientras que el servicio remoto explotado forma parte de la infraestructura/vía utilizada.
 
![4.8](/assets/ACT1/4.8.png)

***
# **ACTO 2: Plan de ataque mapeado a ATT&CK
Estas tablas permiten relacionar las TTPs identificadas en el Acto 1 con las acciones que realmente pueden realizarse dentro del laboratorio utilizando Metasploitable3. De esta manera, se determina cuáles de las técnicas tienen un equivalente práctico en el entorno y qué herramienta o procedimiento puede utilizarse para reproducirlas.
La **Tabla 1** muestra el análisis de cada TTP y permite identificar cuáles pueden llevarse a cabo en el laboratorio y cuáles se descartan debido a las características del entorno. 
La **Tabla 2** resume las técnicas seleccionadas, indicando la táctica, el objetivo o servicio involucrado y la herramienta que se utilizará para realizar la prueba.

## ***Tabla 1. Análisis de las TTPs del APT y su aplicación en Metasploitable3***

| ID     | Técnica                           | ¿Tiene análogo   |                            ¿Por qué?                                                                |
|        |                                   |   en el lab?     |                                                                                                     |
| :----- | :-------------------------------: | :--------------: | -------------------------------------------------------------------------------------------------- :| 
| T1071  | Application Layer Protocol (C2)   |         Sí       | Cuando abro la sesión con Meterpreter, esa conexión ya es un canal de C2.                           |
|        |                                   |                  | Si en vez del payload normal uso uno por HTTPS, se ve más claro que el atacante                     |
|        |                                   |                  | controla la máquina usando un protocolo "normal" de internet para camuflarse.                       |
| T1585  | Establish Accounts                |         No       | Esto es crear cuentas falsas (redes sociales, correos) antes del ataque, para preparar el terreno.  | 
|        |                                   |                  | Es algo que pasa afuera, en internet, no dentro de mi red de laboratorio.                           |
|        |                                   |                  | Como mi lab está aislado y sin internet, no hay forma de reproducirlo.                              |
| T1053  | Scheduled Task/Job                |         Sí       | Una vez que tengo control total (SYSTEM) de la máquina, puedo crear una tarea programada para que,  |
|        |                                   |                  | aunque me desconecte, el sistema me "abra la puerta" otra vez más tarde. Eso sí lo puedo hacer con  |
|        |                                   |                  | un simple comando de Windows.                                                                       |
| T1133  | External Remote Services          |         No       | Esta técnica es para cuando el atacante entra por un servicio que da al internet, como una VPN.     |
|        |                                   |                  | En mi lab no hay "afuera" ni "adentro" — mi Kali y Metasploitable3 están en la misma red chica, así |
|        |                                   |                  | que no existe ese salto externo→interno que pide la técnica.                                        |
| T1005  | Data from Local System            |         Sí       | Es simplemente buscar y leer archivos dentro de la máquina que ya controlo. Con search y cat en     |
|        |                                   |                  | **Meterpreter** lo hago directo.                                                                    |
| T1041  | Exfiltration Over C2 Channel      |         Sí       | Es sacar ese archivo de la víctima hacia mi Kali, usando la misma conexión que ya abrí con el       |
|        |                                   |                  | **exploit**. Con el comando download de Meterpreter queda demostrado.                               |
| T1685  | Disable or Modify Tools           |         Sí       | Con permisos de SYSTEM puedo apagar el firewall o el antivirus de la víctima con un comando, igual  | 
|        |                                   |                  | que haría un atacante para no ser detectado.                                                        |
| T1210  | Exploitation of Remote Services   |  Sí (dentro del  | Es la vulnerabilidad de **SMB (puerto 445)** que exploto con **EternalBlue** para entrar a la máquina.      |
|        |                                   |  ATP asignado)   |                                                                                                     |  




***




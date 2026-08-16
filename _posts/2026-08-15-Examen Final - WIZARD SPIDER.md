---
title: "Examen Final — [Wizard Spider]"
date: 2026-08-15
categories: [Examen, Threat Intelligence]
tags: [examen-final]
---
# **ACTO 1 — LA INTELIGENCIA DIRIGE (OpenCTI del curso)**

Ingresar al ***OpenCTI*** del servidor del curso con las credenciales temporales y elige un grupo APT para analizar.

![T1](/assets/ACT1/T1.png)

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

![T2.1](/assets/ACT1/T2.1.png)
![T2.1](/assets/ACT1/T2.2.png)

- **Indicators/IoCs:** No se encontraron indicadores directamente asociados a Wizard Spider en la instancia de **OpenCTI** consultada ni en **MITRE ATT&CK**

![T2.3](/assets/ACT1/T2.3.png)

- **Malware vinculado** (TrickBot, Ryuk, Conti...).

![T2.4](/assets/ACT1/T2.4.png)


Se seleccionaron 7 tácticas en Attack Patterns.
![T3.1](/assets/ACT1/T3.1.png)
![T3.2](/assets/ACT1/T3.2.png)

## **1. External ID: T1071**
- ***Táctica:*** Command and Control
- ***Técnica:*** Application Layer Protocol
- ***Nivel de inteligencia:*** Táctico
- ***Vértice del modelo Diamante:*** Capability
- ***Justificación:*** Describe cómo el atacante utiliza protocolos de aplicación para establecer o mantener C2. 
Aporta información sobre la infraestructura empleada para comunicarse con el objetivo.

![T4.1](/assets/ACT1/T4.1.png)

## **2. External ID: T1585**
- ***Táctica:*** Resource development
- ***Técnica:*** Establish Accounts
- ***Nivel de inteligencia:*** Capability
- ***Vértice del modelo Diamante:*** Adversario
- ***Justificación:*** Describe una actividad realizada por el grupo para preparar recursos antes de una operación. 
Permite conocer cómo actúa y se prepara el adversario.

![T4.2](/assets/ACT1/T4.2.png)

## **3. External ID: T1053**
- ***Táctica:*** Execution (Ejecución) / Persistence (Persistencia) / Privilege Escalation (Escalada de privilegios)
- ***Técnica:*** Scheduled Task/Job
- ***Nivel de inteligencia:*** Táctica
- ***Vértice del modelo Diamante:*** Capability
- ***Justificación:*** Describe una técnica concreta utilizada para ejecutar acciones, mantener persistencia o escalar privilegios. 
Representa el comportamiento técnico del atacante.

![T4.3](/assets/ACT1/T4.3.png)

## **4. External ID: T1133**
- ***Táctica:*** Persistence (Persistencia) / Initial Access (Acceso Inicial)
Técnica: External Remote Services
Nivel de inteligencia: Táctica
Vértice del modelo Diamante: Capability
Justificación: Describe el uso de servicios remotos externos para acceder al entorno objetivo. Se relaciona con los medios técnicos utilizados para obtener acceso

![T4.4](/assets/ACT1/T4.4.png)

## **5. External ID: T1005**
- ***Táctica:*** Collection
- ***Técnica:*** Data from Local System
- ***Nivel de inteligencia:*** Táctica
- ***Vértice del modelo Diamante:*** Capability
- ***Justificación:*** Describe la recopilación de información almacenada en el sistema afectado. 
Permite identificar qué información del objetivo puede ser buscada o recolectada.

![T4.5](/assets/ACT1/T4.5.png)

## **6. External ID: T1041**
- ***Táctica:*** Exfiltration
- ***Técnica:*** Exfiltration Over C2 Channel
- ***Nivel de inteligencia:*** Táctica
- ***Vértice del modelo Diamante:*** Capability
- ***Justificación:*** Describe la extracción de información mediante el canal de C2. 
Se relaciona con la infraestructura utilizada para transportar los datos robados.

![T4.6](/assets/ACT1/T4.6.png)

## **7. External ID: T1685**
- ***Táctica:*** Defense impairment
- ***Técnica:*** Disable or Modify Tools
- ***Nivel de inteligencia:*** Táctica
- ***Vértice del modelo Diamante:*** Capability
- ***Justificación:*** Describe una acción concreta del atacante para deshabilitar o modificar herramientas de seguridad y dificultar su detección.

![T4.7](/assets/ACT1/T4.7.png)

## **8. External ID: T1210**
- ***Táctica:*** Lateral movement
- ***Técnica:*** Exploitation of Remote Services 
- ***Nivel de inteligencia:*** Táctica
- ***Vértice del modelo Diamante:*** Capability
- ***Justificación:*** Describe la explotación de servicios remotos para realizar movimiento lateral. 
La acción corresponde al adversario, mientras que el servicio remoto explotado forma parte de la infraestructura/vía utilizada.
 
![T4.8](/assets/ACT1/T4.8.png)






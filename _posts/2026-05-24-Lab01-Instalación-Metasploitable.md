---
title: "Lab.01 - Instalación de Metasploitable 3"
date: 2026-05-24
categories: [lab, setup]
tags: [metasploitable, Virtualbox(VM), kali]
---

# **¿QUÉ ES LA VM METASPLOITABLE 3?**
*Este es mi primer post sobre la instalación de la Máquina Virtual Metasploitable 3* 

Es una máquina virtual vulnerable utilizada en laboratorios de ciberseguridad para practicar análisis de seguridad y aprender cómo funcionan distintas vulnerabilidades en un entorno controlado. En este laboratorio se utilizó para simular una máquina con fallos de seguridad que puede ser analizada desde Kali Linux. 

La diferencia entre un archivo ***".ova"*** e ***".iso"*** es que el **".ova"** contiene una máquina virtual ya configurada y lista para importarse en VirtualBox o VMware, mientras que el **".iso"** es una imagen de instalación de un sistema operativo que requiere una configuración manual antes de poder utilizarse.

##**Creación de LA VM Metasploitable 3**
**PASO 1:**
Metasploitable 3 windows viene como archivo **.iso** y **.ova**, en esta guía para crear la máquina virtual se está montando el .ova. Asimismo, dejar por defecto la sección Configuración

Montando **.iso** al VM:
![Ima.0.0](/assets/Ima.0.0.md.png)

![Ima.0.1](/assets/Ima.0.1.md.png)

Máquina virtual creada
![Ima.0.0.0](/assets/Ima.0.0.0.md.png)

## **CONFIGURACIÓN INTERNA DE LA VM METASPLOITABLE 3**

**PASO 2:** 
Durante la creación de la máquina virtual se seleccionó el sistema operativo **Microsoft Windows** y la versión Windows Server 2008 de 64 bits, correspondiente a la configuración requerida para Metasploitable 3.

![Ima.1.1](/assets/Ima.1.1.md.png)

Luego, se procede a configurar el sistema a 2Gb (2048MB), conectar la **red a NAT**, activar el **Hyper-V**, como se observa en las siguientes imágenes de forma correspondiente.

![Ima.1.2](/assets/Ima.1.2.md.png)

Conexión a la **red NAT**
![Ima.1.3](/assets/Ima.1.3.md.png)

Se ingresa en la configuración de Metasploitable, luego:
1. sistema/Aceleración/Paravirtualization Interface 
2. seleccionar Hyper-V.
![Ima.1.4](/assets/Ima.1.4.md.png)
 
**PASO 3:**
El ajuste de la red de VirtualBox, se realiza ingresando a:
1. Archivos/Herramientas/Red/Redes NAT. 

Si no se visualiza, **crear la red NAT (NatNetwork)**

Red NAT creada, con rango de ipv4. 10.0.2.0/24 con el servido DHCP --> HABILITADO]
![Ima.1.5](/assets/Ima.1.5.md.png)

# **CONFIGURACIÓN DE LA RED NATNETWORK**
PASO 4:
En la VM-Metasploitable, se selecciona la VM/Configuración/Red/Adaptador 1/Conectar a/Red NAT y se procede a Aceptar/Guardar y se cierra, los mismos pasos se realizan en la VM-Kali Linux.

En Metasploitable 3
![Ima.1.6](/assets/Ima.1.6.md.png)

En Kali-Linux
![Ima.1.7](/assets/Ima.1.7.md.png)

Usamos Host-Only porque crea una red privada y aislada donde solo se comunican el host y las VMs, sin acceso a internet ni a la red física externa. Con NAT las VMs podrían salir a internet pero es más complicado que se vean entre sí, y con Bridged quedarían expuestas directamente en tu red local, lo cual es riesgoso teniendo una máquina vulnerable como Metasploitable. Entonces, Host-Only es la opción más segura y controlada para un laboratorio de pruebas.

## **VERIFICACIÓN DE CONECTIVIDAD**
**PASO 5:**
## **En Metasploitable**
- Se inicia el VM-Windows.
- Al aparecer windows, dirigirnos en la barra superior ENTRADA/Teclado/Insertar Ctrl-Alt-Del, y luego arrancará con (Administrador --> usuario: vagrant, contraseña: vagrant)

![Ima.1.8](/assets/Ima.1.8.md.png)

## **En Kali-Linux**
- Se inicia el VM.
- Al aparecer windows, dirigirnos en la barra superior ENTRADA/Teclado/Insertar Ctrl-Alt-Del, y luego arrancará con (Administrador --> usuario: user, contraseña: user123)

![Ima.1.9](/assets/Ima.1.9.md.png)

## **Acción:**
En cada VM, abrimos sus respectivas terminales y verificamos la conectividad.
Primero ingresamos *ipconfig (en Metasploitable) e ip address* (en Kali Linux) para conocer sus direcciones IPv4.

![Ima.1.10](/assets/Ima.1.10.md.png)

![Ima.1.11](/assets/Ima.1.11.md.png)

Realizamos ping de uno al otro con las direcciones generadas:
![Ima.1.12](/assets/Ima.1.12.md.png)

##**Ambos muestran un pin exitoso.**
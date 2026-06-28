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

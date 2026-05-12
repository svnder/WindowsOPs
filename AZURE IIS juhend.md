
# IIS seadistamine — Windows Server

> **Keskkond:** Windows Server (Azure VM)
> **Eeldus:** Oled ühendatud serveriga RDP kaudu

Allpool on kaks varianti — vali endale sobiv.

---

## Versioon A — PowerShelli kaudu

### 1. IIS rolli paigaldamine

Ava PowerShell administraatorina ja käivita:

```powershell
Install-WindowsFeature -Name Web-Server -IncludeManagementTools
```

Kontrolli et teenus töötab:

```powershell
Get-Service -Name W3SVC
```

> Olek peab olema `Running`. Kui ei ole:

```powershell
Start-Service -Name W3SVC
```

### 2. Veebilehe asendamine

Kopeeri HTML kood esmalt lõikelauale, seejärel käivita:

```powershell
Set-Content -Path "C:\inetpub\wwwroot\index.html" -Value (Get-Clipboard) -Encoding UTF8
```

### 3. Windowsi tulemüür

Kontrolli et HTTP reegel on olemas:

```powershell
Get-NetFirewallRule -DisplayName "*HTTP*" | Select-Object DisplayName, Enabled, Direction
```

Kui reegel puudub:

```powershell
New-NetFirewallRule -DisplayName "Allow HTTP" -Direction Inbound -Protocol TCP -LocalPort 80 -Action Allow
```

---

## Versioon B — graafilise liidesega (GUI)

### 1. IIS rolli paigaldamine

Ava Server Manager. Klõpsa Manage ja vali Add Roles and Features. Järgi juhiseid kuni Server Roles sammuni ja vali Web Server (IIS). Klõpsa Next ja Install. Oota kuni paigaldamine lõpeb.

### 2. Veebilehe asendamine

Ava Notepad administraatorina — parem klikk Notepadil ja Run as administrator. Kopeeri HTML kood sisse. Vali File, Save As ja navigeeri asukohta `C:\inetpub\wwwroot`. Nimeta fail `index.html` ja salvesta. Kui küsitakse ülekirjutamist, kinnita jah.

### 3. Windowsi tulemüür

Ava Windows Firewall with Advanced Security. Vali Inbound Rules ja New Rule. Vali Port, klõpsa Next. Sisesta port 80, klõpsa Next. Vali Allow the connection, klõpsa Next kaks korda. Anna reeglile nimi Allow HTTP ja klõpsa Finish.

---

## Azure Network Security Group

Mõlema versiooni puhul peab port 80 olema avatud Azure'i tulemüüris. Ilma selleta ei pääse brauser lehele ligi isegi kui IIS töötab.

```
Azure portaal → Virtual Machine → Networking → Add inbound port rule
```

| Väli | Väärtus |
|------|---------|
| Destination port | 80 |
| Protocol | TCP |
| Action | Allow |
| Name | Allow-HTTP |

---

## Tulemuse kontrollimine

Leia serveri avalik IP:

```
Azure portaal → Virtual Machine → Overview → Public IP address
```

Ava brauser ja sisesta:

```
http://SERVERI_IP
```

---

## Levinumad probleemid

**Brauser ei ava lehte**
- Kontrolli et port 80 on Azure NSG-s avatud
- Kontrolli et IIS teenus töötab
- Kasuta `http://` mitte `https://`

**IIS kuvab vaikimisi lehte, mitte sinu lehte**
- Veendu et fail on täpselt `C:\inetpub\wwwroot\index.html`
- Failinimi ei tohi olla `index.html.txt` — kontrolli faililaiendeid

**Set-Content annab vea (versioon A)**
- Veendu et PowerShell on avatud administraatorina
- Kontrolli et kaust on olemas: `Test-Path "C:\inetpub\wwwroot"`

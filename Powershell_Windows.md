# PowerShell — Windows Serveri haldus käsurealt

---

## Sisukord

1. [PowerShelli põhitõed](#1-powershelli-põhitõed)
2. [Võrgu seadistamine](#2-võrgu-seadistamine)
3. [Active Directory — domeeni loomine](#3-active-directory--domeeni-loomine)
4. [Active Directory — kasutajate haldus](#4-active-directory--kasutajate-haldus)
5. [Active Directory — grupid](#5-active-directory--grupid)
6. [Grupipoliitika (GPO)](#6-grupipoliitika-gpo)
7. [CSV masshaldus](#7-csv-masshaldus)
8. [PowerShell remoting — kaughaldus](#8-powershell-remoting--kaughaldus)
9. [Jagatud kaustad](#9-jagatud-kaustad)
10. [DHCP](#10-dhcp)
11. [Kasutaja konto haldus](#11-kasutaja-konto-haldus)
12. [Süsteemi monitooring](#12-süsteemi-monitooring)
13. [Auditeerimispoliitika ja logid](#13-auditeerimispoliitika-ja-logid)
14. [Skriptid](#14-skriptid)
15. [PowerShell profiil](#15-powershell-profiil)
16. [Credentials haldus](#16-credentials-haldus)

---

## 1. PowerShelli põhitõed

PowerShell on Microsofti käsurida ja skriptikeel, mis on mõeldud süsteemide haldamiseks. Erinevalt vanast `cmd`-ist töötab PowerShell **objektidega** — iga käsu vastus on objekt millel on väljad ja meetodid.

### Põhilised navigeerimiskäsud

```powershell
# Vaata praegust kausta sisu
ls
# või
dir
# või
Get-ChildItem

# Mine kausta
cd C:\Users

# Mine üles
cd ..

# Mine C-draivi juurde
cd C:\

# Vaata faili sisu
cat C:\fail.txt
# või
Get-Content C:\fail.txt
```

### Tulemuste filtreerimine ja sorteerimine

```powershell
# Vali ainult teatud veerud
Get-ADUser -Filter * | Select-Object Name, Enabled

# Filtreeri tulemusi
Get-Process | Where-Object {$_.CPU -gt 10}

# Sorteeri tulemusi
Get-Process | Sort-Object CPU -Descending

# Võta ainult esimesed 10
Get-Process | Select-Object -First 10
```

**Selgitus `|` (pipe):** Saadab eelmise käsu tulemuse järgmisele käsule edasi. Näiteks `Get-Process | Sort-Object CPU` võtab protsessid ja sordib need CPU järgi.

**Selgitus `$_`:** Tähendab "praegune objekt". Kasutatakse `Where-Object` ja `foreach` sees.

### Värvid ja tekst ekraanile

```powershell
# Kirjuta tekst ekraanile
Write-Host "Tere maailm!"

# Kirjuta värviga
Write-Host "See on roheline!" -ForegroundColor Green
Write-Host "See on tsüaan!" -ForegroundColor Cyan
Write-Host "See on kollane!" -ForegroundColor Yellow

# Tühja rea lisamine
Write-Host "`n"
```

### Muutujad

```powershell
# Defineeri muutuja
$nimi = "Jaan"
$arv = 42

# Kasuta muutujat
Write-Host "Tere, $nimi!"

# Muutuja käsu sees
$kasutaja = Get-ADUser -Identity "jtamm"
```

### Kasutaja sisend

```powershell
# Küsi kasutajalt sisendit
$nimi = Read-Host "Sisesta oma nimi"
Write-Host "Tere, $nimi!"
```

---

## 2. Võrgu seadistamine

Enne domeeni seadistamist peab serveril olema staatiline IP-aadress — IP mis ei muutu kunagi. Klient leiab serveri alati sama aadressi kaudu.

### Adapteri nime kontroll

```powershell
# Vaata mis nimi on võrguadapteril
Get-NetAdapter
```

> **Tähelepanu:** Server 2012 peal võib adapteri nimi olla `Local Area Connection` mitte `Ethernet`. Kontrolli alati enne IP seadistamist ja asenda vastavalt.

### Staatilise IP seadistamine

```powershell
# Seadista staatiline IP serverile
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.100.1 -PrefixLength 24

# Seadista DNS — server osutab iseendale
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.100.1
```

**Selgitus parameetrid:**
- `-InterfaceAlias` — adapteri nimi, kontrolli `Get-NetAdapter` käsuga
- `-IPAddress` — IP-aadress mida soovid seadistada
- `-PrefixLength 24` — võrgumask `/24` ehk `255.255.255.0`
- `-ServerAddresses` — DNS serveri aadress

```powershell
# Seadista staatiline IP kliendile
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.100.2 -PrefixLength 24 -DefaultGateway 192.168.100.1

# Seadista DNS kliendil — osutab serverile
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.100.1
```

### Ühenduse testimine

```powershell
# Testi kas server on kättesaadav
ping 192.168.100.1
```

### Serveri ümber nimetamine

```powershell
# Nimeta server ümber ja taaskäivita
Rename-Computer -NewName "Server2022" -Restart
```

---

## 3. Active Directory — domeeni loomine

Active Directory Domain Services (AD DS) on teenus mis haldab kasutajaid, arvuteid ja poliitikat domeenis.

### AD DS rolli paigaldamine

```powershell
# Paigalda AD DS roll koos haldus tööriistadega
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

**Selgitus `-IncludeManagementTools`:** Paigaldab ka AD PowerShelli mooduli — ilma selleta ei saa `New-ADUser`, `Get-ADUser` jt käske kasutada.

```powershell
# Kontrolli et roll on paigaldatud
Get-WindowsFeature -Name AD-Domain-Services
```

### Domeeni loomine

```powershell
# Loo uus forest ja domeen
Install-ADDSForest -DomainName "lab.local" -InstallDns
```

**Selgitus:**
- `-DomainName` — domeeni nimi, `.local` on kohalik domeen
- `-InstallDns` — paigaldab automaatselt ka DNS serveri

> **Tähelepanu:** Server taaskäivitub automaatselt pärast domeeni loomist. See on normaalne — ära katkesta protsessi.

### Domeeni kontrollimine

```powershell
# Kontrolli et domeen on olemas
Get-ADDomain
```

**Mida otsida:** `DNSRoot: lab.local`

### Kliendi domeeniga liitumine

```powershell
# Liitu domeeniga (käivita kliendi PowerShellis)
Add-Computer -DomainName "lab.local" -Credential lab\Administrator -Restart

# Kontrolli pärast taaskäivitust
Get-WmiObject -Class Win32_ComputerSystem | Select-Object Name, Domain
```

**Mida otsida:** `Domain: lab.local`

---

## 4. Active Directory — kasutajate haldus

### Kasutaja loomine

```powershell
New-ADUser `
    -Name "Jaan Tamm" `
    -SamAccountName "jtamm" `
    -UserPrincipalName "jtamm@lab.local" `
    -Path "OU=Kasutajad,DC=lab,DC=local" `
    -AccountPassword (ConvertTo-SecureString "Parool123!" -AsPlainText -Force) `
    -Enabled $true
```

**Selgitus parameetrid:**
- `-Name` — kasutaja täisnimi
- `-SamAccountName` — lühike sisselogimisnimi (jtamm)
- `-UserPrincipalName` — täis sisselogimisnimi koos domeeniga (jtamm@lab.local)
- `-Path` — kus AD-s kasutaja asub (OU ja domeen)
- `-AccountPassword` — parool turvalises formaadis
- `-Enabled $true` — konto on kohe aktiivne

### Kasutajate vaatamine

```powershell
# Kõik kasutajad domeenis
Get-ADUser -Filter *

# Kõik kasutajad kindlas OU-s
Get-ADUser -Filter * -SearchBase "OU=Kasutajad,DC=lab,DC=local"

# Vali ainult vajalikud veerud
Get-ADUser -Filter * -SearchBase "OU=Kasutajad,DC=lab,DC=local" | Select-Object Name, Enabled

# Üks konkreetne kasutaja
Get-ADUser -Identity "jtamm"

# Kasutaja koos lisaväljadega
Get-ADUser -Identity "jtamm" -Properties LockedOut, PasswordExpired, PasswordNeverExpires
```

### Kasutaja muutmine

```powershell
# Muuda kasutaja andmeid
Set-ADUser -Identity "jtamm" -PasswordNeverExpires $false -ChangePasswordAtLogon $true
```

### Kasutaja kustutamine

```powershell
# Kustuta kasutaja
Remove-ADUser -Identity "jtamm"
```

---

## 5. Active Directory — grupid

### OU loomine

```powershell
# Loo organisatsiooniüksus (OU)
New-ADOrganizationalUnit -Name "Kasutajad" -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Arvutid" -Path "DC=lab,DC=local"

# Kontrolli
Get-ADOrganizationalUnit -Filter *
```

**Selgitus OU:** Organisatsiooniüksus on kaust AD-s kuhu saab kasutajaid ja arvuteid grupeerida. GPO-d rakendatakse OU tasandil.

### Grupi loomine

```powershell
# Loo grupp
New-ADGroup -Name "Opilased" -GroupScope Global -Path "OU=Kasutajad,DC=lab,DC=local"
```

**Selgitus `-GroupScope Global`:** Grupp on kättesaadav kogu domeenis.

### Kasutajate lisamine gruppi

```powershell
# Lisa üks kasutaja
Add-ADGroupMember -Identity "Opilased" -Members "jtamm"

# Lisa mitu kasutajat korraga
Add-ADGroupMember -Identity "Opilased" -Members "jtamm","mmets","pkask"

# Kontrolli grupi liikmed
Get-ADGroupMember -Identity "Opilased"
```

---

## 6. Grupipoliitika (GPO)

GPO (Group Policy Object) on reeglistik mis rakendub kasutajatele või arvutitele. GPO seotakse OU-ga — kõik selle OU kasutajad saavad GPO reeglid.

### GPO loomine

```powershell
# Loo uus GPO
New-GPO -Name "Opilaste poliitika"

# Kontrolli
Get-GPO -Name "Opilaste poliitika"
```

### GPO linkimine OU-ga

```powershell
# Lingi GPO Kasutajad OU-ga
New-GPLink -Name "Opilaste poliitika" -Target "OU=Kasutajad,DC=lab,DC=local"

# Kontrolli linkimine
Get-GPInheritance -Target "OU=Kasutajad,DC=lab,DC=local"
```

### GPO registriväärtuste seadistamine

```powershell
# Keela Control Panel kasutajatele
Set-GPRegistryValue `
    -Name "Opilaste poliitika" `
    -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
    -ValueName "NoControlPanel" `
    -Type DWord `
    -Value 1

# Seadista tarkvara juurutus
Set-GPRegistryValue `
    -Name "Opilaste poliitika" `
    -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" `
    -ValueName "SoftwareDeploy" `
    -Type String `
    -Value "\\Server2022\Software\programm.msi"

# Kontrolli registriväärtus
Get-GPRegistryValue `
    -Name "Opilaste poliitika" `
    -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Run"
```

**Selgitus registriväärtuse tüübid:**
- `DWord` — täisarv (0 või 1 — keela/luba)
- `String` — tekstiväärtus

---

## 7. CSV masshaldus

CSV (Comma-Separated Values) on tekstifail kus andmed on komadega eraldatud. Kasutatakse suurte kasutajahulkade automaatseks loomiseks.

### CSV faili struktuur

```
Eesnimi,Perenimi,Kasutajanimi
Toomas,Sepp,tsepp
Kati,Tamm,ktamm
Mihkel,Kask,mkask
```

**Reeglid:**
- **Esimene rida** — päis ehk veerunimed
- **Iga järgnev rida** — üks kirje
- **Komad** — eraldavad veerge
- **Encoding UTF8** — tagab eesti tähtede korrektse salvestamise

### CSV faili loomine PowerShellis

```powershell
$csv = @"
Eesnimi,Perenimi,Kasutajanimi
Toomas,Sepp,tsepp
Kati,Tamm,ktamm
Mihkel,Kask,mkask
Liisa,Mets,lmets
Andres,Rand,arand
"@
$csv | Out-File -FilePath "C:\kasutajad.csv" -Encoding UTF8
```

### CSV-st kasutajate loomine

```powershell
# Loe CSV fail
$kasutajad = Import-Csv -Path "C:\kasutajad.csv"

# Käi läbi iga rida ja loo kasutaja
foreach ($k in $kasutajad) {
    New-ADUser `
        -Name "$($k.Eesnimi) $($k.Perenimi)" `
        -SamAccountName $k.Kasutajanimi `
        -UserPrincipalName "$($k.Kasutajanimi)@lab.local" `
        -Path "OU=Kasutajad,DC=lab,DC=local" `
        -AccountPassword (ConvertTo-SecureString "Parool123!" -AsPlainText -Force) `
        -Enabled $true
}
```

**Selgitus `foreach`:** Käib läbi kõik read ükshaaval. `$k` on praegune rida — `$k.Eesnimi` võtab selle rea `Eesnimi` veeru väärtuse.

**Selgitus `$($k.Eesnimi)`:** Sulud `$()` on vajalikud kui kasutad objekti välja teksti sees — `"$($k.Eesnimi) $($k.Perenimi)"` annab näiteks `Toomas Sepp`.

```powershell
# Kontrolli et kõik kasutajad loodi
Get-ADUser -Filter * -SearchBase "OU=Kasutajad,DC=lab,DC=local" | Select-Object Name
```

---

## 8. PowerShell remoting — kaughaldus

PowerShell remoting võimaldab hallata serverit kliendi terminallist. See on realistlik administraatori tööstsenaarium — päris elus ei istu administraator serveri taga.

### Remotingu lubamine serveris

```powershell
# Käivita serveris
Enable-PSRemoting -Force

# Kontrolli et remoting töötab
Get-PSSessionConfiguration
```

### Credentials salvestamine

```powershell
# Salvesta kasutajanimi ja parool muutujasse
# Küsib parooli üks kord graafilises aknas
$cred = Get-Credential lab\Administrator
```

**Selgitus:** `$cred` muutuja sisaldab kasutajanime ja parooli. Kasuta seda kõikides `Invoke-Command` käskudes — nii ei pea parooli iga kord uuesti sisestama.

> **Tähelepanu:** `$cred` kaob kui PowerShell suletakse. Ava uus PowerShell ja käivita uuesti.

### Enter-PSSession — interaktiivne sessioon

```powershell
# Ava interaktiivne sessioon serveriga
Enter-PSSession -ComputerName Server2022 -Credential $cred
```

Prompt muutub: `[Server2022]: PS C:\>`

Nüüd saad serveris navigeerida ja käske jooksutada nagu oleksid serveris kohapeal:

```powershell
# Navigeerimine serveris
cd C:\
ls
cd Software
cat kasutajad.csv
cd ..
```

```powershell
# Väljumine sessioonist — tagasi klienti
Exit-PSSession
```

**Selgitus:**
- `Enter-PSSession` — lähed serverisse, teed mitu käsku, tuled tagasi
- `[Server2022]:` promptis — oled serveris
- `Exit-PSSession` — tuled tagasi klienti

### Invoke-Command — üksik käsk serverile

```powershell
# Saada üksik käsk serverile
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    # See kood jookseb serveris
    Get-ADUser -Filter * -SearchBase "OU=Kasutajad,DC=lab,DC=local"
}
```

**Selgitus:** `Invoke-Command` saadab käsu serverile ja vastus tuleb tagasi klienti. Sa jääd klienti — ei lähe interaktiivseks.

**Selgitus `PSComputerName: Server2022`:** See väli igas vastuses näitab et käsk jooksis serveris — visuaalne kinnitus et kaughaldus töötab.

### $using: — muutujate edastamine serverile

Kui muutuja on defineeritud kliendis aga kasutad seda `Invoke-Command` sees serveris, pead kasutama `$using:` prefiksit:

```powershell
# Muutuja on kliendis
$kasutajanimi = "uususer"

# Kasuta serveris $using: prefiksiga
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    New-ADUser `
        -Name $using:kasutajanimi `
        -SamAccountName $using:kasutajanimi `
        -Path "OU=Kasutajad,DC=lab,DC=local" `
        -AccountPassword (ConvertTo-SecureString "Parool123!" -AsPlainText -Force) `
        -Enabled $true
}
```

**Ilma `$using:`** annab vea — server ei tea kliendi muutujast.

---

## 9. Jagatud kaustad

Jagatud kaust on kaust serveris millele pääseb ligi üle võrgu. Kasutatakse tarkvara jagamiseks ja failide salvestamiseks.

### Jagatud kausta loomine

```powershell
# Loo kaust
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    New-Item -Path "C:\Software" -ItemType Directory
}

# Jaga kaust võrgus
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    New-SmbShare -Name "Software" -Path "C:\Software" -FullAccess "Everyone"
}

# Kontrolli jagatud kaust
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Get-SmbShare -Name "Software"
}
```

### Jagatud kaustale ligipääs kliendilt

```powershell
# Vaata jagatud kausta sisu kliendilt
dir \\Server2022\Software\

# Kopeeri fail jagatud kausta
Copy-Item -Path "C:\fail.txt" -Destination "\\Server2022\Software\"
```

---

## 10. DHCP

DHCP (Dynamic Host Configuration Protocol) jagab automaatselt IP-aadresse võrgu seadmetele.

### DHCP rolli paigaldamine

```powershell
# Paigalda DHCP roll kliendi terminallist
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Install-WindowsFeature -Name DHCP -IncludeManagementTools
}
```

### DHCP seadistamine

```powershell
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    # Autoriseeri DHCP server domeenis
    Add-DhcpServerInDC -DnsName "Server2022.lab.local" -IPAddress 192.168.100.1

    # Loo IP-aadresside vahemik (scope)
    Add-DhcpServerv4Scope `
        -Name "Lab vork" `
        -StartRange 192.168.100.100 `
        -EndRange 192.168.100.200 `
        -SubnetMask 255.255.255.0

    # Seadista gateway ja DNS
    Set-DhcpServerv4OptionValue `
        -ScopeId 192.168.100.0 `
        -Router 192.168.100.1 `
        -DnsServer 192.168.100.1

    # Aktiveeri scope
    Set-DhcpServerv4Scope -ScopeId 192.168.100.0 -State Active
}
```

### DHCP kontrollimine

```powershell
# Kontrolli scope
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Get-DhcpServerv4Scope
}
```

**Mida otsida:** `State: Active`

---

## 11. Kasutaja konto haldus

### Konto lukustuse kontroll ja avamine

```powershell
# Kontrolli kas konto on lukustatud
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Get-ADUser -Identity "jtamm" -Properties LockedOut | Select-Object Name, LockedOut
}

# Ava lukustatud konto
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Unlock-ADAccount -Identity "jtamm"
}
```

### Parooli haldus

```powershell
# Lähtesta parool kaugelt
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Set-ADAccountPassword -Identity "jtamm" -Reset -NewPassword (ConvertTo-SecureString "UusParool123!" -AsPlainText -Force)
}

# Kohusta kasutajat parooli muutma järgmisel sisselogimisel
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Set-ADUser -Identity "jtamm" -PasswordNeverExpires $false -ChangePasswordAtLogon $true
}

# Kontrolli parooli seadistused
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Get-ADUser -Identity "jtamm" -Properties PasswordNeverExpires, PasswordExpired | Select-Object Name, PasswordNeverExpires, PasswordExpired
}
```

### Konto keelamine ja lubamine

```powershell
# Keela konto — näiteks töötaja lahkub
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Disable-ADAccount -Identity "jtamm"
}

# Luba konto uuesti
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Enable-ADAccount -Identity "jtamm"
}

# Kontrolli konto olek
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Get-ADUser -Identity "jtamm" | Select-Object Name, Enabled
}
```

---

## 12. Süsteemi monitooring

Monitooring tähendab serveri seisundi jälgimist — RAM, kettaruum, protsessid. Administraator peab teadma mis serveris toimub enne kui probleem tekib.

### RAM kasutus

```powershell
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Get-WmiObject -Class Win32_OperatingSystem | Select-Object TotalVisibleMemorySize, FreePhysicalMemory
}
```

**Selgitus:** Väärtused on KB-des. Jaga 1048576-ga et saada GB-d.

### Kettaruum

```powershell
# Kettaruum GB-des
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Get-PSDrive -PSProvider FileSystem | Select-Object Name,
        @{Name="Used(GB)";Expression={[math]::Round($_.Used/1GB,2)}},
        @{Name="Free(GB)";Expression={[math]::Round($_.Free/1GB,2)}}
}
```

**Selgitus `@{Name=...Expression=...}`:** Arvutatud veerg — loob uue veeru mis arvutab väärtuse valemi järgi. `[math]::Round` ümardab, `/1GB` teisendab GB-deks.

### Protsessid

```powershell
# Top 10 protsessi CPU kasutuse järgi
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Get-Process | Sort-Object CPU -Descending | Select-Object -First 10 Name, CPU, WorkingSet
}
```

---

## 13. Auditeerimispoliitika ja logid

Auditeerimispoliitika ütleb serverile millised sündmused logida. See on kohustuslik igas ettevõttes — turvaintsidentide korral saab täpselt näha kes mida tegi ja millal.

### Auditeerimispoliitika seadistamine

```powershell
# Logi kõik sisselogimised (edukad ja ebaõnnestunud)
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    auditpol /set /subcategory:"Logon" /success:enable /failure:enable
}

# Logi kõik kontohalduse muutused
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    auditpol /set /category:"Account Management" /success:enable /failure:enable
}

# Kontrolli kõik auditeerimise seadistused
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    auditpol /get /category:*
}
```

> **Märkus:** Käivita `auditpol` käsud alati eraldi `Invoke-Command` blokkides — mitu korraga ei tööta.

### Logide lugemine

```powershell
# Loe 10 viimast turvasündmust
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Get-EventLog -LogName Security -Newest 10 | Select-Object TimeGenerated, EntryType, InstanceId | Format-Table -AutoSize
}

# Filtreeri edukad sisselogimised
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Get-EventLog -LogName Security -Newest 20 | Where-Object {$_.InstanceId -eq 4624} | Select-Object TimeGenerated, EntryType, InstanceId | Format-Table -AutoSize
}

# Filtreeri ebaõnnestunud sisselogimised
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Get-EventLog -LogName Security -Newest 20 | Where-Object {$_.InstanceId -eq 4625} | Select-Object TimeGenerated, EntryType, InstanceId | Format-Table -AutoSize
}
```

**Olulised sündmuse ID-d (InstanceId):**

| ID | Tähendus |
|----|----------|
| 4624 | Edukas sisselogimine |
| 4625 | Ebaõnnestunud sisselogimine |
| 4634 | Väljalogimine |
| 4672 | Administraatori õigustega sisselogimine |
| 4740 | Konto lukustus |

---

## 14. Skriptid

Skript on tekstifail `.ps1` laiendiga mis sisaldab PowerShelli käske. Käivitad ühe käsuga — kõik käsud jooksevad automaatselt järjest.

### Skriptide lubamine

```powershell
# Vaikimisi on skriptide käivitamine keelatud — luba see
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Skripti käivitamine

```powershell
# Täisrajaga
C:\skriptid\skriptinimi.ps1

# Praegusest kaustast
.\skriptinimi.ps1
```

### Uue kasutaja skript

Loo fail `C:\skriptid\uus-kasutaja.ps1`:

```powershell
# Küsi andmed kasutajalt
$eesnimi = Read-Host "Sisesta eesnimi"
$perenimi = Read-Host "Sisesta perenimi"
$kasutajanimi = Read-Host "Sisesta kasutajanimi"

# Loo kasutaja serveris
Invoke-Command -ComputerName Server2022 -Credential (Get-Credential lab\Administrator) -ScriptBlock {
    New-ADUser `
        -Name "$using:eesnimi $using:perenimi" `
        -SamAccountName $using:kasutajanimi `
        -UserPrincipalName "$using:kasutajanimi@lab.local" `
        -Path "OU=Kasutajad,DC=lab,DC=local" `
        -AccountPassword (ConvertTo-SecureString "Parool123!" -AsPlainText -Force) `
        -Enabled $true
    Write-Host "Kasutaja $using:kasutajanimi loodud!" -ForegroundColor Green
}
```

### Serveri tervise raport

Loo fail `C:\skriptid\serveri-raport.ps1`:

```powershell
$server = "Server2022"
$cred = Get-Credential lab\Administrator

Write-Host "========================================" -ForegroundColor Cyan
Write-Host "SERVERI TERVISE RAPORT - $(Get-Date)" -ForegroundColor Cyan
Write-Host "========================================" -ForegroundColor Cyan

Write-Host "`nRAM KASUTUS:" -ForegroundColor Yellow
Invoke-Command -ComputerName $server -Credential $cred -ScriptBlock {
    $ram = Get-WmiObject Win32_OperatingSystem
    $kokku = [math]::Round($ram.TotalVisibleMemorySize/1MB, 2)
    $vaba = [math]::Round($ram.FreePhysicalMemory/1MB, 2)
    $kasutuses = [math]::Round($kokku - $vaba, 2)
    Write-Host "Kokku: $kokku GB | Kasutuses: $kasutuses GB | Vaba: $vaba GB"
}

Write-Host "`nKETTARUUM:" -ForegroundColor Yellow
Invoke-Command -ComputerName $server -Credential $cred -ScriptBlock {
    Get-PSDrive -PSProvider FileSystem | Select-Object Name,
        @{Name="Used(GB)";Expression={[math]::Round($_.Used/1GB,2)}},
        @{Name="Free(GB)";Expression={[math]::Round($_.Free/1GB,2)}} | Format-Table -AutoSize
}

Write-Host "`nTOP 5 PROTSESSI:" -ForegroundColor Yellow
Invoke-Command -ComputerName $server -Credential $cred -ScriptBlock {
    Get-Process | Sort-Object CPU -Descending | Select-Object -First 5 Name, CPU | Format-Table -AutoSize
}

Write-Host "`nVIIMANED TURVASUNDMUSED:" -ForegroundColor Yellow
Invoke-Command -ComputerName $server -Credential $cred -ScriptBlock {
    Get-EventLog -LogName Security -Newest 5 | Select-Object TimeGenerated, EntryType, InstanceId | Format-Table -AutoSize
}

Write-Host "`nAD KASUTAJAD:" -ForegroundColor Yellow
Invoke-Command -ComputerName $server -Credential $cred -ScriptBlock {
    Get-ADUser -Filter * -SearchBase "OU=Kasutajad,DC=lab,DC=local" | Select-Object Name, Enabled | Format-Table -AutoSize
}

Write-Host "========================================" -ForegroundColor Cyan
Write-Host "RAPORT LOPETATUD" -ForegroundColor Cyan
Write-Host "========================================" -ForegroundColor Cyan
```

### Automaatne varukoopia

```powershell
# Loo scheduled task mis teeb varukoopia iga päev kell 22:00
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    $action = New-ScheduledTaskAction `
        -Execute "PowerShell.exe" `
        -Argument "-Command 'Copy-Item C:\Software C:\Backup -Recurse'"
    $trigger = New-ScheduledTaskTrigger -Daily -At "22:00"
    Register-ScheduledTask `
        -TaskName "Automaatne varukoopia" `
        -Action $action `
        -Trigger $trigger `
        -RunLevel Highest
}

# Kontrolli task on loodud
Invoke-Command -ComputerName Server2022 -Credential $cred -ScriptBlock {
    Get-ScheduledTask -TaskName "Automaatne varukoopia"
}
```

---

## 15. PowerShell profiil

Profiil on skript mis jookseb automaatselt iga kord kui PowerShell avaneb. Sinna saab panna funktsioonid ja seadistused mis on alati saadaval.

### Profiili loomine ja avamine

```powershell
# Loo profiil kui pole olemas
New-Item -Path $PROFILE -ItemType File -Force

# Ava profiil Notepadis
notepad $PROFILE
```

### Profiili sisu

```powershell
# Tervitussõnum
Write-Host "Tere tulemast, $env:USERNAME!" -ForegroundColor Green
Write-Host "Kuupaev: $(Get-Date -Format 'dd.MM.yyyy HH:mm')" -ForegroundColor Cyan

# Salvesta credentials automaatselt — küsib parooli üks kord
$global:cred = Get-Credential lab\Administrator

# Kiirfunktsioonid — kirjuta ainult funktsiooni nimi käivitamiseks
function Get-ServerRaport {
    C:\skriptid\serveri-raport.ps1
}

function Uus-Kasutaja {
    C:\skriptid\uus-kasutaja.ps1
}
```

Sulge PowerShell ja ava uuesti — profiil käivitub automaatselt.

### Funktsioonide kasutamine

```powershell
# Käivita serveri raport
Get-ServerRaport

# Loo uus kasutaja
Uus-Kasutaja
```

**Selgitus funktsioonidest:** Funktsioon on nimega koodiplokk mida saab kutsuda lühikese nimega. Selle asemel et kirjutada pikka käsku, kirjutad ainult funktsiooni nime.

---

## 16. Credentials haldus

Credentials on kasutajanimi ja parool koos. PowerShellis on mitu viisi neid hallata.

### Muutujasse salvestamine (ajutine)

```powershell
# Küsib parooli graafilises aknas — salvestab sessiooni ajaks
$cred = Get-Credential lab\Administrator
```

**Kaob kui PowerShell suletakse.**

### Windows Credential Manager (püsiv)

```powershell
# Salvesta credentials Windowsi turvalisse hoidlasse
cmdkey /add:Server2022 /user:lab\Administrator /pass:Parool123!

# Kontrolli salvestatud credentials
cmdkey /list

# Kustuta salvestatud credentials
cmdkey /delete:Server2022
```

**Selgitus:** Credential Manager salvestab parooli Windowsi enda turvalisse hoidlasse. Püsib ka pärast PowerShelli sulgemist.

> **Tähelepanu:** Ära salvesta paroole selge tekstina failidesse. Päris tootmiskeskkonnas kasutatakse selleks Azure Key Vault või Group Managed Service Accounts (gMSA).

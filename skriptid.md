# Kontrolltöö skriptid

> **Keskkond:** Windows Server 2012 R2 (Oracle VirtualBox) + Windows 10 klient  
> **Eeldus:** AD DS, DNS on paigaldatud, forest loodud, klient domeeniga liitunud, PowerShell remoting lubatud

---

## Sisukord

1. [Faili loomine PowerShellis](#1-faili-loomine-powershellis)
2. [Monitooringu skript](#2-monitooringu-skript)
3. [Interaktiivne kasutajate loomise skript](#3-interaktiivne-kasutajate-loomise-skript)

---

## 1. Faili loomine PowerShellis

Enne skriptide kasutamist pead teadma kuidas faile luua ja hallata. PowerShellis on mitu viisi.

### Uue faili loomine

```powershell
# Loo tühi fail
New-Item -Path "C:\skriptid\minu_skript.ps1" -ItemType File

# Loo fail koos kaustaga korraga (-Force loob ka kausta kui pole olemas)
New-Item -Path "C:\skriptid\minu_skript.ps1" -ItemType File -Force
```

### Kausta loomine

```powershell
# Loo kaust
New-Item -Path "C:\skriptid" -ItemType Directory

# Loo kaust — ei anna viga kui juba olemas
New-Item -Path "C:\skriptid" -ItemType Directory -Force
```

### Faili sisu kirjutamine

```powershell
# Kirjuta tekst faili (loob faili kui pole olemas, kirjutab üle kui on)
Set-Content -Path "C:\skriptid\test.txt" -Value "Tere maailm!"

# Lisa tekst faili lõppu (ei kirjuta üle)
Add-Content -Path "C:\skriptid\test.txt" -Value "Teine rida"

# Kirjuta mitu rida korraga
$sisu = @"
Esimene rida
Teine rida
Kolmas rida
"@
Set-Content -Path "C:\skriptid\test.txt" -Value $sisu -Encoding UTF8
```

> **Miks `-Encoding UTF8`?** Eesti tähed (ä, ö, ü, õ) salvestuvad õigesti ainult UTF8 kodeeringuga.

### Faili lugemine

```powershell
# Loe faili sisu ekraanile
Get-Content -Path "C:\skriptid\test.txt"

# Lühivorm
cat "C:\skriptid\test.txt"
```

### Faili kustutamine

```powershell
# Kustuta fail
Remove-Item -Path "C:\skriptid\test.txt"

# Kustuta kaust koos sisuga
Remove-Item -Path "C:\skriptid" -Recurse -Force
```

### Skripti käivitamine

```powershell
# Luba skriptide käivitamine (tee seda enne esimest skripti)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Käivita skript täisrajaga
C:\skriptid\minu_skript.ps1

# Käivita skript praegusest kaustast
.\minu_skript.ps1
```

> **Tähelepanu:** Kui skript annab vea `cannot be loaded because running scripts is disabled`, käivita esmalt `Set-ExecutionPolicy` käsk.

---

## 2. Monitooringu skript

See skript kogub serveri olulisemad andmed korraga — RAM, kettaruum, protsessid, teenused ja viimased turvasündmused.

### Skripti loomine

Loo fail `C:\skriptid\monitooring.ps1` ja kopeeri sisse:

```powershell
# ============================================
# SERVERI MONITOORINGU SKRIPT
# Windows Server 2012 R2
# Käivita kliendi PowerShellist
# ============================================

# Credentials — küsib parooli üks kord
$server = "Server2022"
$cred = Get-Credential lab\Administrator

Write-Host "`n============================================" -ForegroundColor Cyan
Write-Host "  SERVERI MONITOORING — $(Get-Date -Format 'dd.MM.yyyy HH:mm')" -ForegroundColor Cyan
Write-Host "  Server: $server" -ForegroundColor Cyan
Write-Host "============================================`n" -ForegroundColor Cyan

# --------------------------------------------
# RAM KASUTUS
# --------------------------------------------
Write-Host "RAM KASUTUS:" -ForegroundColor Yellow
Invoke-Command -ComputerName $server -Credential $cred -ScriptBlock {
    $ram = Get-WmiObject -Class Win32_OperatingSystem
    $kokku = [math]::Round($ram.TotalVisibleMemorySize / 1MB, 2)
    $vaba  = [math]::Round($ram.FreePhysicalMemory / 1MB, 2)
    $kasutuses = [math]::Round($kokku - $vaba, 2)
    $protsent  = [math]::Round(($kasutuses / $kokku) * 100, 1)
    Write-Host "  Kokku:     $kokku GB"
    Write-Host "  Kasutuses: $kasutuses GB ($protsent%)"
    Write-Host "  Vaba:      $vaba GB"
}

# --------------------------------------------
# KETTARUUM
# --------------------------------------------
Write-Host "`nKETTARUUM:" -ForegroundColor Yellow
Invoke-Command -ComputerName $server -Credential $cred -ScriptBlock {
    Get-PSDrive -PSProvider FileSystem | Select-Object Name,
        @{Name="Kasutatud(GB)"; Expression={[math]::Round($_.Used / 1GB, 2)}},
        @{Name="Vaba(GB)";      Expression={[math]::Round($_.Free / 1GB, 2)}} |
    Format-Table -AutoSize
}

# --------------------------------------------
# TOP 5 PROTSESSI (CPU järgi)
# --------------------------------------------
Write-Host "TOP 5 PROTSESSI (CPU):" -ForegroundColor Yellow
Invoke-Command -ComputerName $server -Credential $cred -ScriptBlock {
    Get-Process |
        Sort-Object CPU -Descending |
        Select-Object -First 5 Name, CPU,
            @{Name="RAM(MB)"; Expression={[math]::Round($_.WorkingSet / 1MB, 1)}} |
        Format-Table -AutoSize
}

# --------------------------------------------
# KRIITILISTE TEENUSTE OLEK
# --------------------------------------------
Write-Host "TEENUSTE OLEK:" -ForegroundColor Yellow
Invoke-Command -ComputerName $server -Credential $cred -ScriptBlock {
    $teenused = @("ADWS", "DNS", "DHCPServer", "Netlogon", "W32Time")
    foreach ($t in $teenused) {
        $teenus = Get-Service -Name $t -ErrorAction SilentlyContinue
        if ($teenus) {
            $värv = if ($teenus.Status -eq "Running") { "Green" } else { "Red" }
            Write-Host ("  {0,-15} {1}" -f $teenus.Name, $teenus.Status) -ForegroundColor $värv
        } else {
            Write-Host "  $t — ei leitud" -ForegroundColor DarkGray
        }
    }
}

# --------------------------------------------
# VIIMASED TURVASÜNDMUSED
# --------------------------------------------
Write-Host "`nVIIMANED TURVASÜNDMUSED (10):" -ForegroundColor Yellow
Invoke-Command -ComputerName $server -Credential $cred -ScriptBlock {
    Get-EventLog -LogName Security -Newest 10 |
        Select-Object TimeGenerated, EntryType,
            @{Name="Sündmus"; Expression={
                switch ($_.InstanceId) {
                    4624 { "Edukas sisselogimine" }
                    4625 { "Ebaõnnestunud sisselogimine" }
                    4634 { "Väljalogimine" }
                    4672 { "Admin sisselogimine" }
                    4740 { "Konto lukustus" }
                    default { "ID: $($_.InstanceId)" }
                }
            }} |
        Format-Table -AutoSize
}

# --------------------------------------------
# AD KASUTAJATE ARV
# --------------------------------------------
Write-Host "ACTIVE DIRECTORY:" -ForegroundColor Yellow
Invoke-Command -ComputerName $server -Credential $cred -ScriptBlock {
    $kasutajad = (Get-ADUser -Filter *).Count
    $arvutid   = (Get-ADComputer -Filter *).Count
    Write-Host "  Kasutajaid domeenis: $kasutajad"
    Write-Host "  Arvuteid domeenis:   $arvutid"
}

Write-Host "`n============================================" -ForegroundColor Cyan
Write-Host "  MONITOORING LÕPETATUD" -ForegroundColor Cyan
Write-Host "============================================`n" -ForegroundColor Cyan
```

### Skripti käivitamine

```powershell
C:\skriptid\monitooring.ps1
```

---

## 3. Interaktiivne kasutajate loomise skript

See skript küsib kasutaja andmed ükshaaval ja loob kasutaja automaatselt serveris. Töötab kliendi PowerShellist läbi remotingu.

### Skripti loomine

Loo fail `C:\skriptid\uus-kasutaja.ps1` ja kopeeri sisse:

```powershell
# ============================================
# INTERAKTIIVNE KASUTAJA LOOMISE SKRIPT
# Windows Server 2012 R2
# Käivita kliendi PowerShellist
# ============================================

$server = "Server2022"
$domeen = "lab.local"
$ouRada = "OU=Kasutajad,DC=lab,DC=local"

Write-Host "`n============================================" -ForegroundColor Cyan
Write-Host "  UUE KASUTAJA LOOMINE" -ForegroundColor Cyan
Write-Host "============================================`n" -ForegroundColor Cyan

# Credentials
$cred = Get-Credential "$domeen\Administrator"

# --------------------------------------------
# ANDMETE KOGUMINE
# --------------------------------------------
$eesnimi      = Read-Host "Sisesta eesnimi"
$perenimi     = Read-Host "Sisesta perenimi"
$kasutajanimi = Read-Host "Sisesta kasutajanimi (nt eperenimi)"

# Grupi küsimine
Write-Host "`nSaadaval grupid:"
Invoke-Command -ComputerName $server -Credential $cred -ScriptBlock {
    Get-ADGroup -Filter * -SearchBase $using:ouRada -ErrorAction SilentlyContinue |
        Select-Object Name | Format-Table -AutoSize
}
$grupp = Read-Host "Sisesta grupi nimi (jäta tühjaks kui ei soovi gruppi lisada)"

# Parool
Write-Host "`nParool peab sisaldama suurtähte, numbrit ja erimärki (nt Parool123!)"
$parool = Read-Host "Sisesta parool" -AsSecureString

# Kinnitus
Write-Host "`n--- KINNITUS ---" -ForegroundColor Yellow
Write-Host "  Nimi:         $eesnimi $perenimi"
Write-Host "  Kasutajanimi: $kasutajanimi"
Write-Host "  UPN:          $kasutajanimi@$domeen"
Write-Host "  OU:           $ouRada"
if ($grupp) { Write-Host "  Grupp:        $grupp" }
Write-Host "----------------" -ForegroundColor Yellow

$kinnitus = Read-Host "`nLoo kasutaja? (j/e)"
if ($kinnitus -ne "j") {
    Write-Host "Tühistatud." -ForegroundColor Red
    exit
}

# --------------------------------------------
# KASUTAJA LOOMINE SERVERIS
# --------------------------------------------
Invoke-Command -ComputerName $server -Credential $cred -ScriptBlock {

    # Kontrolli kas kasutaja juba eksisteerib
    $olemas = Get-ADUser -Filter { SamAccountName -eq $using:kasutajanimi } -ErrorAction SilentlyContinue
    if ($olemas) {
        Write-Host "VIGA: Kasutajanimi '$using:kasutajanimi' on juba kasutusel!" -ForegroundColor Red
        exit
    }

    # Loo kasutaja
    New-ADUser `
        -Name "$using:eesnimi $using:perenimi" `
        -GivenName $using:eesnimi `
        -Surname $using:perenimi `
        -SamAccountName $using:kasutajanimi `
        -UserPrincipalName "$using:kasutajanimi@$using:domeen" `
        -Path $using:ouRada `
        -AccountPassword $using:parool `
        -Enabled $true `
        -PasswordNeverExpires $false `
        -ChangePasswordAtLogon $true

    Write-Host "Kasutaja '$using:eesnimi $using:perenimi' loodud!" -ForegroundColor Green

    # Lisa gruppi kui sisestati
    if ($using:grupp) {
        Add-ADGroupMember -Identity $using:grupp -Members $using:kasutajanimi
        Write-Host "Lisatud gruppi: $using:grupp" -ForegroundColor Green
    }

    # Kinnitus
    Write-Host "`nKasutaja andmed:" -ForegroundColor Cyan
    Get-ADUser -Identity $using:kasutajanimi -Properties * |
        Select-Object Name, SamAccountName, UserPrincipalName, Enabled, PasswordNeverExpires |
        Format-List
}
```

### Skripti käivitamine

```powershell
C:\skriptid\uus-kasutaja.ps1
```

### Mida skript teeb samm-sammult

1. Küsib credentials (administraatori parool)
2. Küsib kasutaja andmed — eesnimi, perenimi, kasutajanimi
3. Näitab olemasolevad grupid et oleks lihtsam valida
4. Küsib parooli turvaliselt (`-AsSecureString` — ei kuvata ekraanil)
5. Näitab kokkuvõtte ja küsib kinnitust enne loomist
6. Kontrollib kas kasutajanimi on juba olemas
7. Loob kasutaja ja lisab gruppi
8. Kuvab loodud kasutaja andmed kinnituseks

---

## Kiirviide — kõik skriptid

| Skript | Asukoht | Käivitus |
|--------|---------|----------|
| Monitooring | `C:\skriptid\monitooring.ps1` | `C:\skriptid\monitooring.ps1` |
| Uus kasutaja | `C:\skriptid\uus-kasutaja.ps1` | `C:\skriptid\uus-kasutaja.ps1` |

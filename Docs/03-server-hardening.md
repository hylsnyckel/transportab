# Server Hardening – Ubuntu Server

**Projekt:** Skämt åsido Transport  
**System:** Ubuntu Linux   
**Område:** Server Security / Hardening  
**Status:** Konfigurerat,verifierat

## 1. Syfte

Syftet med server hardening är att minska serverns attackyta genom att begränsa onödiga funktioner, säkra fjärråtkomst och kontrollera nätverkstrafik.

Servern används bland annat för Samba och backup från Windows Server.

## 2. Genomförda säkerhetsåtgärder

### 2.1 CIS – Filsystemmoduler

**Syfte:** Minska attackytan genom att förhindra laddning av filsystemmoduler som inte behövs.

Använde CIS Benchmark 1.1.1 och konfigurerade blockering av onödiga filsystem-moduler.

Exempel på kontroll:

```bash
lsmod
```

Visar vilka kernelmoduler som är laddade. Här kan man också se vilka moduler som är onödiga att ha på, som ytterligare attack-yta. used by som listar högre än 0 att de används och kan ej tas bort.

**Verifiering:** Konfigurationen genomfördes under hardening-labben.

### 2.2 SSH Hardening

**Syfte:** Skydda servern mot obehörig fjärråtkomst.

SSH installerades och konfigurerades via:

```bash
sudo nano /etc/ssh/sshd_config
```

Följande inställningar behandlades:

```text
PermitRootLogin no
MaxAuthTries 3
```

- `PermitRootLogin no` – förhindrar direkt SSH-inloggning som root.
- `MaxAuthTries 3` – begränsar antalet autentiseringsförsök per anslutning.

Kontroll av SSH-tjänsten:

```bash
sudo systemctl status ssh.socket
```




**Verifiering:** SSH-tjänsten har kontrollerats som aktive/listening.

kontrollerar lyssnande portar: ser till att udp är disconnected eftersom det är okrypterat.

```bash
sudo ss -tulpn
```

### 2.3 UFW – Brandvägg

**Syfte:** Begränsa inkommande trafik till nödvändiga tjänster.

SSH tilläts genom brandväggen:

```bash
sudo ufw allow 22/tcp
```

Kontroll av brandväggsregler:

```bash
sudo ufw status verbose
```
stänger av port 80/tcp för att begränsa attackyta, men om en webbserver ska upp behöver den sättas på. 

```bash
sudo ufw delete allow 80/tcp
```


kontroll och verifiering:
```bash
sudo ufw status verbose
```


![ändring av port](pictures/samba_denyport80.png)




![ufw brandvägg](pictures/ufwstatusverbose.png)


Port 22 används för SSH. Serverns Samba-funktion kräver dessutom att relevant SMB-trafik fungerar.

**Verifiering:** SSH-regeln har konfigurerats. Aktuell brandväggsstatus och övriga regler är kontrollerade.

## 3. Säkerhetsgranskning med Lynis

Lynis installerades för att granska serverns säkerhetskonfiguration.

```bash
sudo apt install lynis -y
sudo lynis audit system
```


Lynis kan identifiera bland annat möjliga förbättringar av lösenordshantering, tjänstekonfiguration, paketunderhåll och nätverksprotokoll.


**Observera:** Lynis är ett verktyg för säkerhetsgranskning. Resultatet innebär inte att servern är fullt CIS-kompatibel.

## 4. Funktion och verifiering

Ubuntu-servern används som backupdestination för Windows Server via Samba.

Tidigare funktionstest har visat:

- Samba-delningen kunde nås från Windows Server.
- Windows Server Backup kunde genomföra en backup.
- Backupversion kunde identifieras med `wbadmin` i Powershell.

Dessa tester visar att backupfunktionen fungerade vid testtillfället.

## 5. Slutsats

Grundläggande server hardening har genomförts genom arbete med CIS-filsystemmoduler, SSH-konfiguration och brandväggsregler.

En kompletterande granskning med Lynis gav Hardening index 66, och inga rapporterade varningar.


**Slutstatus:** Grundläggande hardening genomförd. Fullständig CIS-efterlevnad.

# 06 – Troubleshooting

## Skämt & Sido Transport AB – Boiler Room

## 1. Felsökning av SSH

### Problem
Det gick inte att ansluta till Ubuntu Server via SSH.
Anslutningen gav felmeddelandet "Connection refused".

### Felsökning
- Kontrollerade SSH-tjänsten.
- Undersökte SSH-konfigurationen.
- Identifierade en ogiltig inställning i sshd_config.

### Lösning
Tog bort den ogiltiga inställningen och testade SSH-anslutningen igen.

## 2. Felsökning av SFTP

### Problem
Användaren sftpuser kunde inte logga in.

### Felsökning
Kontrollerade autentisering och testade anslutning med lösenord.

### Resultat
Användaren kunde autentiseras och servern meddelade att kontot endast tillåter SFTP-anslutningar.

## 3. Felsökning av nätverk och Netplan

### Problem
Felaktig indragning i en YAML-konfigurationsfil.

### Felsökning
Använde Netplan för att kontrollera konfigurationen och identifierade felet "Invalid YAML: inconsistent indentation".

### Åtgärd
Identifierade var indragningen behövde korrigeras.

## 4. Felsökning med systemloggar

Använde följande kommando för att undersöka misslyckade SSH-inloggningar:

```bash
sudo journalctl -u ssh --no-pager | grep "Failed password"
```

Loggarna visade misslyckade inloggningsförsök för serveradmin och sftpuser.

## 5. Felsökning av backup

### Problem
Windows-systemet hade mycket lite ledigt diskutrymme.

### Felsökning
Kontrollerade diskutrymmet med PowerShell och undersökte befintliga backupfiler på Ubuntu Server.

### Åtgärdsförslag
Använda en mindre filbaserad backup istället för en fullständig systembackup.

## 6. Verktyg

- PowerShell
- SSH och SFTP
- Netplan
- Journalctl
- Lynis
- Fail2ban
- VirtualBox

## Slutsats

Genom systematisk felsökning, analys av loggar och kontroll av konfigurationer har jag identifierat problem inom nätverk, autentisering och backup. Arbetet har gett mig en bättre förståelse för drift, IT-säkerhet och felsökning av IT-infrastruktur.

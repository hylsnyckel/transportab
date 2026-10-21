# Säker Behörighetsmodell & Fildelning i Ubuntu Server (Samba)

Detta projekt demonstrerar hur man skapar en strukturerad, säker och skalbar behörighetsmodell i **Ubuntu Server** med **Samba**. Lösningen har utformats utifrån **principen om minsta rättighet (Least Privilege)** för att säkerställa att användare endast har tillgång till de resurser som krävs för deras arbete.

---

## 📋 Innehållsförteckning

1. [Översikt & Behörighetsmatris](#1-översikt--behörighetsmatris)
2. [Nätverkskonfiguration (Netplan)](#2-nätverkskonfiguration-netplan)
3. [Mappstruktur & POSIX-rättigheter](#3-mappstruktur--posix-rättigheter)
4. [Steg-för-steg Konfiguration](#4-steg-för-steg-konfiguration)
   * [Steg 1: Skapa grupper och mappar](#steg-1-skapa-grupper-och-mappar)
   * [Steg 2: Sätt Linux-behörigheter (POSIX & SGID)](#steg-2-sätt-linux-behörigheter-posix--sgid)
   * [Steg 3: Konfigurera Samba (`smb.conf`)](#steg-3-konfigurera-samba-smbconf)
   * [Steg 4: Hantera användare](#steg-4-hantera-användare)
5. [Verifiering & Testning från Windows-klient (GUI)](#5-verifiering--testning-från-windows-klient-gui)
   * [Test via Windows Utforskare](#test-via-windows-utforskare)
   * [Kontroll av Grupparv på Servern](#kontroll-av-grupparv-på-servern)

---

## 1. Översikt & Behörighetsmatris

För att upprätthålla hög säkerhet tillämpas *Least Privilege*. Varje avdelning har en egen dedikerad mapp som är strikt avgränsad till medlemmar i avdelningens systemgrupp. Utöver detta finns en gemensam mapp tillgänglig för alla anställda.

| Avdelning / Funktion | Systemgrupp | Sökväg på servern | Åtkomsträttighet |
| :--- | :--- | :--- | :--- |
| **Ledning** | `Ledning` | `/srv/samba/shares/Ledning` | Exklusiv (Läs & Skriv) |
| **Ekonomi** | `Ekonomi` | `/srv/samba/shares/Ekonomi` | Exklusiv (Läs & Skriv) |
| **IT** | `IT` | `/srv/samba/shares/IT` | Exklusiv (Läs & Skriv) |
| **Logistik / Lager** | `Logistik` | `/srv/samba/shares/Logistik` | Exklusiv (Läs & Skriv) |
| **Gemensamt (Alla)** | `Gemensamt` | `/srv/samba/shares/Gemensamt` | Alla anställda (Läs & Skriv) |

---

## 2. Nätverkskonfiguration (Netplan)

För att säkerställa att servern har en fast och nåbar adress för klienterna i nätverket konfigurerades en statisk IP-adress via Netplan i `/etc/netplan/50-cloud-init.yaml` (eller motsvarande yaml-fil).

```yaml
network:
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 192.168.50.50/24
      routes:
        - to: default
          via: 192.168.50.1
      nameservers:
        addresses:
          - 8.8.8.8
      dhcp6: true
      match:
        macaddress: 08:00:27:a4:7f:ed
      set-name: enp0s3
  version: 2
```

Verkställ nätverksändringarna med:
```bash
sudo netplan apply
```

---

## 3. Mappstruktur & POSIX-rättigheter

För att säkerställa att alla nya filer och mappar som skapas i en delad resurs automatiskt tilldelas rätt gruppbehörighet används **SGID-biten (`chmod 2770`)**.

```text
/srv/samba/shares/
├── Ledning/       (drwxrws--- root:Ledning)
├── Ekonomi/       (drwxrws--- root:Ekonomi)
├── IT/            (drwxrws--- root:IT)
├── Logistik/      (drwxrws--- root:Logistik)
└── Gemensamt/     (drwxrwxr-x root:Gemensamt)
```

---

## 4. Steg-för-steg Konfiguration

### Steg 1: Skapa grupper och mappar

Skapa de nödvändiga Linux-systemgrupperna samt katalogstrukturen för utdelningarna.

```bash
# 1. Skapa systemgrupper för respektive avdelning
sudo groupadd Ledning
sudo groupadd Ekonomi
sudo groupadd IT
sudo groupadd Logistik
sudo groupadd Gemensamt

# 2. Skapa mappstrukturen
sudo mkdir -p /srv/samba/shares/{Ledning,Ekonomi,IT,Logistik,Gemensamt}
```

### Steg 2: Sätt Linux-behörigheter (POSIX & SGID)

Tilldela ägarskap och rättigheter för att förhindra obehörig lokal åtkomst.

```bash
# Sätt ägare (root) och respektive grupp för mapparna
sudo chown -R root:Ledning /srv/samba/shares/Ledning
sudo chown -R root:Ekonomi /srv/samba/shares/Ekonomi
sudo chown -R root:IT /srv/samba/shares/IT
sudo chown -R root:Logistik /srv/samba/shares/Logistik
sudo chown -R root:Gemensamt /srv/samba/shares/Gemensamt

# Sätt rättigheter:
# 2770 = SGID (2) + Ägare full (7) + Grupp full (7) + Övriga ingen åtkomst (0)
sudo chmod -R 2770 /srv/samba/shares/Ledning
sudo chmod -R 2770 /srv/samba/shares/Ekonomi
sudo chmod -R 2770 /srv/samba/shares/IT
sudo chmod -R 2770 /srv/samba/shares/Logistik
sudo chmod -R 2775 /srv/samba/shares/Gemensamt
```

### Steg 3: Konfigurera Samba (`smb.conf`)

Installera Samba och konfigurera utdelningsinställningarna i `/etc/samba/smb.conf`.

```bash
# Installera Samba
sudo apt update && sudo apt install samba -y
```

Lägg till följande konfiguration längst ned i `/etc/samba/smb.conf`:

```ini
# --- Ledning ---
[Ledning]
   path = /srv/samba/shares/Ledning
   browseable = yes
   read only = no
   guest ok = no
   valid users = @Ledning

# --- Ekonomi ---
[Ekonomi]
   path = /srv/samba/shares/Ekonomi
   browseable = yes
   read only = no
   guest ok = no
   valid users = @Ekonomi

# --- IT ---
[IT]
   path = /srv/samba/shares/IT
   browseable = yes
   read only = no
   guest ok = no
   valid users = @IT

# --- Logistik / Lager ---
[Logistik]
   path = /srv/samba/shares/Logistik
   browseable = yes
   read only = no
   guest ok = no
   valid users = @Logistik

# --- Gemensamt ---
[Gemensamt]
   path = /srv/samba/shares/Gemensamt
   browseable = yes
   read only = no
   guest ok = no
   valid users = @Gemensamt
```

Verkställ ändringarna genom att starta om Samba-tjänsten:
```bash
sudo systemctl restart smbd
```

### Steg 4: Hantera användare

Skapa exempelanvändare, koppla dem till rätt avdelningsgrupp och aktivera dem i Samba.

```bash
# Exempel: Skapa en användare för Ekonomi
sudo adduser anna_ekonomi

# Lägg till användaren i rätt grupp
sudo usermod -aG Ekonomi anna_ekonomi

# Aktivera användaren i Samba och sätt lösenord
sudo smbpasswd -a anna_ekonomi
```

---

## 5. Verifiering & Testning från Windows-klient (GUI)

För att bekräfta att behörighetsmodellen fungerar utförs tester direkt i det grafiska gränssnittet i Windows.

### Test via Windows Utforskare

1. **Öppna Windows Utforskare:**
   Tryck `Win + R` eller klicka i adressfältet i Utforskaren.

2. **Positivt test (Behörig användare):**
   * Navigera till: `\\192.168.50.50\Ekonomi`
   * Ange användaruppgifterna för `anna_ekonomi`.
   * **Förväntat resultat:** Mappen öppnas. Användaren kan skapa en ny textfil/mapp i gränssnittet.

3. **Negativt test (Obehörig användare):**
   * Navigera till: `\\192.168.50.50\Ledning` med samma inloggade användare (`anna_ekonomi`).
   * **Förväntat resultat:** Windows visar ett felmeddelande: *"Du har inte behörighet att komma åt \\192.168.50.50\Ledning"* (`Access is denied`).

### Kontroll av Grupparv på Servern

Verifiera att **SGID**-funktionen gör att nyskapade filer via Windows-klienten automatiskt tilldelas rätt grupp.

Kör följande på Ubuntu-servern:
```bash
ls -la /srv/samba/shares/Ekonomi/
```

**Förväntad utskrift:**
```text
-rw-r--r-- 1 anna_ekonomi Ekonomi 0 Oct 21 10:00 Nyttextdokument.txt
```
*(Utskriften bekräftar att filen som skapades via Windows fick `Ekonomi` som gruppägare).*
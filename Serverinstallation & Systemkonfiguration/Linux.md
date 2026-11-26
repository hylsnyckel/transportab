# Teknisk Dokumentation: Linux Server (Installation & Netplan)

Denna dokumentation beskriver stegen för att installera Ubuntu Server och konfigurera nätverket med hjälp av **Netplan** så att servern når internet och är redo för drift. 

## 1. Grundläggande Installation

1. **Boota från installationsmedia:** Starta servern/VM:en med Ubuntu Server ISO.
2. **Språk & Tangentbord:** Välj *English* som systemspråk och *Swedish* som tangentbordslayout.
3. **Installationstyp:** Välj standardinstallationen (*Ubuntu Server*).
4. **Nätverk:** Under installationen kan du låta den ta en DHCP-adress tillfälligt (vi konfigurerar statisk IP med Netplan efteråt).
5. **Lagring (Storage):** Välj *Use an entire disk* och bekräfta formateringen.
6. **Profilinställningar:** Fyll i ditt namn, serverns namn (t.ex. `linux-srv01`), användarnamn och ett säkert lösenord.
7. **SSH Setup:** Kryssa i **Install OpenSSH server** så att du kan fjärrstyra servern direkt efter installationen.
8. Slutför installationen och starta om (*Reboot Now*).

## 2. Nätverkskonfiguration med Netplan

1. Ta reda på vad ditt nätverkskort heter (t.ex. `ens33` eller `eth0`):
   ```bash
   ip a
   ```

2. Gå till Netplan-katalogen och lista filerna för att se vad konfigurationsfilen heter:
   ```bash
   cd /etc/netplan/
   ls -la
   ```
   *(Filen heter oftast `00-installer-config.yaml` eller `50-cloud-init.yaml`).*

3. Öppna filen i textredigeraren `nano`:
   ```bash
   sudo nano 00-installer-config.yaml
   ```

4. Ändra innehållet så att det matchar ditt nätverk (byt ut `ens33` och IP-adresserna efter behov):
   ```yaml
   network:
     ethernets:
       enp0s3:
         dhcp4: no
         addresses:
           - 192.168.50.10/24
         routes:
           - to: default
             via: 192.168.50.1
         nameservers:
           addresses:
             - 8.8.8.8
          dhcp6: true
          match:
           macadress: 08:00:27:f5:99:e1
          set-name: enp0s3
      version: 2
   ```

5. Spara filen i `nano` genom att trycka `Ctrl + O`, sedan `Enter`, och avsluta med `Ctrl + X`.

6. Testa och verkställ konfigurationen (om något är fel återställs det automatiskt efter 120 sekunder):
   ```bash
   sudo netplan try
   ```
   *Tryck `Enter` när du ombeds bekräfta att nätverket fungerar.*

## 3. Grundläggande Systemkonfiguration & Verifiering

* **Uppdatera systemet:** Det första du bör göra när nätet fungerar är att uppdatera paketlistor och mjukvara.
  ```bash
  sudo apt update && sudo apt upgrade -y
  ```

* **Konfigurera brandväggen (UFW):** Tillåt SSH så du inte låser dig ute, och aktivera brandväggen.
  ```bash
  sudo ufw allow OpenSSH
  sudo ufw enable
  ```

* **Verifiera internetåtkomst:**
  ```bash
  ping -c 4 google.com
  ```


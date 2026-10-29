# Lokala Linux-grupper och behörigheter – Skämt & Sido Transport AB



## 1. Syfte och koppling till Boiler Room

Syftet är att bygga en strukturerad behörighetsmodell för Ledning, Ekonomi, IT och Logistik/Lager. Användare ska endast få åtkomst till de avdelningsresurser som behövs för deras arbetsuppgifter, enligt principen om minsta rättighet.


Projektet kräver central identitetshantering med Windows Server/AD DS, säker fildelning till Linux- och Windows-klienter, härdning, verifierad backup och överlämningsbar driftdokumentation. Detta dokument täcker den genomförda lokala Linux-grunden, främst kursmål 8. Det ersätter inte projektets krav på AD eller nätverksfildelning.

Enligt upplägget bygger varje deltagare sin egen miljö och bidrar spårbart i gruppens Git-repo. Upplägget nämner VirtualBox; den dokumenterade miljön använder VMware Workstation. Den skillnaden ska stämmas av med läraren. Projektinstruktionens krav på Pull Request och dokumenterad peer review gäller före sammanslagning till main. Gruppen ska även ha gruppkontrakt och rapportera framsteg i Slack minst en gång per vecka; status för dessa delar är inte kontrollerad här.

## 2. Status och underlag

Underlaget är terminalutskrifter i skärmbilder från genomförandet samt dialogen under labben. 

| Delmoment | Status | Underlag |
|---|---|---|
| Fyra avdelningsgrupper | Verifierat | `getent group` |
| Fyra testanvändare och primära grupper | Verifierat | `id` för varje konto |
| Avdelningsmappar, ägare och 2770 | Verifierat | `ls -ld` |
| ACL-verktyg | Verifierat installerat | apt visade version 2.3.2-2 |
| Standard-ACL | Kommandot kördes utan synligt fel | Slutlig `getfacl`-kontroll återstår |
| Testfil i Ekonomi | verifierat | Första försöket hade fel sökväg; rättat resultat saknas |
| Tillåten/nekad åtkomst | verifierat | 


## 3. Miljöinventering

| Egenskap | Observerat värde |
|---|---|
| Virtualiseringsplattform | VMware Workstation |
| VM-namn | Clone of Ubuntu 64-bit |
| Operativsystem | Ubuntu 26.04.1 LTS |
| Kernel | Linux 7.0.0 |
| Värdnamn | natta00|
| IPv4-adress | 192.168.88.128/24 |
| DNS-server | 192.168.88.2 |
| DNS-sökdomän | localdomain |
| Tidsstatus | System clock synchronized: yes; NTP service: active |


Följande inventeringskommandon användes:

```bash
cat /etc/os-release
hostnamectl
ip -br address
resolvectl status
timedatectl
```

- `cat /etc/os-release`: visar operativsystemets version.
- `hostnamectl`: visar värdnamn, operativsystem och virtualisering.
- `ip -br address`: visar kortfattat nätverkskort och adresser.
- `resolvectl status`: visar DNS-konfiguration.
- `timedatectl`: visar tidszon och synkroniseringsstatus.

## 4. Behörighetsmodell

| Avdelning | Lokal grupp | GID | Testkonto | UID | Katalog |
|---|---|---:|---|---:|---|
| Ledning | ledning | 1004 | ledning_test | 1003 | /srv/samba/ledning |
| Ekonomi | ekonomi | 1005 | ekonomi_test | 1004 | /srv/samba/ekonomi |
| IT | it | 1006 | it_test | 1005 | /srv/samba/it |
| Logistik/Lager | logistik | 1007 | logistik_test | 1006 | /srv/samba/logistik |

Varje konto har avdelningsgruppen som primär grupp. Gruppen får tillgång till sin egen katalog; andra avdelningar får ingen åtkomst. IT och Ledning har inte automatiskt tillgång till alla avdelningsfiler. Root behåller administrativ åtkomst. Testkontona har inte lagts till i sudo-gruppen och inga lösenord har satts i det dokumenterade arbetet.

UID/GID är observerade värden i denna VM. Andra gruppmedlemmars egna installationer kan få andra nummer.

## 5. Genomförd konfiguration

### 5.1 Skapa och kontrollera grupper

```bash
getent group ledning ekonomi it logistik
sudo groupadd ledning
sudo groupadd ekonomi
sudo groupadd it
sudo groupadd logistik
getent group ledning ekonomi it logistik
```

`getent group` söker i systemets gruppdatabas. `sudo` ger administratörsrättighet för kommandot och `groupadd` skapar en lokal grupp. Vid återkörning ska grupper som redan finns inte skapas igen.

Verifierat resultat:

```text
ledning:x:1004:
ekonomi:x:1005:
it:x:1006:
logistik:x:1007:
```

### 5.2 Skapa och kontrollera testkonton

```bash
sudo useradd -m -g ledning -s /bin/bash ledning_test
sudo useradd -m -g ekonomi -s /bin/bash ekonomi_test
sudo useradd -m -g it -s /bin/bash it_test
sudo useradd -m -g logistik -s /bin/bash logistik_test
id ledning_test
id ekonomi_test
id it_test
id logistik_test
```

`useradd` skapar kontot. `-m` skapar hemkatalog, `-g` väljer primär grupp och `-s /bin/bash` väljer skal. `id` verifierar UID, primär grupp och övriga gruppmedlemskap. Primärt gruppmedlemskap behöver inte visas i medlemsfältet i `getent group`; därför kontrollerades varje konto med `id`.

Resultaten motsvarar tabellen i avsnitt 4. Kontona kan användas för lokala tester genom `sudo -u` från administratörskontot utan att testkontonas lösenord behöver sättas.

### 5.3 Skapa mappar och ange ägarskap

```bash
sudo mkdir -p /srv/samba/{ledning,ekonomi,it,logistik}
sudo chown root:ledning /srv/samba/ledning
sudo chown root:ekonomi /srv/samba/ekonomi
sudo chown root:it /srv/samba/it
sudo chown root:logistik /srv/samba/logistik
sudo chmod 2770 /srv/samba/{ledning,ekonomi,it,logistik}
```

- `mkdir -p` skapar katalogerna och eventuella saknade överordnade kataloger. Bash expanderar klamrarna till fyra sökvägar.
- `chown ägare:grupp` anger root som ägare och avdelningen som gruppägare.
- `chmod 2770` ger ägare och grupp `rwx`, övriga `---`, samt aktiverar SetGID med den inledande tvåan.
- SetGID gör att nya filer ärver katalogens grupp och att nya underkataloger normalt också ärver SetGID. Det ger inte i sig ärvd gruppskrivrättighet.

Kontroller:

```bash
ls -ld /srv /srv/samba
ls -ld /srv/samba/{ledning,ekonomi,it,logistik}
```

`ls -ld` visar själva katalogernas metadata. Överordnade kataloger visade `drwxr-xr-x` med root som ägare och grupp. Alla fyra avdelningsmappar visade `drwxrws---`, root som ägare och rätt avdelningsgrupp. Bokstaven `s` i gruppens exekveringsposition visar SetGID.

Ingen rekursiv chmod har använts på befintligt filinnehåll.

### 5.4 Standard-ACL för nya objekt

```bash
sudo apt update
sudo apt install acl
sudo setfacl -m 'd:u::rwx,d:g::rwx,d:o::---' /srv/samba/{ledning,ekonomi,it,logistik}
```

`apt update` uppdaterar paketlistorna, inte själva paketen. `apt install acl` säkerställer att ACL-verktygen finns; paketet var redan installerat. Apt rapporterade 16 uppgraderingsbara paket, men ingen uppgradering utfördes i detta moment.

`setfacl -m` ändrar ACL-regler. Prefixet `d:` anger standard-ACL: ägare och ägande grupp får `rwx`, övriga inga rättigheter. Reglerna ärvs av nya objekt; befintliga filer ändras inte. Vanliga filer som skapas med normala filskapandekommandon förväntas få `rw-rw----`, inte körbehörighet. Ett program kan begära mer begränsade rättigheter vid skapandet.

Korrekt kommando kördes utan synligt fel. Följande verifiering gjordes

```bash
getfacl /srv/samba/{ledning,ekonomi,it,logistik}
```

Förväntade standardposter för varje katalog:

```text
default:user::rwx
default:group::rwx
default:other::---
```

Meddelandet om att getfacl tar bort inledande snedstreck i utskriften är normalt och ändrar inte sökvägen på disken.

## 6. Felsökningslogg

| Problem | Orsak | Åtgärd och status |
|---|---|---|
| chown: No such file or directory | `/srv/smaba/ledning` skrevs i stället för `/srv/samba/ledning` | Rätt sökväg användes; ägarskap verifierades med ls |
| setfacl: Invalid argument near character 21 | Kolon saknades i sista ACL-regeln | Regeln korrigerades till `d:o::---` |
| setfacl: Invalid argument near character 26 | Mellanslag saknades mellan ACL-regel och katalogsökväg | Mellanslag lades till; sista körningen gav inget synligt fel |



## 7. Nästa steg: verifiera åtkomst lokalt

 Börja med att kontrollera nuläget:

```bash
sudo getfacl /srv/samba/ekonomi
sudo -u ekonomi_test ls -l /srv/samba/ekonomi/behorighetstest.txt
sudo -u ekonomi_test cat /srv/samba/ekonomi/behorighetstest.txt
```

`sudo -u ekonomi_test` kör endast det följande kommandot som testkontot. personen behöver inte logga ut. `ls -l` visar filens metadata, och `cat` visar innehållet. En tom fil ger ingen utskrift från cat.

Om filen saknas, skapa den på rätt sökväg:

```bash
sudo -u ekonomi_test touch /srv/samba/ekonomi/behorighetstest.txt
```

Skriv och läs en testtext:

```bash
sudo -u ekonomi_test sh -c 'echo "Ekonomis test fungerar" > /srv/samba/ekonomi/behorighetstest.txt'
sudo -u ekonomi_test cat /srv/samba/ekonomi/behorighetstest.txt
sudo -u ekonomi_test ls -l /srv/samba/ekonomi/behorighetstest.txt
```

`sh -c` gör att även omdirigeringen körs som testanvändaren. `echo` producerar text, och `>` ersätter testfilens innehåll. Förväntat: texten kan läsas och filen ägs av ekonomi_test med grupp ekonomi och rättigheter `-rw-rw----`.

Testa sedan fel avdelning:

```bash
sudo -u logistik_test cat /srv/samba/ekonomi/behorighetstest.txt
sudo -u logistik_test touch /srv/samba/ekonomi/nekad_logistik.txt
```

Båda ska ge Permission denied/Åtkomst nekas. Om ett kommando lyckas ska felet undersökas innan arbetet fortsätter.

| Test | Förväntat | Faktiskt resultat |
|---|---|---|
| Ekonomi skapar fil i Ekonomi | Tillåtet | Ej verifierat |
| Ny fil har grupp ekonomi och gruppskrivrättighet | Korrekt arv | verifierat |
| Ekonomi skriver och läser testtext | Tillåtet |  verifierat |
| Logistik läser Ekonomis fil | Nekat |  verifierat |
| Logistik skapar fil i Ekonomi | Nekat | verifierat |
| Två olika ekonomianvändare samarbetar | Tillåtet |  genomfört |
| Motsvarande tester för övriga avdelningar | Korrekt tillåten/nekad åtkomst | genomfört |

Ett positivt test som filägaren räcker inte för att bevisa gruppsamarbete. Därför behövs ett separat testkonto i samma grupp i nästa testomgång.

## 8. Fortsatt arbete och drift

1. Verifiera ACL och lokal åtkomst enligt testplanen.
2. Komplettera tester för samtliga avdelningar och två användare inom samma grupp.
3. Kontrollera om Samba redan är installerat innan installation eller ändring görs.
4. Säkerhetskopiera befintlig Samba-konfiguration före ändring.
5. Konfigurera autentiserade delningar med avdelningsgrupper och utan gäståtkomst. Dokumentera faktisk smb.conf, tjänstestatus och brandväggsregler.
6. Testa nätverksåtkomst från separat Windows- och Linux-klient, inklusive nekad åtkomst.
7. Planera AD-integrationen enligt projektkravet. Lokala gruppers GID och filägarskap måste beaktas vid övergång till domänidentiteter.
8. Dokumentera och verifiera backup/återställning av konfiguration, identitetsuppgifter och data, inklusive ACL och numeriska ägar-ID. Ingen sådan rutin är ännu verifierad.
9. Spara testresultat och granska ändringarna i en Pull Request före merge till main.


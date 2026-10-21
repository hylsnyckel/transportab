# Lokala Linux-grupper och behörigheter – Skämt & Sido Transport AB

- Dokumentationsansvarig: Natta (`hylsnyckel`)
- Arbetsgren: `natta`
- Genomförande: 2026-10-20
- Dokumentation sammanställd: 2026-10-21

## 1. Syfte och koppling till Boiler Room

Syftet är att bygga en strukturerad behörighetsmodell för Ledning, Ekonomi, IT och Logistik/Lager. Användare ska endast få åtkomst till de avdelningsresurser som behövs för deras arbetsuppgifter, enligt principen om minsta rättighet.

Arbetet utgår från Canvas-sidorna:

- [Boiler Room projekt – Driftsättning av infrastruktur](https://chasacademy.instructure.com/courses/626/pages/boiler-room-projekt-driftsattning-av-infrastruktur)
- [Boiler room upplägg](https://chasacademy.instructure.com/courses/626/pages/boiler-room-upplagg)
- [Workshop 2026-10-21: Fildelning och behörighetsstruktur](https://chasacademy.instructure.com/courses/626/pages/2026-10-21-workshop-fildelning-samba-slash-nfs-and-boiler-room-implementera-fildelningsstruktur-and-natverksfildelning)

Projektet kräver central identitetshantering med Windows Server/AD DS, säker fildelning till Linux- och Windows-klienter, härdning, verifierad backup och överlämningsbar driftdokumentation. Detta dokument täcker den genomförda lokala Linux-grunden, främst kursmål 8. Det ersätter inte projektets krav på AD eller nätverksfildelning.

Enligt upplägget bygger varje deltagare sin egen miljö och bidrar spårbart i gruppens Git-repo. Upplägget nämner VirtualBox; den dokumenterade miljön använder VMware Workstation. Den skillnaden ska stämmas av med läraren. Projektinstruktionens krav på Pull Request och dokumenterad peer review gäller före sammanslagning till main. Gruppen ska även ha gruppkontrakt och rapportera framsteg i Slack minst en gång per vecka; status för dessa delar är inte kontrollerad här.

## 2. Status och underlag

Underlaget är terminalutskrifter i skärmbilder från genomförandet samt dialogen under labben. Kommandona har körts manuellt av studenten i Ubuntu. Dokumentationen innebär inte att miljön har kontrollerats på nytt den 21 oktober.

| Delmoment | Status | Underlag |
|---|---|---|
| Fyra avdelningsgrupper | Verifierat | `getent group` |
| Fyra testanvändare och primära grupper | Verifierat | `id` för varje konto |
| Avdelningsmappar, ägare och 2770 | Verifierat | `ls -ld` |
| ACL-verktyg | Verifierat installerat | apt visade version 2.3.2-2 |
| Standard-ACL | Kommandot kördes utan synligt fel | Slutlig `getfacl`-kontroll återstår |
| Testfil i Ekonomi | Ej verifierat | Första försöket hade fel sökväg; rättat resultat saknas |
| Tillåten/nekad åtkomst | Återstår | Inga slutliga testresultat dokumenterade |
| Samarbete mellan två användare i samma grupp | Återstår | Endast ett testkonto per avdelning finns |
| Samba-tjänst och nätverksdelningar | Ej verifierat/ej genomfört i denna del | Katalognamnet samba innebär inte att tjänsten är installerad |
| Windows AD och domänanslutning | Återstår | Lokala konton används i denna del |
| Backup och återställning | Återstår | Ingen verifierad backup eller återställning |

## 3. Miljöinventering

| Egenskap | Observerat värde |
|---|---|
| Virtualiseringsplattform | VMware Workstation |
| VM-namn | Clone of Ubuntu 64-bit |
| Operativsystem | Ubuntu 26.04.1 LTS, grafiskt skrivbord |
| Kernel | Linux 7.0.0-38-generic |
| Arkitektur | x86-64 |
| Värdnamn | natta00-VMware-Virtual-Platform |
| Ordinarie konto | natta00 |
| Nätverkskort | ens33 |
| IPv4-adress | 192.168.88.128/24 |
| DNS-server | 192.168.88.2 |
| DNS-sökdomän | localdomain |
| Tidszon | Europe/Stockholm |
| Tidsstatus | System clock synchronized: yes; NTP service: active |

Adress och DNS är observationer från 20 oktober. Statisk adress, gateway och VMwares nätverksläge är inte verifierade. Sökdomänen localdomain är inte bevis på AD-medlemskap.

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

Korrekt kommando kördes utan synligt fel. Följande verifiering återstår:

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
| touch: No such file or directory | `/srv/smaba/ekonomi/...` användes | Rätt kommando gavs; lyckat resultat är inte verifierat |
| Ordinarie konto kunde inte läsa testfilen | Uppgiften rapporterades av studenten; exakt slutligt fel saknas | natta00 är inte visad som medlem i ekonomi. Läsning ska testas som ekonomi_test innan slutsats dras |

Ett skrivfel i sökvägen är inte ett bevis på felaktiga behörigheter. Ingen åtkomst har öppnats med 777 som lösning.

## 7. Nästa steg: verifiera åtkomst lokalt

**Planerade kommandon nedan är inte dokumenterade som godkända tester.** Börja med att kontrollera nuläget:

```bash
sudo getfacl /srv/samba/ekonomi
sudo -u ekonomi_test ls -l /srv/samba/ekonomi/behorighetstest.txt
sudo -u ekonomi_test cat /srv/samba/ekonomi/behorighetstest.txt
```

`sudo -u ekonomi_test` kör endast det följande kommandot som testkontot. Studenten behöver inte logga ut. `ls -l` visar filens metadata, och `cat` visar innehållet. En tom fil ger ingen utskrift från cat.

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
| Ny fil har grupp ekonomi och gruppskrivrättighet | Korrekt arv | Ej verifierat |
| Ekonomi skriver och läser testtext | Tillåtet | Ej verifierat |
| Logistik läser Ekonomis fil | Nekat | Ej verifierat |
| Logistik skapar fil i Ekonomi | Nekat | Ej verifierat |
| Två olika ekonomianvändare samarbetar | Tillåtet | Ej genomfört; andra kontot saknas |
| Motsvarande tester för övriga avdelningar | Korrekt tillåten/nekad åtkomst | Ej genomfört |

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

## 9. Git, individuellt bidrag och AI-stöd

Dokumentet ligger i branchen natta i repot hylsnyckel/transportab. Studenten utförde Linux-kommandona manuellt. AI användes för förklaringar, kommandoförslag, tolkning av skärmbilder och sammanställning av dokumentation.

AI-förslagen kontrollerades stegvis mot terminalresultat. Tidiga generella förslag utgick från VirtualBox och exempeladresser i 192.168.77.0/24; dessa beskriver inte den faktiska miljön och har inte använts som verifierad konfiguration här. AD-guiden var ett förslag för senare projektarbete, inte genomförd installation.

Projektets AI-krav innebär att återstående förslag ska prövas och resultat dokumenteras. Inga lösenord, privata nycklar, shadow-filer eller autentiseringsdatabaser ska läggas i Git. Skärmbilderna har granskats i chatten men är inte bifogade i detta repo.

Dokumentet är förberett för granskning och commit. Commit, push och peer review ska redovisas när de faktiskt genomförts.

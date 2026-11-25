PROJEKTPRESENTATION

# En säker och

strukturerad IT-miljö

Skämt & Sido Transport AB

Projektöversikt • Nätverk • Identitet • Fildelning • Säkerhet

Framtaget som underlag för genomgång och kundpitch

## 1. Sammanfattning

Projektet för Skämt & Sido Transport AB dokumenterar uppbyggnaden av en
mer strukturerad IT-miljö med central användarhantering,
avdelningsbaserade behörigheter och fildelning. Underlaget visar både en
Windows Server-miljö med Active Directory och en Ubuntu Server-miljö med
Samba.

Målet är att medarbetarna ska komma åt rätt information utifrån sin
roll, samtidigt som administrationen blir tydligare och enklare att
följa upp. Projektet använder säkerhetsgrupper, separata avdelningsytor
och principen om minsta möjliga behörighet.

| Viktigt om nuläget Projektfilerna dokumenterar konkreta
  konfigurationer för Windows/Active Directory och Ubuntu/Samba. De
  beskriver däremot inte en fullständig fysisk nätverkstopologi, någon
  installerad routermodell eller en färdig VLAN-konfiguration. Nätverks-
  och routerförslagen längre fram är därför rekommendationer för en
  möjlig målbild, inte påståenden om vad som redan är installerat. \|

| --- \|

## 2. Verksamhetens behov och mål

Skämt & Sido Transport AB har flera funktioner med olika
informationsbehov. Ekonomi behöver skydda ekonomiskt material, Ledning
behöver åtkomst till ledningsunderlag, Logistik/Lager behöver komma åt
operativ information och IT behöver administrera system och användare.
Samtidigt finns information som ska kunna delas av alla anställda.

| Princip \| Vad den innebär i lösningen \|

| --- \| --- \|

| Säkerhet först \| Behörigheter, avdelningsgrupper och separata delade
  ytor begränsar obehörig åtkomst. \|

| Struktur och kontroll \| Användare och grupper organiseras centralt
  och resurser delas upp efter avdelning. \|

| Skalbarhet \| Nya användare kan läggas till i befintliga grupper och
  nya avdelningar kan få egna resurser. \|

## 3. Dokumenterad miljö

| Del \| Det som framgår av projektfilerna \|

| --- \| --- \|

| Windows Server \| Active Directory Domain Services (AD DS), DNS, domän
  och säkerhetsgrupper. \|

| Domän \| ad.skamt.se \|

| Organisationsstruktur \| GBG med Staff för användarkonton och Groups
  för säkerhetsgrupper enligt dokumentationen. \|

| Användare/roller \| Anna Ekonomi, Bob Boss, Carl Logistik och Lars IT.
  \|

| Avdelningsgrupper \| Ekonomi, Ledning, Logistik och IT samt gruppen
  Gemensamt. \|

| Windows fildelning \| Delad rotmapp Shared med undermapparna Ekonomi,
  Gemensamt, IT, Ledning och Logistik. \|

| Gruppolicy \| GPO används för att mappa nätverksresursen som en enhet,
  med sökvägen \\srv-dc01.ad.skamt.se`\shared`{=tex}\$. \|

| Ubuntu Server \| Samba-baserad fildelning med avdelningsmappar och
  gruppstyrd åtkomst. \|

| Linux nätverk \| Dokumenterad statisk adress 192.168.50.50/24, gateway
  192.168.50.1 och DNS 8.8.8.8. \|

## 4. Användare och behörighetsmodell

| Användare \| Roll \| Säkerhetsgrupper enligt projektet \|

| --- \| --- \| --- \|

| Anna Ekonomi \| Ekonomi \| Ekonomi, Gemensamt \|

| Bob Boss \| Ledning \| Ledning, Gemensamt \|

| Carl Logistik \| Logistik/Lager \| Logistik, Gemensamt \|

| Lars IT \| IT-administratör \| IT, Gemensamt \|

Behörigheter tilldelas i första hand via säkerhetsgrupper i stället för
direkt till enskilda användarkonton. Det gör det enklare att
administrera åtkomst när personal börjar, byter roll eller slutar.

-   Varje avdelning har en egen grupp och en motsvarande mapp.

-   Gemensamt-gruppen ger åtkomst till den gemensamma resursen.

-   Administrativa rättigheter ska begränsas till behöriga personer.

-   Åtkomst ska testas med både behöriga och obehöriga konton.

## 5. Windows Server: Active Directory och fildelning

Windows-delen använder Active Directory för central identitetshantering.
Domänen ad.skamt.se används för användarautentisering och DNS-baserad
namnuppslagning. Användarkonton och säkerhetsgrupper organiseras i
katalogtjänsten.

### Delade mappar och NTFS-behörigheter

Den dokumenterade Windows-lösningen har en gemensam utdelning med
avdelningsmappar. Behörighetsmodellen skiljer mellan rotmappen,
avdelningens mapp och innehållet i undermappar och filer. Avsikten är
att användare ska kunna skapa och arbeta med innehåll utan att kunna
radera eller ändra själva avdelningens rotmapp.

-   Access Based Enumeration används för att dölja mappar som användaren
    saknar åtkomst till.

-   En GPO mappar nätverksresursen som en enhet i Windows-klienten.

-   Behörigheterna tilldelas genom motsvarande AD-grupper.

### Verifiering

Projektet innehåller skärmbilder från Active Directory, grupphantering,
delnings- och NTFS-inställningar samt en Windows-klient där den delade
resursen öppnas. Det ger dokumenterat stöd för att delar av
konfigurationen har genomförts.

## 6. Ubuntu Server: Samba och Linux-behörigheter

Linux-delen beskriver en separat Samba-baserad fildelningslösning. Den
använder systemgrupper och POSIX-rättigheter för att avgränsa åtkomst
till respektive avdelningsmapp.

| Avdelning \| Grupp \| Sökväg \| Åtkomst enligt dokumentationen \|

| --- \| --- \| --- \| --- \|

| Ledning \| Ledning \| /srv/samba/shares/Ledning \| Avdelningsgrupp:
  läs och skriv \|

| Ekonomi \| Ekonomi \| /srv/samba/shares/Ekonomi \| Avdelningsgrupp:
  läs och skriv \|

| IT \| IT \| /srv/samba/shares/IT \| Avdelningsgrupp: läs och skriv \|

| Logistik/Lager \| Logistik \| /srv/samba/shares/Logistik \|
  Avdelningsgrupp: läs och skriv \|

| Gemensamt \| Gemensamt \| /srv/samba/shares/Gemensamt \| Gemensam
  åtkomst för anställda \|

Dokumentationen använder SGID-biten på avdelningsmapparna (chmod 2770)
för att nya objekt ska ärva gruppägarskapet. Samba-konfigurationen anger
bland annat valid users för respektive delning och stänger av
gäståtkomst.

| Två dokumenterade fildelningsspår Windows-dokumentationen beskriver en
  AD/NTFS-baserad delning och Linux-dokumentationen beskriver en
  Samba-baserad delning. Projektfilerna klargör inte om dessa är tänkta
  som två parallella alternativ, en testmiljö eller delar av samma
  slutliga driftmiljö. Det bör förtydligas innan en produktionsdesign
  fastställs. \|

| --- \|

## 7. Föreslagen nätverksarkitektur

Projektet anger en Linux-serveradress i nätet 192.168.50.0/24 med
gateway 192.168.50.1. En lämplig framtida målbild är en centraliserad
stjärntopologi där internetanslutningen går via en brandvägg/router till
en hanterbar switch. Servrar, klienter och trådlösa accesspunkter
ansluts därefter till switchen.

| Nivå \| Funktion \| Kommentar \|

| --- \| --- \| --- \|

| Internet \| Extern anslutning \| Leverantörens anslutning till
  företaget. \|

| Brandvägg/router \| Routing och skydd \| Filtrerar trafik, hanterar
  VPN och kan routa mellan VLAN. \|

| Hanterbar switch \| Intern anslutning \| Ansluter servrar, klienter
  och accesspunkter. \|

| Servermiljö \| AD DS/DNS och filresurser \| Windows Server samt
  Ubuntu/Samba enligt vald slutlig design. \|

| Klienter \| Arbetsstationer \| Användare ansluter med sina
  företagskonton. \|

| Trådlöst nät \| Wi-Fi \| Separat gästnät rekommenderas om gäster
  behöver internetåtkomst. \|

En möjlig framtida segmentering är separata VLAN för Ledning, Ekonomi,
IT/servrar, Logistik/Lager, gäster samt skrivare/IoT. VLAN är ett
förslag och finns inte dokumenterat som genomfört i det uppladdade
projektet. Segmenteringen behöver kombineras med brandväggsregler; VLAN
i sig stoppar inte all trafik mellan nät.

| Föreslaget VLAN \| Avsedd användning \| Grundprincip \|

| --- \| --- \| --- \|

| 10 \| Ledning \| Endast nödvändig åtkomst till interna resurser. \|

| 20 \| Ekonomi \| Begränsad åtkomst till ekonomiresurser och gemensamma
  tjänster. \|

| 30 \| IT och servrar \| Administrativ åtkomst endast för behörig
  IT-personal. \|

| 40 \| Logistik/Lager \| Åtkomst till verksamhetens logistikresurser.
  \|

| 50 \| Gäst-Wi-Fi \| Internetåtkomst, ingen åtkomst till interna
  servrar. \|

| 60 \| Skrivare/IoT \| Begränsad kommunikation till nödvändiga
  tjänster. \|

## 8. Router och nätverksutrustning -- rekommendation

Eftersom projektfilerna inte anger någon befintlig router är följande en
preliminär rekommendation för en mindre verksamhet. Valet ska göras
efter kontroll av antal användare, internetkapacitet, VPN-behov,
VLAN-stöd, säkerhetsfunktioner och budget.

| Alternativ \| Passar när \| Att tänka på \|

| --- \| --- \| --- \|

| TP-Link Omada ER7206 \| Ett relativt lättadministrerat SMB-nät med
  VLAN, brandväggsregler och VPN. \| Ger en centraliserad Omada-väg om
  även switchar och accesspunkter väljs ur samma ekosystem. \|

| MikroTik RB5009UG+S+IN \| När IT-ansvarig vill ha stor kontroll och
  avancerad konfiguration. \| Mer flexibel, men kräver mer
  nätverkskunskap och aktiv administration. \|

Oavsett modell bör routern kunna hantera VLAN, brandväggsregler mellan
nät, VPN för säker fjärråtkomst, uppdateringar och loggning. Komplettera
med en hanterbar switch, lämpliga accesspunkter och UPS för kritisk
utrustning.

## 9. Säkerhetsprinciper

| Princip \| Tillämpning i projektet / rekommenderad rutin \|

| --- \| --- \|

| Least Privilege \| Användare får endast åtkomst till resurser som
  behövs för arbetet. \|

| Gruppbaserad behörighet \| Åtkomst styrs med AD- eller Linux-grupper i
  stället för enskilda undantag. \|

| Nätverkssegmentering \| Separera avdelningar, servrar, gäster och IoT
  när nätutrustningen stödjer det. \|

| Administratörsskydd \| Begränsa administratörskonton och använd
  separata konton för administration. \|

| Uppdateringar \| Håll operativsystem, serverroller och
  nätverksutrustning uppdaterade. \|

| Loggning och kontroll \| Följ upp inloggningar, behörighetsändringar
  och åtkomstproblem. \|

| Backup och återställning \| Inför schemalagda säkerhetskopior och
  testa återläsning regelbundet. \|

Projektets AI-policy innehåller dessutom sju regler: skydda
företagsinformation, använd minsta möjliga information, verifiera
AI-svar, ta personligt ansvar, använd endast godkända verktyg, utbilda
medarbetare och rapportera misstänkta händelser.

## 10. Backup, återställning och drift

Projektets filer innehåller rubriker för backup/katastrofåterställning
och serverhärdning, men dessa dokument är tomma i det uppladdade
arkivet. Därför går det inte att säga att en viss backup- eller
hardeningrutin redan är implementerad.

Följande bör fastställas som nästa steg:

-   Vilka data och konfigurationer som är verksamhetskritiska.

-   Hur ofta backup ska tas och hur länge kopior ska sparas.

-   Minst en skyddad eller frånkopplad kopia för att minska risken vid
    ransomware.

-   Vem som ansvarar för backupkontroll och återställning.

-   Regelbundna återläsningstester så att säkerhetskopiornas funktion
    verifieras.

-   Dokumenterade rutiner för uppdateringar, brandväggsregler och
    administratörsåtkomst.

## 11. Föreslagen fortsatt plan

| Steg \| Aktivitet \| Resultat \|

| --- \| --- \| --- \|

| 1. Bekräfta målbild \| Bestäm om Windows/AD och Ubuntu/Samba ska
  samexistera eller om en lösning ska väljas. \| Tydlig och sammanhållen
  driftarkitektur. \|

| 2. Kartlägg nätet \| Fastställ router, switch, accesspunkter, IP-plan,
  VLAN och brandväggsregler. \| Dokumenterad nätverksdesign. \|

| 3. Slutför säkerheten \| Verifiera behörigheter, administratörskonton,
  uppdateringar och loggning. \| Minskad risk för obehörig åtkomst. \|

| 4. Inför backup \| Bestäm schema, lagringsplats, ansvar och
  återställningstest. \| Återställningsbar miljö. \|

| 5. Testa och dokumentera \| Testa inloggning, DNS, filåtkomst, avslag
  för obehöriga och klientanslutning. \| Verifierad och överlämningsbar
  lösning. \|

## 12. Kundpitch -- förslag på muntlig presentation

"Vi har tagit fram en grund för en mer säker och lättadministrerad
IT-miljö för Skämt & Sido Transport AB. Genom central användarhantering
och avdelningsbaserade säkerhetsgrupper får medarbetarna åtkomst till
rätt information utifrån sin roll. Det minskar risken för att känsliga
dokument blir tillgängliga för fel personer.

Vi har dokumenterat fildelning i både Windows Server och Ubuntu Server,
med separata resurser för Ledning, Ekonomi, IT, Logistik och Gemensamt.
Nästa steg är att fastställa den slutliga serverarkitekturen,
komplettera med en dokumenterad nätverksdesign och införa verifierade
rutiner för backup, uppdateringar och övervakning. Resultatet blir en
tydligare IT-miljö som kan växa med verksamheten."

## 13. Förslag på presentationsbilder

1.  Företagets behov och projektets mål.

2.  Översikt över dokumenterade system och tjänster.

3.  Active Directory: användare, grupper och domän.

4.  Fildelning i Windows och Ubuntu/Samba.

5.  Behörighetsmodellen och principen om minsta behörighet.

6.  Föreslagen nätverkstopologi och VLAN-segmentering.

7.  Router- och nätverksutrustning: rekommendationer.

8.  Säkerhetsrutiner, AI-policy och nästa steg.

## 14. Avgränsningar och punkter att bekräfta

-   Projektarkivet visar domänen ad.skamt.se och en Linux-server med
    adress 192.168.50.50/24.

-   Arkivet innehåller ingen komplett IP-plan för Windows-klienter eller
    domänkontrollant.

-   Det framgår inte om Windows-fildelningen och Samba-fildelningen är
    parallella lösningar eller alternativ.

-   Ingen routermodell, switchmodell eller installerad
    VLAN-konfiguration anges i projektfilerna.

-   Filerna för serverhärdning, backup/återställning och felsökning är
    tomma.

-   Routerval, VLAN och driftförslag i detta dokument är därför
    målbild/rekommendationer och behöver beslutas och testas.

## Källunderlag

Dokumentet har sammanställts utifrån följande filer i det uppladdade
projektarkivet:

-   Docs/02.DC & ADDS Skämt & Sido Transport AB.md

-   Docs/01. share folder.md

-   Docs/07-Behörighetsmodell & Fildelning i Ubuntu Server (Samba).md

-   Docs/05-AI.md

-   Skärmbilder i Docs/Pictures/

Övriga projektfiler med rubrikerna server-hardening,
backup/disaster-recovery och troubleshooting saknade innehåll i arkivet.

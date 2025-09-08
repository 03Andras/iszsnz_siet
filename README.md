# RouterOS Skolska Siet Konfiguracia

Tento repository obsahuje konfiguraciu RouterOS/MikroTik switch-ov pre skolsku/internátnu sietovu infrastrukturu.

## 🎯 Ciele a Poziadavky

### Sietova Struktura
- **ether1**: Internet pripojenie (DHCP client)
- **ether9-16**: Ucitelska siet (192.168.88.0/24) - vysoka priorita
- **ether17-24**: Ziacka siet (192.168.90.0/24) - obmedzeny pristup

### Prioritny System
1. **Najvyssia priorita**: Ucitelska siet (vzdy garantovany internet)
2. **Vysoka priorita**: Vzdelavaci obsah (EduPage, Wikipedia, Google vyhladavanie)
3. **Stredna priorita**: Normalne webove prehliadanie
4. **Nizka priorita**: Video streaming (YouTube, Netflix)

### Filtrovanie Obsahu a Obmedzenia
- ✅ **Povolene s vysokou prioritou**: EduPage, Wikipedia, Google vyhladavanie, vzdelacie stranky
- ⚠️ **Obmedzene**: YouTube, Facebook, video streaming sluzby
- ❌ **Zakazane**: TikTok, pornograficky obsah, torrent/P2P, velke subory na stiahnutie

## 📁 Subory

### `config.txt`
Podrobne komentovana konfiguracia pre vzdelacie ucely a pochopenie.

### `script.txt`
Priamo importovatelny RouterOS script - toto je **hlavny instalacny subor**.

### `README.md`
Tento dokument - instalacny navod a popis pouzitia.

## 🚀 Instalacny Navod

### 1. Priprava

**DOLEZITE**: Ulozenie aktualnej konfiguracie ako backup!

```bash
# RouterOS terminal
/export file=backup-$(date +%Y%m%d)
```

### 2. Metody Instalacie

#### A) S WinBox-om (Odporucane)
1. Otvorte WinBox
2. Pripojte sa k RouterOS zariadeniu
3. Menu: `Files` → Pretiahnite `script.txt` subor
4. Menu: `New Terminal`
5. Napiste: `/import script.txt`
6. Stlacte Enter

#### B) SSH/Telnet
1. Pripojte sa SSH alebo Telnet-om k RouterOS
2. Skopirujte obsah `script.txt` po castiach, alebo
3. Nahrajte subor a importujte: `/import script.txt`

#### C) Web rozhranie
1. Prihlaste sa do RouterOS web rozhrania
2. Menu: `Files` → Nahrajte `script.txt` subor
3. Menu: `Terminal` → Napiste: `/import script.txt`

### 3. Overenie Instalacie

Po instalacii overte:

```bash
# Sietove rozhrania
/interface bridge print

# DHCP servery
/ip dhcp-server print

# Firewall pravidla
/ip firewall filter print

# QoS nastavenia
/queue tree print

# DNS nastavenia
/ip dns print
```

## ⚙️ Detaily Konfiguracie

### Sietove Nastavenia
- **Ucitelska siet**: 192.168.88.0/24 (gateway: 192.168.88.1)
- **Ziacka siet**: 192.168.90.0/24 (gateway: 192.168.90.1)
- **DNS servery**: OpenDNS Family Shield + Cloudflare Family
- **DHCP lease cas**: Ucitelia 4 hodiny, ziaci 2 hodiny

### Rozdelenie Sirky Pasma
- **Ucitelia**: 60M garantovane, 100M maximum
- **Ziak vzdelavacie**: 20M garantovane, 40M maximum
- **Ziak normalne**: 10M garantovane, 30M maximum
- **Ziak video**: 2M garantovane, 15M maximum

### Bezpecnostne Funkcie
- Izolovanie ziakov medzi sebou
- Automaticke odpojenie pri prilis velkej nagate pripojeni (50+ suctasnych pripojeni)
- Prerusenie velkych subtorov nad 100MB
- Komplexne filtrovanie obsahu

## 🛠️ Prispôsobenie

### Modifikacia Sirky Pasma
V `script.txt` subore najdite Queue Tree cast a upravte limit hodnoty:

```bash
# Priklad: Zvysenie ucitelskej sirky pasma na 80M
/queue tree add name="2-Teachers-Out" parent="1-Internet-Out" packet-mark=teachers-out limit-at=80M max-limit=100M priority=1 queue=default
```

### Pridanie Novych Vzdelacich Stranok
```bash
# Pridanie novych adries do edu-priority zoznamu
/ip firewall address-list add address=codecademy.com list=edu-priority comment="Codecademy"
/ip firewall address-list add address=udemy.com list=edu-priority comment="Udemy"
```

### Dalsie Zakazane Stranky
```bash
# Pridanie noveho Layer7 protokolu
/ip firewall layer7-protocol add name=gaming regexp="^.*(steam|battle\\.net|riot|epicgames).*$"
/ip firewall filter add chain=forward src-address=192.168.90.0/24 layer7-protocol=gaming action=drop comment="Blokovanie hier"
```

## 📊 Monitorovanie a Riesenie Problemov

### Monitorovanie Prevadzky
```bash
# Queue statistiky
/queue tree print stats

# Aktivne pripojenia
/ip firewall connection print count-only

# IP adresy s najvacsou nagatou sirky pasma
/tool torch interface=bridge-students
```

### Caste Problemy

#### 1. Ucitelia nemaju internet
```bash
# Overenie DHCP client-a
/ip dhcp-client print

# Overenie NAT pravidla
/ip firewall nat print
```

#### 2. Ziaci nedostanu IP adresu
```bash
# Stav DHCP servera
/ip dhcp-server print
/ip dhcp-server lease print
```

#### 3. Filtrovanie obsahu nefunguje
```bash
# Stav Layer7 protokolov
/ip firewall layer7-protocol print

# Overenie firewall pravidiel
/ip firewall filter print stats
```

## 🔧 Udrzba

### Denne Ulohy
- Overenie statistik prevadzky
- Hladanie neobycajnej aktivity
- Prehladanie DHCP lease-ov

### Tyzdenove Ulohy
- Vytvorenie backup konfiguracie
- Overenie aktualizacie firmware
- Prehladanie logov

### Mesacne Ulohy
- Uplna revzia konfiguracie
- Analyza pouzitia sirky pasma
- Aktualizacia bezpecnostnych nastaveni

## 📞 Podpora

### Overenie Logov
```bash
# Systemove logy
/log print where topics~"system"

# Firewall logy (ak su povolene)
/log print where topics~"firewall"
```

### Obnovenie Konfiguracie
Ak narazite na problem, mozete obnovit predchadzajuci backup:

```bash
# Import backup-u
/import backup-20231201.rsc

# Alebo uplny reset (NEBEZPECNE!)
/system reset-configuration no-defaults=yes skip-backup=yes
```

## ⚠️ Upozornenia

1. **Vzdy vytvorte backup** pred zmenami
2. **Testujte** konfiguraciu najprv s malou skupinou
3. **Neupravujte** NAT pravidla, ak si nie ste isty
4. **Sledujte** systemove zdroje (CPU, pamat)

## 📝 Changelog

### v1.0 (2023-12-01)
- Zakladna sietova konfiguracia
- Implementacia filtrovania obsahu
- QoS a traffic shaping
- Bezpecnostne nastavenia

---

**Vytvoril**: RouterOS Sietova Konfiguracia  
**Posledna zmena**: 2023-12-01  
**Verzia**: 1.0  
**Kompatibilita**: RouterOS 7.x
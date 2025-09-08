# RouterOS Iskola Hálózat Konfiguráció

Ez a repository tartalmazza a RouterOS/MikroTik switch-ek konfigurációját egy iskola/kollégium hálózati infrastruktúrájához.

## 🎯 Célok és Követelmények

### Hálózati Struktúra
- **ether1**: Internet kapcsolat (DHCP client)
- **ether9-16**: Nevelői hálózat (192.168.88.0/24) - magas prioritás
- **ether17-24**: Diák hálózat (192.168.90.0/24) - korlátozott hozzáférés

### Prioritási Rendszer
1. **Legmagasabb prioritás**: Nevelői hálózat (mindig garantált internet)
2. **Magas prioritás**: Oktatási tartalom (EduPage, Wikipedia, Google keresés)
3. **Közepes prioritás**: Normál webböngészés
4. **Alacsony prioritás**: Video streaming (YouTube, Netflix)

### Tartalomszűrés és Korlátozások
- ✅ **Engedélyezett magas prioritással**: EduPage, Wikipedia, Google keresés, oktatási oldalak
- ⚠️ **Korlátozott**: YouTube, Facebook, video streaming szolgáltatások
- ❌ **Tiltott**: TikTok, pornográf tartalom, torrent/P2P, nagy fájl letöltések

## 📁 Fájlok

### `config.txt`
Részletesen kommentezett konfiguráció oktatási célokra és megértéshez.

### `script.txt`
Közvetlen importálásra alkalmas RouterOS script - ez a **fő telepítési fájl**.

### `README.md`
Ez a dokumentum - telepítési útmutató és használati leírás.

## 🚀 Telepítési Útmutató

### 1. Előkészületek

**FONTOS**: Mentse el a jelenlegi konfigurációt backup-ként!

```bash
# RouterOS terminálban
/export file=backup-$(date +%Y%m%d)
```

### 2. Telepítés Módszerek

#### A) WinBox-szal (Ajánlott)
1. Nyissa meg a WinBox-ot
2. Csatlakozzon a RouterOS eszközhöz
3. Menü: `Files` → Húzza be a `script.txt` fájlt
4. Menü: `New Terminal`
5. Írja be: `/import script.txt`
6. Nyomja meg az Enter-t

#### B) SSH/Telnet-tel
1. Csatlakozzon SSH-val vagy Telnet-tel a RouterOS-hez
2. Másolja be a `script.txt` tartalmát részletekben, vagy
3. Töltse fel a fájlt és importálja: `/import script.txt`

#### C) Web interfész
1. Lépjen be a RouterOS web felületére
2. Menü: `Files` → Töltse fel a `script.txt` fájlt
3. Menü: `Terminal` → Írja be: `/import script.txt`

### 3. Telepítés Ellenőrzése

A telepítés után ellenőrizze:

```bash
# Hálózati interfészek
/interface bridge print

# DHCP szerverek
/ip dhcp-server print

# Tűzfal szabályok
/ip firewall filter print

# QoS beállítások
/queue tree print

# DNS beállítások
/ip dns print
```

## ⚙️ Konfiguráció Részletei

### Hálózati Beállítások
- **Tanári hálózat**: 192.168.88.0/24 (gateway: 192.168.88.1)
- **Diák hálózat**: 192.168.90.0/24 (gateway: 192.168.90.1)
- **DNS szerverek**: OpenDNS Family Shield + Cloudflare Family
- **DHCP lease idő**: Tanárok 4 óra, diákok 2 óra

### Sávszélesség Elosztás
- **Tanárok**: 60M garantált, 100M maximum
- **Diák oktatási**: 20M garantált, 40M maximum
- **Diák normál**: 10M garantált, 30M maximum
- **Diák videó**: 2M garantált, 15M maximum

### Biztonsági Funkciók
- Diákok izolálása egymástól
- Automatikus lekapcsolás túl sok kapcsolat esetén (50+ egyidejű kapcsolat)
- Nagy fájl letöltések megszakítása 100MB felett
- Komprehenzív tartalomszűrés

## 🛠️ Személyre Szabás

### Sávszélesség Módosítása
A `script.txt` fájlban keresse meg a Queue Tree részlegét és módosítsa a limit értékeket:

```bash
# Példa: Tanári sávszélesség növelése 80M-re
/queue tree add name="2-Teachers-Out" parent="1-Internet-Out" packet-mark=teachers-out limit-at=80M max-limit=100M priority=1 queue=default
```

### Új Oktatási Oldalak Hozzáadása
```bash
# Új címek hozzáadása az edu-priority listához
/ip firewall address-list add address=codecademy.com list=edu-priority comment="Codecademy"
/ip firewall address-list add address=udemy.com list=edu-priority comment="Udemy"
```

### További Tiltott Oldalak
```bash
# Új Layer7 protokoll hozzáadása
/ip firewall layer7-protocol add name=gaming regexp="^.*(steam|battle\\.net|riot|epicgames).*$"
/ip firewall filter add chain=forward src-address=192.168.90.0/24 layer7-protocol=gaming action=drop comment="Gaming blokkolás"
```

## 📊 Monitorozás és Hibakeresés

### Forgalom Monitorozása
```bash
# Queue statisztikák
/queue tree print stats

# Aktív kapcsolatok
/ip firewall connection print count-only

# Legtöbb sávszélességet használó IP-k
/tool torch interface=bridge-students
```

### Gyakori Problémák

#### 1. Nincs internet a tanároknak
```bash
# Ellenőrizze a DHCP client-et
/ip dhcp-client print

# Ellenőrizze a NAT szabályt
/ip firewall nat print
```

#### 2. Diákok nem kapnak IP címet
```bash
# DHCP szerver állapot
/ip dhcp-server print
/ip dhcp-server lease print
```

#### 3. Tartalomszűrés nem működik
```bash
# Layer7 protokollok állapota
/ip firewall layer7-protocol print

# Tűzfal szabályok ellenőrzése
/ip firewall filter print stats
```

## 🔧 Karbantartás

### Napi Feladatok
- Forgalom statisztikák ellenőrzése
- Rendellenes aktivitás keresése
- DHCP lease-ek áttekintése

### Heti Feladatok
- Konfiguráció backup készítése
- Firmware frissítés ellenőrzése
- Naplók áttekintése

### Havi Feladatok
- Teljes konfiguráció felülvizsgálata
- Sávszélesség használat elemzése
- Biztonsági beállítások frissítése

## 📞 Támogatás

### Logok Ellenőrzése
```bash
# Rendszer logok
/log print where topics~"system"

# Tűzfal logok (ha engedélyezve)
/log print where topics~"firewall"
```

### Konfiguráció Visszaállítása
Ha problémába ütközik, visszaállíthatja a korábbi backup-ot:

```bash
# Backup importálása
/import backup-20231201.rsc

# Vagy teljes reset (VESZÉLYES!)
/system reset-configuration no-defaults=yes skip-backup=yes
```

## ⚠️ Figyelmeztetések

1. **Mindig készítsen backup-ot** a módosítások előtt
2. **Tesztelje** a konfigurációt kis csoporttal először
3. **Ne módosítsa** a NAT szabályokat, ha nem biztos benne
4. **Figyelje** a rendszer erőforrásokat (CPU, memória)

## 📝 Changelog

### v1.0 (2023-12-01)
- Alapvető hálózati konfiguráció
- Tartalomszűrés implementálása
- QoS és traffic shaping
- Biztonsági beállítások

---

**Készítette**: RouterOS Hálózati Konfiguráció  
**Utolsó módosítás**: 2023-12-01  
**Verzió**: 1.0  
**Kompatibilitás**: RouterOS 7.x
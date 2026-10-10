# Lenovo Tab3 7 (TB3-710F) – instalace Resurrection Remix 7.1.2

Resurrection Remix N (Android 7.1.2) místo původního Androidu 5.0.1 od Lenova (z Linuxu).

ROM, TWRP, odkaz na GApps a původní návod: [XDA – Resurrection Remix N v5.8.5 for TB3-710F](https://xdaforums.com/t/discontinued-rom-unofficial-7-1-2-resurrection-remix-n-v5-8-5-for-lenovo-tb3-710f.4705982/)

## Zařízení

| | |
|---|---|
| Model | Lenovo Tab3 7, **TB3-710F** (Wi-Fi) |
| Čip | MediaTek **MT8127**, 32bit ARM (`armeabi-v7a`) |
| Úložiště | 8 GB eMMC |
| Původní firmware | `TB3-710F_151211` (Android 5.0.1) |
| Nová ROM | Resurrection Remix N v5.8.5, unofficial, build z prosince 2024 |

## Základní poznatky (lol)

- V SP Flash Tool vždycky **Download Only**, **nikdy Format All + Download**.
- **NVRAM** je unikátní pro konkrétní kus a v žádném staženém firmwaru není. Jediná kopie je v záloze.
- Původní Lenovo Android při startu **přepisuje recovery zpátky na stock**. Po nahrání TWRP se tablet musí poprvé spustit rovnou do TWRP, ne do Androidu.
---

## 1. SP Flash Tool na Linuxu

Stáhnout z [spflashtool.com/download](https://spflashtool.com/download/). Heslo RAR `spflashtool.com`.

```bash
unrar x -pspflashtool.com SP_Flash_Tool-5.1916_Linux.rar
cd SP_Flash_Tool-5.1916_Linux
chmod +x flash_tool flash_tool.sh
./flash_tool.sh
```

Spadlo to na `libpng12.so.0: cannot open shared object file`. Knihovna už v nových distrech není, musí se vybalit z debianího balíku [libpng12-0_1.2.50-2+deb8u2_amd64.deb](https://archive.debian.org/debian-security/pool/updates/main/libp/libpng/libpng12-0_1.2.50-2+deb8u2_amd64.deb) (jen do složky SPFT, nic se neinstaluje):

```bash
dpkg -x libpng12-0_1.2.50-2+deb8u2_amd64.deb /tmp/png12
cp /tmp/png12/lib/x86_64-linux-gnu/libpng12.so.0* .
./flash_tool.sh
```

Nic dalšího v Linuxu (udev, ModemManager) se měnit nemusí.

## 2. Stock firmware a scatter

1. Stáhnout z [firmwarefile.com](https://firmwarefile.com/lenovo-tb3-710f/) a rozbalit stock firmware ZIP.
2. V SPFT na tabu **Download** → *Scatter-loading File* → vybrat `MT8127_Android_scatter.txt` z rozbaleného firmwaru.

Adresy (ze scatteru):

| Sektor | Start | Size |
|---|---|---|
| NVRAM | `0x400000` | `0x500000` (5 MB) |
| BOOTIMG | `0x1D80000` | `0x1000000` (16 MB) |
| RECOVERY | `0x2D80000` | `0x1000000` (16 MB) |
| USRDATA | `0x8CA00000` | `0x144100000` → konec paměti `0x1D0B00000` |

## Jak připojit tablet k SP Flash Tool

Platí pro Readback i Download:

1. Tablet **vypnutý** a **odpojený** od kabelu.
2. V SPFT kliknout na akci (*Read Back* / *Download*).
3. Držet **Volume Down** a připojit kabel.

## 3. Záloha celé paměti (Readback)

Tab **Readback** → *Add* → dvojklik na řádek:

- Soubor: `tb3_full_backup.bin` (potřeba ~7,5 GB místa)
- Region: `EMMC_USER`
- Start Address: `0x0`
- Length: `0x1D0B00000`

Kliknout nahoře *Read Back* a připojit tablet (viz výše).

Trvá cca 25 minut.

**Kontrola a vedlejší zálohy:**

```bash
ls -l tb3_full_backup.bin            # musí být 7796162560 bajtů
sha256sum tb3_full_backup.bin > tb3_full_backup.sha256

# NVRAM zvlášť (5 MB, nejcennější soubor)
dd if=tb3_full_backup.bin of=nvram_backup.bin bs=1M skip=4 count=5

# původní recovery zvlášť
dd if=tb3_full_backup.bin of=recovery_stock_backup.img bs=1M \
   iflag=skip_bytes,count_bytes skip=$((0x2D80000)) count=$((0x1000000))

# ověření, že adresy ze scatteru sedí (oba musí vypsat ANDROID!)
tail -c +$((0x2D80000 + 1)) tb3_full_backup.bin | head -c 8; echo
tail -c +$((0x1D80000 + 1)) tb3_full_backup.bin | head -c 8; echo
```

## 4. Nahrání TWRP

Použít **TWRP 3.7 z [XDA threadu](https://xdaforums.com/t/discontinued-rom-unofficial-7-1-2-resurrection-remix-n-v5-8-5-for-lenovo-tb3-710f.4705982/)**. Tzulovo TWRP 3.0 skončilo v nekonečné smyčce loga Lenovo.

1. Tab **Download**, načtený scatter.
2. Zaškrtnout jenom **RECOVERY**.
3. U RECOVERY v *Location* **přepsat cestu na TWRP `.img`**.
4. **Download Only** → Download → připojit tablet (viz výše).

## Spuštění do TWRP

Z vypnutého stavu: držet **Volume Up + Power**. Power držet, dokud se neobjeví logo Lenovo, pak ho pustit. Volume Up držet dál, dokud nenaběhne TWRP.

## 5. Instalace ROM v TWRP

1. Úvodní obrazovka: **Swipe to Allow Modifications**.
2. Dostat do tabletu oba ZIPy (odkazy v [XDA threadu](https://xdaforums.com/t/discontinued-rom-unofficial-7-1-2-resurrection-remix-n-v5-8-5-for-lenovo-tb3-710f.4705982/)):
   - `RR-N-v5.8.5-xxxxxxxx-TB3710F-Unofficial.zip` (ROM je ZIP, ne img)
   - `open_gapps-arm-7.1-pico-20220215.zip` (Open GApps: **ARM** / **7.1** / **pico**)

   Buď přes MTP (tablet v TWRP se v počítači ukáže jako `Lenovo/Internal Storage`), nebo přes adb:
   ```bash
   adb push RR-N-v5.8.5-*-TB3710F-Unofficial.zip /sdcard/
   adb push open_gapps-arm-7.1-pico-20220215.zip /sdcard/
   ```
3. **Wipe → Advanced Wipe**: zaškrtnout *Dalvik / ART Cache, System, Data, Cache*.  
**Ne Internal Storage**, tam jsou ZIPy.
1. **Install** → ROM ZIP → swipe. Nebo z počítače v `adb shell`: `twrp install /sdcard/RR-N-v5.8.5-…-TB3710F-Unofficial.zip`.
2. GApps **před prvním startem systému** (dodatečná instalace po spuštění dělá problémy). V TWRP 3.7 ale nejdou nainstalovat normálně, viz další kapitola.

## 6. GApps v TWRP 3.7

TWRP 3.7 je ořezané a chybí v něm `uname`, `unzip`, `install` i `head`. Instalátor Open GApps je potřebuje, takže padá na:

- `ERROR 64: Wrong architecture to set-up Open GApps' pre-bundled busybox`: chybí `uname` a `unzip`, busybox se nerozbalí
- `ERROR 10: Failed to set-up Open GApps' pre-bundled busybox`: chybí `install`, nevznikne `/tmp/bin`


**Na počítači** vytáhnout busybox z GApps a poslat do tabletu:

```bash
unzip -o open_gapps-arm-7.1-pico-20220215.zip busybox-arm -d /tmp/gapps
adb push /tmp/gapps/busybox-arm /tmp/busybox-arm
adb shell
```

**V shellu tabletu** (`TB3710F:/ #`).

```sh
chmod 755 /tmp/busybox-arm

# falešný uname
echo '#!/sbin/sh' > /sbin/uname
echo 'echo armv7l' >> /sbin/uname
chmod 755 /sbin/uname

# unzip přes busybox
echo '#!/sbin/sh' > /sbin/unzip
echo 'exec /tmp/busybox-arm unzip "$@"' >> /sbin/unzip
chmod 755 /sbin/unzip

# složka, kterou by jinak vytvořil chybějící "install"
mkdir -p /tmp/bin

uname -m      # musí vypsat armv7l
twrp install /sdcard/open_gapps-arm-7.1-pico-20220215.zip
```

Úpravy platí jen do restartu TWRP.

## 7. Dokončení

```sh
twrp wipe cache
twrp wipe dalvik
reboot
```

První start trvá 5–10 minut.

Známé chyby ROM (podle XDA): GPS se může špatně chytat, reproduktory hrají jeden kanál.

---

## Cesta zpět

**Zpět na stock Lenovo:** SPFT → načíst stock scatter → **Download Only** (všechny oddíly, které jsou ve scatteru předvybrané) → Download.

**Obnova jen NVRAM** (když nejde Wi-Fi nebo je divná MAC adresa): SPFT → pokročilý režim (`Ctrl+Alt+V`) → *Write Memory* → soubor `nvram_backup.bin`, region `EMMC_USER`, adresa `0x400000`.

**Obnova 1:1 z plné zálohy:** *Write Memory* s `tb3_full_backup.bin` od adresy `0x0`. Poslední možnost, dlouhé.

## Fily

| Soubor | Co to je |
|---|---|
| `tb3_full_backup.bin` + `.sha256` | kompletní backup paměti |
| `nvram_backup.bin` | NVRAM, 5 MB, **nenahraditelný** |
| `recovery_stock_backup.img` | původní recovery |
| stock firmware ZIP | `Lenovo_TB3-710F_…` se scatterem |
| TWRP 3.7 `.img` | funkční TWRP recovery |

## Poznámky

- **Root / Magisk** je už v RR.

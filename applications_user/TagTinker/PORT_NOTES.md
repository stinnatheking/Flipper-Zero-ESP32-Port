# TagTinker — samengevoegd voor Sor3nt/Flipper-Zero-ESP32-Port

Dit is de **volledige, nieuwste TagTinker-broncode** (van
github.com/i12bp8/TagTinker, v2.1 — inclusief NFC-scan) met drie
aanpassingen zodat het compileert tegen de ESP32-firmware-port in plaats
van tegen een echte Flipper Zero.

## Wat is er veranderd

1. **`ir/tagtinker_ir.c` vervangen.** Het origineel praat rechtstreeks met
   STM32-timerregisters (`TIM1`, `DWT->CYCCNT`) — dat bestaat niet op een
   ESP32. Dit bestand is 1-op-1 overgenomen uit de kopie die de
   Sor3nt-maintainer zelf al had aangepast: hij gebruikt
   `furi_hal_infrared_async_tx` (de Furi-HAL-abstractie), die de port
   vertaalt naar RMT op de ESP32-S3.
   **Let op:** die aangepaste driver knijpt de draaggolf af op **1 MHz**
   in plaats van de originele **1,255 MHz** (`INFRARED_MAX_FREQUENCY` van
   hun HAL). Staat zo in hun eigen commentaar. Of tags dat nog accepteren
   moet je testen.

2. **WiFi Plugins-feature verwijderd** (`wifi/`, `esp32-wifi-fw/`,
   `cloud-plugins/`, `shared/tt_wifi_proto_fap.h`, en de drie
   `scenes/tagtinker_scene_wifi_*.c`-bestanden). Die feature praat via
   UART met een los, fysiek Flipper WiFi Dev Board-plankje (eigen
   ESP32-S2) — jouw T-Embed heeft daar geen plek voor en is zelf al een
   ESP32 met wifi, dus dit sloot nergens op aan. Om dit netjes te
   verwijderen zonder de build te breken zijn ook aangepast:
   - `scenes/tagtinker_scene.h` / `.c` — de 3 WiFi-scenes uit de
     scene-tabellen gehaald (anders schuiven alle scene-indexen scheef)
   - `scenes/tagtinker_scene_target_actions.c` — het "WiFi Plugins"
     menu-item + de bijbehorende event-case verwijderd
   - `tagtinker_app.c` — de opruim-aanroep naar `tagtinker_wifi_free()`
     verwijderd (die functie bestond alleen in het weggehaalde `wifi/`)
   - `application.fam` — wifi-bronbestanden en de `"expansion"`-vereiste
     (GPIO-expansiepoort-detectie, alleen nodig voor het Dev Board)
     verwijderd

3. **Verder alles ongewijzigd gelaten** — `protocol/`, `nfc/` (de
   NFC-scan-barcode-decoder, werkt want de firmware heeft
   `components/nfc/protocols/mf_ultralight/mf_ultralight.h` al ingebakken
   omdat NFC een ondersteunde systeem-app is op dit board), alle overige
   scenes, `views/`, `web-image-prep/` (los stukje, hoort niet bij de
   FAP-build maar liet ik erin als naslag).

## Wat ik niet heb kunnen testen

Ik heb dit **niet** daadwerkelijk gebouwd tegen ESP-IDF/hun firmware —
dat vereist hun specifieke toolchain (ESP-IDF v5.4.1) die ik hier niet
heb draaien. Wel gecontroleerd: alle bestanden die in `application.fam`
staan bestaan ook echt, en de handmatig bewerkte bestanden hebben
kloppende haakjes/accolades (geen evidente syntaxfout). Een echte
compile-fout kan dus nog steeds naar boven komen — meld die gewoon hier
terug, dan kijk ik mee.

## Installeren

1. Clone `Sor3nt/Flipper-Zero-ESP32-Port`.
2. Zet deze map neer als `applications_user/TagTinker` (verwijder eerst
   hun oudere `applications_user/TagTinker-main/`, of geef deze map een
   andere naam en pas `application.fam`'s `appid` niet aan — die moet
   uniek blijven, dus niet allebei tegelijk laten staan).
3. Bouw de firmware zoals in hun README (`./buildAndFlash_T-Embed.sh`),
   of bouw alleen deze FAP met `./buildFap.sh applications_user/TagTinker`
   nadat de firmware zelf één keer gebouwd is.
4. Kopieer de resulterende `.fap` naar `/ext/apps/` (of de map die hun
   Archive-app gebruikt) op de SD-kaart.

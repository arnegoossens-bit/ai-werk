---
name: script-to-edit-capcut
description: Bouw automatisch een talking-head montage uit een script (Word-tabel met shots) en losse takes, kies per shot de take die het script het best volgt, controleer tekst en beeld, render een preview en zet na goedkeuring een kant-en-klaar CapCut-project klaar met push-ins en woord-per-woord ondertitels in de huisstijl van de klant (klantprofiel). Gebruik bij "maak de edit van video X", "script naar edit", "zet de shoot in CapCut", "nieuwe klant voor de auto-edit".
---

# Script naar edit (CapCut)

Van een videoscript en een map met losse takes (iPhone-clips) naar een gecontroleerde montage en een CapCut-project
met ondertitels in de huisstijl van de klant. **Deze skill doet enkel de edit en de ondertitels.** Stickers,
illustraties en motion design horen er niet in (die gebeuren daarna, bv. in After Effects).

Gebouwd en getest op de EnergieGenie-shoot Q4 2026 (klant energie.be): 5 video's, 105 clips, telkens 7 tot 8 shots,
59 tot 80 s, als TikTok; feedpost 4:5 en 1:1 gemeten op CapCut-exports. Alles wat hieronder als regel staat, is op
echt materiaal gemeten. Gemeten machinetijd per video
(19 clips, 6,4 min audio): 10 min 30 s, waarvan 7 min uitschrijven (cloud, 2 cores). Downloaden en nakijken niet
meegeteld.

## Wat je krijgt

1. Per shot uit het script de take die het script het best volgt (babbels, regie-opmerkingen, valse starts eruit).
2. Knippen op het gemeten stem-begin en -einde, niet op de woordtijden van Whisper.
3. Een controle-transcriptie van de afgewerkte audio, en een contactsheet van het eerste en laatste beeld van elke shot.
4. Een preview (540x960, HDR naar SDR omgezet, met push-ins) om goed te keuren.
5. **Na goedkeuring, altijd:** een CapCut-project met de clips, push-ins als keyframes en ondertitels in de huisstijl
   van de klant (woord-per-woord markering), dat meteen opent.

## Hoe de skill per klant werkt

Alles wat per klant verschilt, staat in een **klantprofiel**: een map `klanten/<klant>/` met twee bestanden.

| Bestand | Inhoud |
|---|---|
| `profiel.json` | huisstijl ondertitels (kleuren, positie, breedte, lettertype), montage-instellingen, afkortingen die voluit gezegd worden, vaste spellingscorrecties, exportmap en naamgeving |
| `style.json` | het CapCut-ondertitelsjabloon (woord-per-woord markering), een keer uit CapCut gehaald met `harvest_caption_style.py` |

De scripts lezen het profiel via twee verwijzingen:
- in `plan.json`: `"profile": "klanten/<klant>/profiel.json"` (gebruikt door `match.py` en `plan_edit.py`);
- in `job.json`: `"profile": "klanten/<klant>/profiel.json"` (gebruikt door `build_capcut.py`).

Een waarde die je in `job.json` zelf zet, wint altijd van het profiel. Zo pas je een enkele video aan zonder het
profiel te wijzigen. Zonder profiel werken de scripts met de standaardwaarden hieronder.

**Eerst vragen:** voor welke klant is de edit? Bestaat er al een profiel (zie onderaan: EnergieGenie), gebruik dat.
Is het een nieuwe klant, doe dan eerst "Nieuwe klant opzetten".

### Velden in `profiel.json`

```json
{
 "klant": "naam van de klant",
 "taal": "nl",
 "fps": 30,
 "ondertitels": {
  "style": "style.json",
  "beweging": "blok achter het actieve woord",
  "lettertype": "naam zoals in CapCut",
  "letters_per_eenheid": 18.3,
  "blok_kleur": "#RRGGBB",
  "rand": "#RRGGBB:0.08",
  "tekst_kleur": null,
  "schaal": null,
  "max_tekens": 42
 },
 "platformen": {"reel": {"y": -0.15, "max_breedte": 0.77}},
 "montage": {"push_in": true, "punch_in": 1.12},
 "expand": {"vk": "verenigd koninkrijk"},
 "vaste_correcties": [["wat Whisper hoort", "wat er moet staan"]],
 "export": {"map": "~/Documents/<Klant> Exports", "naam": "Video {n} - {titel}", "resolutie": "1080p", "fps": 30}
}
```

| Veld | Betekenis | Standaard |
|---|---|---|
| `ondertitels.style` | sjabloonbestand naast het profiel | geen (gewone CapCut-ondertitels) |
| `ondertitels.beweging` | hoe de ondertitels bewegen, zoals gekozen door de klant (enkel ter info, het sjabloon in `style.json` doet het werk) | |
| `letters_per_eenheid` | hoeveel hoofdletters op een regelbreedte van 1,0 passen. **Per lettertype meten** (zie "Nieuwe klant") | 18.3 (FT Aktual Bold) |
| `blok_kleur` | kleur van het blok achter het actieve woord (blok-sjabloon) | uit `style.json` |
| `rand` | tekstrand `kleur:breedte` | uit `style.json` |
| `tekst_kleur` | tekstkleur; bij het woordkleur-sjabloon de kleur van het actieve woord | uit `style.json` |
| `schaal` | grootte van de ondertitel | uit `style.json` |
| `platformen` | enkel invullen als een klant per platform een andere positie wil: `y` (verticale positie, -1 = onderrand, +1 = bovenrand) en `max_breedte` (deel van de beeldbreedte) | leeg: de platforminstellingen hieronder |
| `max_tekens` | maximaal aantal tekens per ondertitel | 42 |
| `montage.push_in` | trage push-in op hook, vragen en slot | true |
| `montage.punch_in` | punch-in bij een jump cut (1.0 = uit) | 1.12 |
| `expand` | afkortingen die voluit gezegd worden (voor het matchen) | VK, VS, CO2 en 0 tot 12 zitten er altijd in |
| `vaste_correcties` | woorden die Whisper bij deze klant fout hoort (merknamen, vaktermen); wordt toegepast als het in de video voorkomt, anders stil overgeslagen | geen |
| `export` | waar en hoe de video's uit CapCut geexporteerd worden (de export gebeurt met de hand in CapCut, stap 9) | |
| `fps` | beeldsnelheid van het CapCut-project. **Enkel 30 is getest**; `render_preview.py` werkt altijd op 30 | 30 |

## Nodig

- De Claude desktop-app met deze chat gekoppeld aan de computer (tools `get_device_info`, `device_request_folder_access`,
  `device_bash`, `device_stage_files`, `device_commit_files`), en computer use voor CapCut.
- CapCut desktop (internationale versie) op de Mac. Getest met CapCut 9.x. De projectbestanden moeten gewone JSON zijn
  (eerste byte `{`). Versleuteld (JianYing): stop en meld het.
- In de cloud-werkruimte: `pip install faster-whisper --break-system-packages` en ffmpeg. Voor een feedpost ook
  `pip install opencv-python-headless --break-system-packages` (versie 4.8 of hoger) en het YuNet-gezichtsmodel:
  `mkdir -p models && curl -sSL -o models/yunet.onnx https://media.githubusercontent.com/media/opencv/opencv_zoo/main/models/face_detection_yunet/face_detection_yunet_2023mar.onnx`
  (controleer: ongeveer 230 kB en begint niet met `{`; anders is het een foutmelding).
- Op de Mac (in `device_bash`): ffmpeg, python3 met numpy.
- Het lettertype van de klant moet op de Mac geinstalleerd zijn en een keer in CapCut gekozen zijn (CapCut kopieert het
  dan naar zijn eigen map, zie stap 8).
- Het klantprofiel (map `klanten/<klant>/`). Zet het in de werkmap van de cloud.
- Verder gebruikt: `SendUserFile` (preview sturen) en, als de clips op SharePoint staan, de browser (Claude in Chrome)
  om ze te downloaden.

## Mappen, paden en bestanden heen en weer

**Cloud-werkmap** (alles draait vanuit deze map, dus `python3 scripts/<naam>.py`):
```
scripts/            de scripts hieronder, letterlijk
klanten/<klant>/    profiel.json + style.json
tests/              testset + referentie (zie "Testen")
plan.json           stap 1
audio/              de wavs (van de Mac)
work/               words.json, <clip>_db.npy, cuts_<id>.json (stap 3-4)
edit.json           stap 5 (nieuwe naam per versie: edit_v2.json ...)
sjabloon/           ingebouwd CapCut-sjabloon (stap 8), letterlijk uit deze skill
build_v1/           CapCut-project (stap 8, nieuwe map per versie)
```
**Twee soorten Mac-paden, niet verwisselen:**
- In `device_bash` zie je de gekoppelde mappen als `$HOME/mnt/<map>/...`. Die paden gebruik je voor de Mac-scripts en
  als derde argument van `install_capcut.py`.
- In `job.json` staat `projects_root` als **echt Mac-pad** (`/Users/<user>/Movies/CapCut/User Data/Projects/com.lveditor.draft`).
  CapCut leest dat pad. De `/Users/<user>` vind je in `get_device_info` (de gekoppelde mappen). Met het mount-pad in
  `projects_root` vindt CapCut het project niet.

**Wat heen en weer moet** (`device_stage_files` = Mac naar cloud, `device_commit_files` = cloud naar Mac; die laatste
neemt enkel losse bestanden, maximaal 30 MB per bestand en 100 MB per keer, en schrijft het best naar een nieuw pad):

| Stap | Van Mac naar cloud | Van cloud naar Mac |
|---|---|---|
| begin | | `scripts/framesim.py`, `scripts/render_preview.py`, `scripts/install_capcut.py`, `scripts/hand_marks.py` naar de resultatenmap op de Mac |
| 2 | de wavs uit `audio/` en `hand.json` (naar `work/`) | |
| 5 | `fs/framesim.json` en `fs/sheet.jpg`; bij een feedpost ook de `head_*.jpg` (als tar) | `edit.json` |
| 7 | `preview.mp4` (om te sturen) | |
| 8 | | elk bestand uit `build_v<n>/`, ook die in `Timelines/<id>/`, met dezelfde mappenstructuur |

## Werkwijze

Maak bij de start een takenlijst van deze stappen. Er zijn twee vaste wachtmomenten: **goedkeuring van de preview** en
**CapCut volledig afgesloten** voor elke schrijfactie.

| # | Stap | Waar | Wacht op gebruiker |
|---|---|---|---|
| 0 | Intake (klant, platform, script, clips, export), toegang: map met clips, `~/Movies` (CapCut), map voor resultaten | Mac | antwoorden, toegang goedkeuren |
| 1 | Script inlezen naar `plan.json` (met verwijzing naar het profiel) | cloud | |
| 2 | Clips meten (ffprobe) en audio eruit halen (16 kHz mono wav) | Mac | |
| 3 | Transcriberen met woordtijden en volume per 10 ms (`transcribe.py`), daarna overgeslagen takes terughalen (`fill_gaps.py`) | cloud | |
| 4 | Takes matchen (`match.py`), topic bepalen, herformuleerde shots zetten met `recut.py` | cloud | |
| 5 | Edit plannen (`plan_edit.py`), kadrering meten (`framesim.py`), opnieuw plannen met die meting | cloud + Mac | |
| 6 | Controle tekst (`qa.py`) en beeld (contactsheet bekijken) | cloud + Mac | |
| 7 | Preview renderen (`render_preview.py`) en sturen | Mac | **goedkeuring** |
| 8 | CapCut-project bouwen (`build_capcut.py`) en installeren (`install_capcut.py`) | cloud + Mac | **CapCut afgesloten** |
| 9 | Openen in CapCut, controleren, exporteren, rapport | Mac | |

### 0. Intake en toegang
**Elke job start met dezelfde zes vragen.** Haal wat al in het bericht staat eruit en vraag de rest in een keer
(`AskUserQuestion`, of in tekst als die er niet is). Begin niet te bouwen voor alle zes bekend zijn.

**Achtergrond eerst.** Alles tot en met stap 8 draait zonder het scherm over te nemen: cloudwerk, bestandswerk op de Mac
(`device_bash`) en CapCut afsluiten via het menu (`computer_app_menu`, op de achtergrond; vraag het eerst, want
de gebruiker kan zelf in CapCut werken). De gebruiker kan intussen gewoon verder werken. Voorwaarde: de Mac blijft
wakker en de Claude-app open; valt de verbinding weg, dan stopt het werk op de Mac (gemeten: verbinding weg toen de
Mac sliep).

| # | Vraag | Waarom |
|---|---|---|
| 1 | **Klant** | kiest het klantprofiel (`klanten/<klant>/`). Geen profiel: eerst "Nieuwe klant opzetten" |
| 2 | **Platform**: TikTok, Instagram Reel, Instagram Story, website, of Instagram-feedpost (vraag dan: 4:5 of 1:1). Meerdere mag | zet het formaat en de positie en breedte van de ondertitels (tabel hieronder). Meerdere = een montage, een CapCut-project per platform |
| 3 | **Script en welke video** (bestand, topic of videonummer) | `plan.json` |
| 4 | **Map met de clips** | stap 2 |
| 5 | **Exportmap** (standaard uit het profiel) | stap 9 |
| 6 | **Mag Claude op het einde je scherm kort overnemen** om het project te openen en te exporteren? | ja = alles loopt automatisch tot de export; nee = Claude zet het klaar en jij opent en exporteert |

**Platforminstellingen** (`"platform"` in `job.json`, standaard `tiktok`):

| Platform | Formaat | Onderaan bedekt door de app | `caption_y` | Status |
|---|---|---|---|---|
| `tiktok` | 9:16 | 350 tot 480 px, rechts 120 px | -0.349 | gemeten op onze exports: 482 tot 522 px vrij onder de ondertitels |
| `story` | 9:16 | ongeveer 250 px boven en onder | -0.349 | past volgens de bron (inro.social); nog niet gemeten op een echte Story |
| `reel` | 9:16 | ongeveer 670 px, rechts ongeveer 770 px | -0.15 | berekend: blok van 1000 tot hoogstens 1247 px (bron hopperhq.com); meten op de eerste echte Reel |
| `website` | 9:16 | niets | -0.349 | |
| `feed45` | 4:5 (export 1080x1350) | niets | -0.349 | gemeten op een CapCut-export (2-11-2026): beeld vult het kader (0 zwarte rijen of kolommen), ondertitels even breed in px als bij TikTok, blok op 62% tot 70% van de hoogte |
| `feed11` | 1:1 (export 1080x1080) | niets | -0.349 | gemeten op een CapCut-export (2-11-2026): beeld vult het kader, ondertitels even breed in px, blok op 61% tot 71% van de hoogte |

**Feedpost (4:5 en 1:1).** De clips blijven 9:16 gefilmd; de bouwer zet het CapCut-kader zoals CapCut het zelf schrijft
(gemeten: 4:5 bestaat niet als knop in CapCut, enkel als Custom 1080x1350; 1:1 = 1920x1920) en vergroot elke clip tot hij
het kader vult (CapCut toont een clip op schaal 1 binnen het kader, met zwarte balken: gemeten). Bij 4:5 valt 570 px van
de hoogte weg (op 1080 breed), bij 1:1 840 px. Push-ins blijven, gerekend vanaf die vulschaal.

**Kadrering per shot (automatisch, met `heads` uit stap 5).** In het midden bijsnijden gaat in 1:1 mis: in video 6 komt
de kruin van shot 3 op het einde 1,7% boven de rand (berekend), en bij de close-up van shot 1 staat de kin op 68,5%,
onder de bovenkant van de ondertitels (gemeten op de export). Daarom schuift de bouwer elk shot verticaal: **omlaag** tot er boven het hoofd
minstens `kopruimte` vrij is (standaard 7% van de hoogte, profiel `montage.kopruimte`), **omhoog** als de kin onder 60%
zakt (daar beginnen de ondertitels), maar nooit zo ver dat het hoofd weer krap komt, en nooit zo ver dat de rand van de
clip in beeld komt. Gerekend op het einde van een push-in (dan is het hoofd het grootst). Het bouwrapport toont per shot
de verschuiving en waar hoofd en kin uitkomen, en meldt wat niet lukt.
Gemeten op de CapCut-export van video 6 in 1:1 (13 shots, begin- en eindbeeld): met 5% kopruimte bleef op het krapste
moment 2,8% vrij boven de kruin; met 7% minstens 4,9%, en de kin hoogstens op 61% (shot 1: van 68,5% naar 55 tot 57%). In 4:5 dezelfde berekening op de export nagemeten (met 5%): voorspelde en
gemeten kruin binnen 2,2%. De
gezichtsmeting schommelt tot 2,2% van de hoogte tussen frames, daarom 7%. Geen zwarte randen. Zonder `heads` waarschuwt
de bouwer en snijdt hij in het midden bij.
`transform y` van een clip telt in halve kaderhoogtes, + is omhoog (gemeten op de export: voorspelde en gemeten kruin
verschillen hoogstens 2,2%, de meetruis).

Hoe `caption_y` werkt (gemeten op exports in 9:16, 4:5 en 1:1): `y` telt in halve beeldhoogtes (-1 = onderrand), de
grootte van de ondertitels volgt de beeldbreedte (in alle drie de formaten exact even breed in px). Bovenrand van het
blok in px = H/2 x (1 - y) min a x (B/1080), met H en B de hoogte en breedte van het kader in CapCut en a de afstand van
het ankerpunt tot de bovenrand van het blok (35 tot 104 px naargelang de regel, gemeten). Na de eerste export op een
nieuw platform: meet de echte positie op de export en pas de tabel aan.

- Zet `klanten/<klant>/` in de cloud-werkmap.
- `get_device_info`, dan in een keer `device_request_folder_access` vragen voor: de map met clips, `~/Movies`
  (CapCut-projecten in `~/Movies/CapCut/User Data/Projects/com.lveditor.draft`) en een resultatenmap (bv. `~/Documents/<project>`).
- SharePoint: syncen lukt niet vanuit Claude (OneDrive-venster). Clips downloaden via de browser: per bestand met
  `fetch(download.aspx?SourceUrl=...)` naar een blob en die als download opslaan (een zip-download van een hele map stopt
  soms halverwege). Daarna de bytegrootte van elk bestand vergelijken met de SharePoint-API (`/_api/web/GetFolderByServerRelativePath(...)/Files?$select=Name,Length`).
  Chrome hernoemt een bestand dat al in Downloads staat naar "naam (1).MOV": verplaats op naam en exacte grootte.
  Bestanden van 0 bytes op SharePoint zijn mislukte uploads: meld ze, die shots ontbreken anders.

### 1. Script naar plan.json
Typisch scriptformaat: per topic/video een tabel met "Shot n" en de tekst, soms met een kolom icoon/illustratie
(die kolom negeert deze skill).
```json
{"profile": "klanten/<klant>/profiel.json",
 "videos": [{"id": "t1", "title": "...", "lines": [{"n": 1, "text": "..."}, {"n": 8, "role": "end", "text": "..."}]}],
 "expand": {"afk": "voluit voor dit ene script"}}
```
Klemtoon-accenten (láátste, Dáárom) mogen weg. Afkortingen die bij elke video van de klant terugkomen horen in het
profiel (`expand`), eenmalige in `plan.json`.

### 2. Clips meten (Mac)
```bash
ffprobe -v error -select_streams v:0 -show_streams -show_format -of json CLIP.MOV   # w, h, rotatie, fps, duur, color_transfer
ffmpeg -v error -y -i CLIP.MOV -vn -ac 1 -ar 16000 -c:a pcm_s16le audio/CLIP.wav
```
iPhone: 3840x2160 met rotatie -90 = staand, in CapCut `w 2160, h 3840`. `color_transfer arib-std-b67` = HDR (HLG): de
preview moet tonemappen, anders oogt alles verbleekt.

**Hand-teken (extra, altijd draaien):** zoek de takes die op de set als beste aangeduid zijn met een hand vóór de lens.
```bash
python3 scripts/hand_marks.py CLIPS hand.json MOV     # op de Mac; zet hand.json daarna in work/ in de cloud
```
Afspraak op de set: na een take die echt goed was, wacht de spreker 1 seconde en houdt iemand een hand helemaal vóór
de lens, ongeveer 2 seconden, en stopt dan de opname. Een bedekte lens = een beeld met bijna geen detail. Gemeten op
111 echte clips (2583 beelden, EnergieGenie): de laagste detailwaarde in een gewoon beeld was 12,06 (donkere kamer),
de mediaan 23,1. De drempel staat op 6,0. Het script leest enkel sleutelbeelden (iPhone: ongeveer 1 per seconde),
dus de hand moet minstens 1,5 s blijven. **Nog niet geijkt op een echte bedekte lens**: laat bij de eerste shoot met
dit teken een testclip maken (10 s filmen, dan 3 s hand vóór de lens) en kijk of `hand_marks.py` die vindt (de
uitvoer toont de laagste detailwaarde per clip). Geen enkel teken op de set = `hand.json` met lege lijsten: dan
verandert er niets.

### 3-4. Transcriberen en matchen (cloud)
```bash
python3 scripts/transcribe.py audio work medium nl        # words.json + <clip>_db.npy; "nl" = taal uit het profiel
python3 scripts/fill_gaps.py audio work medium nl          # altijd: haalt takes terug die Whisper oversloeg
python3 scripts/match.py plan.json t1 work                     # cuts_t1.json + leesbaar rapport
```
- **Tijd**: uitschrijven duurt ongeveer 1 minuut per minuut audio (gemeten: 7 min voor 6,4 min audio), langer dan een
  shell-opdracht mag duren. Draai `transcribe.py` op de achtergrond of herhaal hem tot alle clips erin staan: hij slaat
  clips over die al in `words.json` staan.
- **`fill_gaps.py` is niet optioneel.** Whisper slaat in lange clips soms hele herhalingen over (gemeten: in een clip van
  25 s met 3 pogingen gaf het enkel de eerste, met een "woord" van 5 s erover). Het script zoekt stukken waar het volume
  spraak toont maar geen geloofwaardig woord staat (stuk >= 1 s met < 40% dekking, of een gat >= 0,8 s binnen een zin) en
  schrijft enkel die opnieuw uit. Het vervangt alleen als er minstens 4 letters bijkomen (anders werd "dure contracten"
  ooit "huurcontracten"). Kijk de uitvoer na: op stilte verzint Whisper zinnetjes ("Bedankt.", "GELACH"); die zijn onschuldig.
- **Topic bepalen**: draai `match.py` voor elk topic op de map en kijk welk topic de shots vindt. Mapnummers kloppen niet
  altijd met topicnummers (gemeten bij EnergieGenie: een map "Video 2" bleek topic 1 te zijn). Controleer het aan de transcriptie.
- Regels in `match.py` (gemeten):
  - Score op **letters**, niet op woorden: "vanaf" = "van af", leestekens tellen niet; echte woordwissels ("hij" vs "die") wel.
  - **Script-getrouw eerst**: de take die het script het best volgt wint; pas bij (bijna) gelijke score de laatste take.
  - **Zinnen die op de set geschrapt zijn**: valt de volledige shot onder 0,8, dan matcht hij enkel de zinnen die wel gezegd zijn en meldt de rest als "niet gezegd op set".
  - **Continuiteit**: twee shots die in een adem in dezelfde take gezegd worden, blijven samen (geen jump cut).
  - **Knippen op het volume**: Whisper laat een eerste woord soms een seconde te vroeg beginnen. Stem-begin/einde = 5 frames van 10 ms boven ruisvloer + 18 dB; de knip valt op het stilste punt daarvoor/daarna.
- **Cijfers**: Whisper schrijft opgesomde punten als "3." terwijl het script "Drie:" heeft. `match.py` en de
  ondertitels lezen 0 tot 12 als woorden.
- **Hand-teken**: staat `work/hand.json` er, dan wint een take met een hand-teken erna (tussen het laatste woord en
  het volgende woord in de clip), als die het script goed volgt: score minstens 0,8 en hoogstens 0,10 onder de beste
  take. Een hand op een take die het script niet volgt, wint dus niet (gemeten in de test: een gemarkeerde take met
  0,59 werd genegeerd, een met 0,93 naast een beste van 0,94 won). De continuiteitsregel wisselt een gemarkeerde take
  nooit om. Het rapport toont "hand-teken op de set" bij de shot.
- Rapporteer aan de gebruiker: waar de gekozen take afwijkt van het script, en wat in geen enkele take gezegd wordt (dat kan de tool niet oplossen).

**Herformuleerd op de set (`recut.py`).** De matcher zoekt de scriptwoorden. Zegt de spreker een shot in eigen woorden,
of verdeelt die een shot over twee takes of clips, dan komt hij terug als CHECK, half, of NIET GEVONDEN. Lees dan de
volledige transcriptie van de kandidaat-clips en zet het shot zelf, op exacte gesproken woorden:
```bash
python3 scripts/recut.py work/cuts_t3.json work 2  IMG_0390 "Centraal staat" "uit te leggen" 1      # vervangt shot 2 (1e keer in de clip)
python3 scripts/recut.py work/cuts_t3.json work 2+ IMG_0390 "Daarop noteren" "dag van overnamen"     # tweede stuk achter shot 2 (laatste keer)
```
Randen komen op het gemeten stembegin en -einde, net als bij `match.py`. Kies per stuk de laatste volledige take, tenzij
een eerdere het script duidelijk beter volgt. Leg het recut-recept vast in een `recuts.sh`: na elke nieuwe
`match.py`-run (bv. na `fill_gaps.py`) draai je het opnieuw. Gemeten bij EnergieGenie: twee video's hadden 7 en 12
recuts nodig, twee andere geen of 2. Hoe vrijer de spreker praat, hoe meer recuts.

### 5. Edit plannen en kadrering
```bash
python3 scripts/plan_edit.py work/cuts_t1.json plan.json t1 edit.json                 # eerste versie
python3 scripts/framesim.py edit.json CLIPS fs                                         # op de Mac: framesim.json + sheet.jpg
python3 scripts/plan_edit.py work/cuts_t1.json plan.json t1 edit.json fs/framesim.json # met punch-ins
```
**Feedpost (4:5, 1:1): waar staat het hoofd.** Draai framesim op de Mac met `--kop` (ook de frames voor de
kopmeting: eerste beeld, een per seconde, laatste beeld van elk shot, 4 tegelijk; gemeten 79 s voor een video van 80 s,
zonder `--kop` 32 s). Haal de `head_*.jpg` naar de cloud (in een keer: `tar cf fs_head.tar head_*.jpg` in de map,
stagen, uitpakken in `fs/`) en meet daar:
```bash
python3 scripts/framesim.py edit.json CLIPS fs MOV --kop                              # op de Mac
python3 scripts/headroom.py edit.json fs models/yunet.onnx                            # in de cloud: fs/heads.json
```
`headroom.py` zoekt per frame het gezicht (OpenCV YuNet) en schrijft per shot de hoogste kruin en laagste kin. Gemeten
op 72 frames van de EnergieGenie-shoot: in elk frame het juiste gezicht, ook bij wie binnenwandelt of ver staat; de
kruin ligt 0,25 x de hoogte van het gezichtskader boven dat kader (haar en kale hoofden, nagekeken). Frames waar het
gezicht de bovenrand van de clip zelf raakt (binnen- of buitenwandelen) tellen niet mee: daar valt niets te kaderen.
- Shots uit dezelfde clip met minder dan 1 s ertussen worden een stuk.
- Trage push-in (4 tot 9%, 1,2% per seconde) op de hook, shots die met een vraag beginnen en het slot (uit te zetten
  in het profiel: `montage.push_in`). Controleer dit met de gebruiker: een retorische vraag midden in de video hoeft
  niet altijd een push-in.
- Punch-in (standaard 112%, profiel `montage.punch_in`) alleen bij een jump cut: correlatie tussen laatste beeld van
  shot k-1 en eerste beeld van shot k > 0,57 (gemeten: zelfde opstelling 0,589 tot 0,97; andere opstelling of ander
  standpunt tot 0,553).
- Op de Mac kan je niet verwijderen of altijd overschrijven: geef een nieuwe uitvoermap (`fs2`, `pv2`) en een nieuwe
  bestandsnaam (`edit_v2.json`) bij elke nieuwe versie. `framesim.py` en `render_preview.py` slaan bestaande frames en
  stukken over, dus een oude map geeft een oude preview.

### 6. Controle (verplicht voor de preview)
- `python3 scripts/qa.py work/cuts_t1.json AUDIO_DIR work/qa.wav`: plakt de gekozen audio, transcribeert opnieuw en vergelijkt
  per shot (letters). Alles >= 0,9 = OK. Lager: zelf nagaan of het een knip is of een Whisper-twijfel.
  Bekende Whisper-verzinsels op stilte ("Bedankt voor het kijken!") worden genegeerd.
- Taal: `python3 scripts/qa.py work/cuts_t1.json audio work/qa.wav nl` (taal uit het profiel als 4e argument).
- Lees ook de ondertitels na met een proefbouw: `python3 scripts/build_capcut.py job.json build_test` (de job uit stap 8,
  enkel bouwen, niet installeren; `build_test/build_report.json` en de teksten in `draft_info.json`). Daar zie je knipfouten die `qa.py` mist (een stuk dat midden in een
  zin begint: "ook andere factoren" waar "Er zijn ook andere factoren" gezegd werd; een herhaalde zin in een stuk).
- **Bekijk `sheet.jpg` zelf**: eerste en laatste beeld van elke shot. Tekstcontrole ziet geen lege kadrering, iemand die
  nog binnenwandelt of wegkijkt. Gemeten geval: een take begon met 0,9 s lege zetel terwijl de stem al liep; de andere
  take (stem pas bij binnenkomen) was beter.
- Meld regie-opmerkingen in de buurt van een gekozen take ("Vond ik weer niet zo...", "je keek naar buiten") en meet dat ze
  buiten het in/uit-punt vallen.

### 7. Preview
```bash
python3 scripts/render_preview.py edit.json CLIPS pv preview.mp4 MOV 4   # herhalen tot DONE (max 180 s per device_bash-call)
```
Feedpost: geef het formaat als laatste argument (`... MOV 4 4:5` of `... MOV 4 1:1`): de preview wordt dan in het
midden bijgesneden zoals CapCut doet (540x674 of 540x540, gemeten). Ondertitels staan niet in de preview.
Meet de duur van het resultaat tegen `edit.json` (verschil = enkel afronding op frames), bekijk een paar frames, stuur
de preview (`device_stage_files` + `SendUserFile`) en **wacht op goedkeuring**. Wijzigingen: terug naar stap 4 of 5.

### 8. CapCut-project (na goedkeuring, altijd)
1. **Sjabloon: zit in de skill (`sjabloon/`), de gebruiker hoeft niets klaar te zetten.** De bouwer kloont elk object
   uit een echt CapCut-project, nooit zelf verzonnen. Het ingebouwde sjabloon is het goedgekeurde EnergieGenie-sjabloon
   (iPhone-HDR-clip, HDR-modus 4, ruisonderdrukking aan op 85%), ingekort tot wat de bouwer leest en zonder
   klantgegevens. Gemeten (2-11-2026): de 4 goedgekeurde video's bouwen er veld voor veld identiek mee, en de CapCut-export
   van video 6 is identiek aan de goedgekeurde export (beeld en geluid: 0,0 verschil).
   **Neem geen willekeurig project van de gebruiker als sjabloon**: op de Mac van Arne hadden andere projecten
   HDR-modus 1, geen ruisonderdrukking of een gedempte clip (gemeten), en die instellingen komen dan in elke edit.
   Een ander sjabloon enkel met `"template_dir"` in de job, en dan eerst testen.
2. **Lettertypepad op deze Mac.** CapCut kopieert een gekozen lettertype naar zijn eigen map met een hash in de naam,
   en die hash verschilt per Mac. Zoek het pad op en geef het mee als `font_path`:
   ```bash
   ls ~/Library/Containers/com.lemon.lvoverseas/Data/Library/Fonts/ | grep -i "<deel van de lettertypenaam>"
   ```
   Vind je niets: open CapCut, kies het lettertype een keer op een tekst, sluit CapCut en zoek opnieuw. Gebruik het
   volledige pad (`/Users/<user>/Library/Containers/...`).
   **Ondertitelsjabloon: niets met de hand toe te passen.** CapCut zoekt de effecten van het sjabloon op hun id, niet
   op het pad in `style.json`. Gemeten (2-11-2026): een project waarin alle 88 effectpaden naar een niet-bestaande map
   wezen, toonde de markering gewoon, en CapCut schreef bij het openen 77 van die paden zelf om naar zijn eigen map.
   Nog niet gemeten op een Mac die het sjabloon nooit gebruikte (dan moet CapCut het via die id downloaden, met
   internet). Toont het project daar toch geen markering: open in CapCut een keer Captions > Templates, kies het
   sjabloon uit het profiel op een testproject, en bouw opnieuw. Meld het, dan kan deze regel weg.
3. `job.json` invullen en bouwen:
```json
{"name": "Klant Topic - auto-edit", "projects_root": "/Users/<user>/Movies/CapCut/User Data/Projects/com.lveditor.draft",
 "edit": "edit.json", "words": "work/words.json", "plan": "plan.json", "video": "t1",
 "clips": {"IMG_0317": {"dur": 22.1, "w": 2160, "h": 3840, "ext": "MOV"}},
 "profile": "klanten/<klant>/profiel.json", "platform": "tiktok",
 "font_path": "/Users/<user>/Library/Containers/com.lemon.lvoverseas/Data/Library/Fonts/<bestand>",
 "sub_phrases": [["fout gehoord in deze video", "juiste tekst"]]}
```
Ondertitels = gesproken woorden (timing) met de spelling van het script waar het duidelijk hetzelfde woord is. Automatisch:
cijfers als woorden, "QR -code" wordt "QR-code", komma's uit het script blijven, maar een punt of dubbelpunt uit het
script vervalt waar de spreker anders verder ging ("vertrouwd. aan." wordt "vertrouwd aan."), hoofdletter per zin, een
woord dat Whisper net buiten de knip zet terwijl de audio daar stil is ("Op" de elektriciteitsmarkt) komt erin.
Wat Whisper verkeerd hoort: een merknaam of vakterm die bij elke video terugkomt, zet je in het profiel
(`vaste_correcties`); iets wat enkel in deze video voorkomt in `sub_phrases` van de job. **Alleen gemeten**: schrijf
het stuk opnieuw uit met de scriptwoorden als `initial_prompt` en neem over wat dan twee keer hetzelfde terugkomt.
Wat onzeker blijft, meld je.
```bash
python3 scripts/build_capcut.py job.json build_v1
```
Meerdere platformen: bouw per platform een project met het platform in de naam (`"name": "Klant Topic - reel"`,
`"platform": "reel"`, map `build_reel_v1`). De montage en de ondertitels blijven gelijk, enkel de positie (en bij een
feedpost het kader) verschilt. Feedpost: `"platform": "feed45"` (4:5) of `"feed11"` (1:1) en `"heads": "fs/heads.json"`
(stap 5).
Zonder `font_path` stopt de bouwer. Is het een Trial-lettertype, dan waarschuwt hij: niet publiceren zonder de
gelicentieerde versie.
4. Vraag of CapCut **volledig afgesloten** is (of sluit het zelf via het menu CapCut > Quit). Commit elk bestand uit `build_v1/` (ook `Timelines/<id>/...`) naar een nieuwe map op de Mac en
   installeer in `device_bash` (vanuit de resultatenmap op de Mac, waar ook `scripts/` staat): `python3 scripts/install_capcut.py build_v1 CLIPS "$HOME/mnt/Movies/CapCut/User Data/Projects/com.lveditor.draft" BACKUPMAP`
   (alle paden hier zijn `$HOME/mnt/...`-paden). Dat maakt een backup van
   de index, kopieert de clips in de projectmap, schrijft de 6 tijdlijnkopieen + meta, zet het project in de index en
   verifieert (md5 van de 6 kopieen, alle media aanwezig, index-record).

### 9. Openen, controleren en exporteren
Twee handelingen lukken niet op de achtergrond (gemeten): **het project openen** (een klik op de achtergrond
selecteert het enkel) en **de exportmap kiezen** (het mapvenster hoort bij macOS, niet bij CapCut). Daarvoor is
volledige schermcontrole nodig.
- Antwoord "ja" op intakevraag 6: zeg in een zin wat je gaat doen ("ik neem je scherm over om project X te openen,
  na te kijken en naar map Y te exporteren"), vraag de schermcontrole aan en doe het. Geef het scherm meteen terug
  zodra de export loopt of klaar is.
- Antwoord "nee" of nog niet gevraagd: **stel het altijd voor** ("Het project staat klaar in CapCut. Zal ik je scherm
  kort overnemen om het te openen en te exporteren?"). Zegt de gebruiker nee, meld dan de projectnaam en de exportmap
  en stop.

Start CapCut, dubbelklik het project.
Speel af met de spatiebalk en bekijk de ondertitels op een paar momenten. Laat het project open voor de gebruiker.
Exporteren (als gevraagd): Export, instellingen uit het profiel (`export`), standaard 1080p, H.264, mp4, 30 fps. CapCut
maakt per video een submap (met .srt) in de laatst gebruikte exportmap en kapt lange namen af ("auto-ed"): zet de map
eerst goed (mapicoon, Cmd+Shift+G, pad) en zoek achteraf op map, niet op naam. Sluit het "opgeslagen"-venster met
Close, nooit met Share. Meet daarna op de export dat de ondertitels binnen de veilige zone vallen (zie onder).
Feedpost (gemeten): het exportvenster toont bij 4:5 "1080*1350 (720P)" en geeft een mp4 van 1080x1350; bij 1:1 staat er
1080P en komt er 1080x1080 uit. Na de export maakt CapCut in het opgeslagen-venster zelf nog een 9:16-versie voor
TikTok aan ("Generating"): negeer die en sluit met Close.

## Harde regels CapCut (gemeten, niet overslaan)

1. **Nooit schrijven terwijl CapCut openstaat.** Het overschrijft bij afsluiten.
2. **Media in de projectmap** (`<project>/video/`) met het pad `##_draftpath_placeholder_<UUID>_##/video/<clip>`. De
   UUID neem je over uit de eigen projecten van de gebruiker. CapCut op de Mac draait in een sandbox en kan bv. Downloads
   niet lezen ("Link media"-venster).
3. **Zes tijdlijnkopieen**: `draft_info.json`, `.bak`, `template-2.tmp` in de projectmap EN in `Timelines/<draft-id>/`.
   Alleen de root schrijven = CapCut opent het project niet (geen foutmelding).
4. **Commit naar nieuwe paden.** Een `device_commit_files` naar een bestaand bestand meldde "written" terwijl de Mac de
   oude inhoud hield. Schrijf elke versie naar een nieuwe naam of map en verifieer de inhoud (md5 of een veld) op de Mac.
5. Tijden in microseconden, ook `create_time`/`update_time`. Keyframe `time_offset` in brontijd (bronstart + offset),
   laatste keyframe 1 frame voor het einde. Clip-lengtes op de tijdlijn exact laten aansluiten (geen gaten van 1 us).
6. **Lokaal project, geen cloudkopie**: cloudvelden op -1 / 0 / "" zoals in de lokale projecten van de gebruiker.
7. Is een project-ID eenmaal "kapot" geopend, dan blijft CapCut het weigeren: bouw met een nieuw ID en verplaats de oude
   map naar `~/Movies/_to_delete` (verwijderen mag pas na toestemming).
8. Pop-ups bij opstarten (meldingen, Teams-reclame, "Auto upload stopped"): sluiten of "Cancel", nooit iets aankopen.

## Ondertitels: hoe de huisstijl werkt

CapCut-ondertitels met markering zijn **caption-sjablonen** (`text_templates`): een sjabloon met een binnentekst,
woordtijden (`words`, ms vanaf het begin van de ondertitel, spaties als tokens van 0 ms) en een caption-animatie die de
markering tekent. Die stijl haal je een keer per klant uit CapCut (`style.json`) en de bouwer past hem daarna toe op de
eigen scripttekst en woordtijden (dus geen CapCut-transcriptie met fouten).

**Gemeten in de sjabloon-code:**
- Blok-sjabloon "边框弹出Ⅰ" (resource `7535406282551971088`): het blok achter het actieve woord krijgt de
  **achtergrondkleur** van de tekst als die aanstaat (`background_style 1`, `background_color`, en `check_flag` +16),
  anders vast paars (121, 14, 231). Witte tekst op een licht blok: geef een donkere rand mee (`rand`).
  Dit sjabloon zet tekst in hoofdletters.
- Woordkleur-sjabloon "逐词高亮-发光黄" (resource `7543210783027547409`): witte tekst = actief woord puur geel #FFFF00;
  een andere tekstkleur = actief woord in die kleur, de rest wit (`tekst_kleur`).

**Positie en breedte (per platform, zie de tabel in stap 0):**
- TikTok legt onderaan 350 tot 480 px (beschrijving, hashtags, muziek) en rechts 120 px (knoppen) over de video.
  `y -0.349` in CapCut (-1 = onderrand, +1 = bovenrand): het ondertitelblok staat op 62% tot 75% van
  de hoogte (1191 tot 1438 px van 1920), onder de gezichten en boven de TikTok-zone. Eerst stond het op -0,625: dat gaf
  1456 tot 1703 px, deels achter de TikTok-beschrijving.
- Bij een Reel ligt de app-zone onderaan veel hoger (ongeveer 670 px): daar staan de ondertitels op `y -0.15`.
- Feedpost (4:5, 1:1): geen app-zone over het beeld. Dezelfde `y -0.349` zet het blok op 61% tot 71% van de hoogte,
  onder de kin (bekeken op de testexports).
- Breedte nooit meer dan 0,77 van het beeld (`max_breedte`), zodat de knoppen rechts vrij blijven.
- Lange woorden: het sjabloon breekt anders midden in een woord ("ELEKTRICITEITSMA / RKT"). De bouwer verbreedt de
  regel tot het langste woord past en verkleint die ene ondertitel pas als hij anders breder wordt dan `max_breedte`.
  Hoeveel letters op een regel passen hangt af van het lettertype: dat is `letters_per_eenheid` in het profiel
  (gemeten voor FT Aktual Bold in hoofdletters: 16 letters op regelbreedte 0,82 = 19,5 per eenheid, 18,3 gebruikt
  met marge; breedte op beeld = regelbreedte x schaal x 0,976).

## Nieuwe klant opzetten (een keer per klant)

1. **Huisstijl verzamelen.** De collega hoeft het lettertype of de kleurcodes niet te kennen: **Claude zoekt ze uit**
   en legt ze ter controle voor. Meet, gok niet. Bronnen, in deze volgorde:
   - **Brandbook-PDF**: `pdffonts brandbook.pdf` toont de ingebedde lettertypes met hun exacte naam en gewicht
     (bv. `FTAktual-Bold`). Kleurcodes staan meestal als HEX/RGB in de tekst (`pdftotext`).
   - **Website van de klant**: open de site in de browser en lees in de console de echte lettertypes en kleuren uit
     (`getComputedStyle(document.querySelector("h1")).fontFamily`, `color`, `background-color`, en
     `[...document.fonts].map(f => f.family + " " + f.weight + " " + f.status)` voor de geladen fonts). Zo is
     energie.be gemeten: FTAktualRegular/FTAktualBold, #FAFF96 en #221334.
   - **Figma-bestand van de klant** (als dat er is): de tekststijlen en kleurvariabelen uit het bestand.
   Rapporteer per gevonden lettertype: exacte naam en gewicht, waar gevonden, en of het **gelicentieerd** beschikbaar
   is. Een lettertype van de site is niet automatisch voor jullie gelicentieerd.
2. **Lettertype regelen.** Kijk op de Mac of het er al staat (`ls ~/Library/Fonts /Library/Fonts | grep -i <naam>`,
   en de CapCut-fontmap uit stap 8). Staat het er niet, of enkel als "Trial":
   - vraag de klant of de art director om de fontbestanden (de gelicentieerde versie);
   - is het lettertype niet te krijgen, stel dan een vrij lettertype voor dat er het dichtst bij ligt (bv. van Google
     Fonts), en laat die keuze uitdrukkelijk goedkeuren. Noteer in het profiel dat het een vervanger is.
   Installeer het op de Mac (dubbelklik, Font Book) en kies het een keer in CapCut, zodat CapCut het naar zijn eigen
   map kopieert.
3. **Eerst vragen hoe de ondertitels moeten bewegen.** Dat is een keuze van de klant of de art director, niet van
   Claude. Vraag het voor je iets bouwt (`AskUserQuestion`), met de gemeten opties:
   - **Blok achter het actieve woord**: een gekleurd blok springt van woord naar woord (EnergieGenie, sjabloon
     "边框弹出Ⅰ", resource `7535406282551971088`, zet tekst in hoofdletters);
   - **Actief woord in kleur**: het woord dat gezegd wordt licht op in de merkkleur, de rest blijft wit (sjabloon
     "逐词高亮-发光黄", resource `7543210783027547409`);
   - **Iets anders**: vraag een voorbeeld (een video van de klant of van een concurrent die ze goed vinden) en zoek in
     CapCut (Captions > Templates) het sjabloon dat er het dichtst bij komt. Een nieuw sjabloon is niet gemeten: test
     het eerst op een korte proef (markering, hoofdletters, lange woorden) voor je het in het profiel zet.
   **Doe altijd eerst voorstellen voor je bouwt**: maak van 2 tot 3 opties een korte proef van 5 seconden in de
   huisstijl van de klant (zelfde zin, zelfde beeld, enkel de beweging verschilt), stuur ze samen en laat kiezen.
   Weet de gebruiker het al, dan volstaat een proef van die ene keuze ter bevestiging.
   Noteer de keuze in het profiel (`ondertitels.beweging`).
4. **Sjabloon toepassen in CapCut**: open een project, Captions > Templates > Highlight, kies het gekozen sjabloon,
   "Generate captions" ("Delete current captions" uit). Kies het lettertype op een ondertitel.
5. **Stijl oogsten**: sluit CapCut, stage `draft_info.json` van dat project en draai
   ```bash
   python3 scripts/harvest_caption_style.py draft_info.json <resource_id> klanten/<klant>/style.json --background "#RRGGBB" --stroke "#RRGGBB:0.08" --font "<pad>"
   ```
6. **`letters_per_eenheid` meten** (enkel als het lettertype nog niet gemeten is): bouw een testproject met een
   ondertitel met een lang woord (bv. 16 hoofdletters), open in CapCut en verhoog `line_max_width` tot het woord net op
   een regel past. `letters_per_eenheid` = aantal letters / die breedte, min 6% marge.
7. **`profiel.json` invullen** (velden hierboven). Begin `vaste_correcties` leeg: vul aan met wat bij de eerste video's
   gemeten fout gehoord wordt (merknamen, vaktermen).
8. **Proefvideo**: maak een video, exporteer, meet de positie en breedte van de ondertitels op de export (veilige zone)
   en laat de huisstijl goedkeuren door de klant of de art director. Noteer de datum van goedkeuring in het profiel.
9. Deel het profiel met de collega's (de map `klanten/<klant>/`), of voeg het onderaan deze skill toe.

## Testen (na elke wijziging aan een script of profiel)
De testset (`tests/`) is een stuk van 19 s uit de goedgekeurde EnergieGenie-video 6 (3 clips, 11 ondertitels). De
referentie (`golden_<platform>.json`) is wat de goedgekeurde versie bouwde; voor 4:5 en 1:1 is het de versie die in
CapCut geexporteerd en gemeten is. Draai in de cloud-werkmap:
```bash
python3 scripts/test_skill.py              # ALLES OK of de eerste verschillen per platform
```
- Bouwt de testset voor alle zes platformen en vergelijkt per clip (bestand, in/uit, positie, schaal, push-in) en per
  ondertitel (tekst, timing, woordtijden, positie, breedte). Controleert ook: geen gaten, juist kader, clip vult het kader,
  ondertitels gelijk over de platformen.
- **FAIL = niet verspreiden.** Een bewuste wijziging (bv. een nieuwe positie die gemeten en goedgekeurd is): pas na
  goedkeuring van de gebruiker `--update`, en zeg welke waarden veranderden.
- Twee builds vergelijken (oude versus nieuwe scripts op dezelfde job): `python3 scripts/test_skill.py --compare A B`.
- Draai de test ook een keer na het opzetten op een nieuwe Mac: zo zie je meteen of de skill daar hetzelfde bouwt.

## Rapport aan het einde
Wat er gebouwd is (aantal shots, duur gemeten), welke takes afwijken van het script, wat nergens gezegd werd, wat je met
je ogen moet checken (opkomst, wegkijken), welke ondertitelcorrecties onzeker zijn, waar de backups staan en waar het
project staat.

## Scripts
Zet elk script letterlijk in `scripts/`. Cloud: transcribe, fill_gaps, match, recut, qa, plan_edit, headroom, build_capcut, harvest_caption_style, test_skill.
Mac (`device_bash`): hand_marks, framesim, render_preview, install_capcut.

### `transcribe.py`
```python
"""Transcribe each clip (16 kHz mono wav) with word timings + 10 ms loudness. usage: transcribe.py AUDIO_DIR OUT_DIR [model] [lang]"""
import json, sys, wave, glob, os
import numpy as np
from faster_whisper import WhisperModel
ad, od = sys.argv[1], sys.argv[2]
model = sys.argv[3] if len(sys.argv) > 3 else "medium"
lang = sys.argv[4] if len(sys.argv) > 4 else "nl"
os.makedirs(od, exist_ok=True)
m = WhisperModel(model, device="cpu", compute_type="int8")
out = json.load(open(f"{od}/words.json")) if os.path.exists(f"{od}/words.json") else {}
for p in sorted(glob.glob(f"{ad}/*.wav")):
    clip = os.path.basename(p)[:-4]
    if clip in out: continue
    wf = wave.open(p); a = np.frombuffer(wf.readframes(wf.getnframes()), np.int16).astype(np.float32) / 32768
    n = len(a) // 160
    db = 20 * np.log10(np.sqrt((a[:n*160].reshape(n,160)**2).mean(1) + 1e-12))
    np.save(f"{od}/{clip}_db.npy", db)
    segs, _ = m.transcribe(a, language=lang, word_timestamps=True, condition_on_previous_text=False, beam_size=5, temperature=0)
    ws = [{"w": w.word.strip(), "s": round(w.start,3), "e": round(w.end,3), "p": round(w.probability,3)} for s in segs for w in s.words]
    out[clip] = {"dur": round(len(a)/16000,3), "words": ws}
    json.dump(out, open(f"{od}/words.json","w"), ensure_ascii=False, indent=0)
    print(f"{clip} {len(a)/16000:6.1f}s | " + " ".join(w["w"] for w in ws), flush=True)
```

### `fill_gaps.py`
```python
"""Second pass after transcribe.py: Whisper sometimes skips whole retakes inside a long clip (it hears the first
attempt, then returns one word stretched over 5+ seconds, or nothing). This finds stretches where the 10 ms loudness
clearly shows speech but the transcript has no believable words, re-transcribes ONLY those stretches, and merges the
words back into words.json (old words inside a stretch are replaced).

usage: python3 fill_gaps.py AUDIO_DIR WORK_DIR [model] [lang]
A stretch = loudness > noise floor + 18 dB (pauses < 0.4 s bridged), at least 1.0 s long, of which less than 40% is
covered by words that last under 1.2 s (a real word is short; a 5 s "word" is Whisper smearing over speech it skipped);
or a hole of >= 0.8 s loud audio inside a transcribed stretch that no believable word covers.
"""
import json, os, sys, wave
import numpy as np
from faster_whisper import WhisperModel

AD, WD = sys.argv[1], sys.argv[2]
MODEL = sys.argv[3] if len(sys.argv) > 3 else "medium"
LANG = sys.argv[4] if len(sys.argv) > 4 else "nl"
out = json.load(open(f"{WD}/words.json"))
m = None
total = 0
for clip, rec in sorted(out.items()):
    db = np.load(f"{WD}/{clip}_db.npy")
    on = db > np.percentile(db, 10) + 18
    regions, i = [], 0
    while i < len(on):
        if not on[i]:
            i += 1; continue
        j = i
        while j < len(on) and (on[j] or on[j:j + 40].any()):
            j += 1
        if j - i >= 100:
            regions.append((i / 100, j / 100))
        i = j
    good = np.zeros(len(db), bool)
    for w in rec["words"]:
        if w["e"] - w["s"] < 1.2:
            good[int(w["s"] * 100):int(w["e"] * 100) + 1] = True
    todo = [(a, b) for a, b in regions if good[int(a * 100):int(b * 100)].mean() < 0.4]
    # also shorter holes INSIDE a transcribed stretch: >= 0.8 s of loud audio with no believable word on it
    # (a retake of half a sentence that Whisper merged away: "En die zet dan de prijs. [En die zet dan de prijs.] Daarom")
    hole = on & ~good
    k = 0
    while k < len(hole):
        if not hole[k]:
            k += 1; continue
        e_ = k
        while e_ < len(hole) and (hole[e_] or hole[e_:e_ + 30].any()):
            e_ += 1
        if e_ - k >= 80 and not any(a * 100 <= k and e_ <= b * 100 for a, b in todo):
            todo.append((k / 100, e_ / 100))
        k = e_
    if not todo:
        continue
    if m is None:
        m = WhisperModel(MODEL, device="cpu", compute_type="int8")
    wf = wave.open(f"{AD}/{clip}.wav")
    a = np.frombuffer(wf.readframes(wf.getnframes()), np.int16).astype(np.float32) / 32768
    for t0, t1 in todo:
        p0, p1 = max(0.0, t0 - 0.3), min(len(a) / 16000, t1 + 0.3)
        segs, _ = m.transcribe(a[int(p0 * 16000):int(p1 * 16000)], language=LANG, word_timestamps=True,
                               condition_on_previous_text=False, beam_size=5, temperature=0)
        new = [{"w": w.word.strip(), "s": round(w.start + p0, 3), "e": round(w.end + p0, 3), "p": round(w.probability, 3)}
               for s in segs for w in s.words]
        if not new:
            continue
        keep = [w for w in rec["words"] if not (p0 <= (w["s"] + w["e"]) / 2 <= p1)]
        old = [w for w in rec["words"] if p0 <= (w["s"] + w["e"]) / 2 <= p1]
        if sum(len(w["w"]) for w in new) < sum(len(w["w"]) for w in old) + 4:
            continue      # only when it ADDS speech (>= 4 letters more); never trade words for other words of equal length
        rec["words"] = sorted(keep + new, key=lambda w: w["s"])
        total += 1
        print(f"{clip} {t0:6.2f}-{t1:6.2f}s opnieuw: " + " ".join(w["w"] for w in new), flush=True)
json.dump(out, open(f"{WD}/words.json", "w"), ensure_ascii=False, indent=0)
print(f"{total} stukken opnieuw uitgeschreven")
```

### `match.py`
```python
"""Script-to-takes matcher.

For every script shot: search all clips for the window of words that best matches the shot text,
then pick the take. Cut edges are snapped into the quietest point between words (10 ms loudness).

usage: python3 match.py EDITPLAN.json VIDEO_ID WORK_DIR
  WORK_DIR/words.json  {clip: {"dur":s, "words":[{w,s,e,p}]}}   (transcribe.py)
  WORK_DIR/<clip>_db.npy                                         10 ms loudness per clip
  WORK_DIR/hand.json (optional, hand_marks.py)                   hand over the lens after a take = best take
writes WORK_DIR/cuts_<VIDEO_ID>.json and prints a readable report
"""
import json, os, re, sys, unicodedata
from difflib import SequenceMatcher
import numpy as np

PLAN, VID, WORK = sys.argv[1:4]
MIN_SCORE = 0.55      # below this a shot counts as "not found"
TAKE_MARGIN = 0.005   # takes this close to the most script-faithful one count as equal -> pick the LAST one
QUIET_SEARCH = 0.35   # s searched before/after a take for the quietest cut point
MIN_PAD = 0.04        # s of air kept before the first / after the last word at minimum

# spoken forms of abbreviations; extend per project with "expand": {"abbr": "spoken form"} in plan.json
NUMS = {"vk": "verenigd koninkrijk", "vs": "verenigde staten", "co2": "co twee"}
# whisper writes counted list items as digits ("3. De maandafrekening"); the script spells them out
NUMS.update({str(i): w for i, w in enumerate("nul een twee drie vier vijf zes zeven acht negen tien elf twaalf".split())})


def norm_tokens(text):
    t = unicodedata.normalize("NFKD", text.lower())
    t = "".join(c for c in t if not unicodedata.combining(c))
    t = t.replace("-", " ")
    out = []
    for w in t.split():
        w = re.sub(r"[^a-z0-9%]", "", w)
        if not w:
            continue
        out += NUMS.get(w, w).split()
    return out


def score(a, b):
    # letters only: "van af" == "vanaf", punctuation and spacing don't count; real word swaps still do
    return SequenceMatcher(None, "".join(a), "".join(b), autojunk=False).ratio()


plan = json.load(open(PLAN))
# client profile (plan.json "profile": "klanten/<klant>/profiel.json"): fixed spoken forms per client
if plan.get("profile"):
    NUMS.update({k.lower(): v.lower() for k, v in json.load(open(plan["profile"])).get("expand", {}).items()})
NUMS.update({k.lower(): v.lower() for k, v in plan.get("expand", {}).items()})
video = next(v for v in plan["videos"] if v["id"] == VID)
W = json.load(open(f"{WORK}/words.json"))
# optional: best-take signals from the set (hand over the lens after a good take), from hand_marks.py
HAND = json.load(open(f"{WORK}/hand.json")) if os.path.exists(f"{WORK}/hand.json") else {}
HAND_MARGIN = 0.10    # a marked take wins if its score is >= 0.8 and within 0.10 of the best take
clips = sorted(W)
order = {c: k for k, c in enumerate(clips)}
toks = {c: [(norm_tokens(w["w"]) or [""])[0] for w in W[c]["words"]] for c in clips}
db = {c: np.load(f"{WORK}/{c}_db.npy") for c in clips}


def candidates(line_toks):
    n = len(line_toks)
    out = []
    for c in clips:
        t = toks[c]
        best_here = []
        for size in range(max(2, int(n * 0.6)), int(n * 1.4) + 2):
            for i in range(0, len(t) - size + 1):
                s = score(line_toks, t[i:i + size])
                if s >= MIN_SCORE:
                    best_here.append({"clip": c, "i": i, "j": i + size - 1, "score": s})
        # non-overlapping, best first
        best_here.sort(key=lambda x: -x["score"])
        kept = []
        for x in best_here:
            if all(x["j"] < k["i"] or x["i"] > k["j"] for k in kept):
                kept.append(x)
        out += kept
    return out


def quiet_point(clip, t0, t1):
    d = db[clip]
    a, b = max(0, int(t0 * 100)), min(len(d), int(t1 * 100))
    if b <= a:
        return (t0 + t1) / 2
    return (a + int(np.argmin(d[a:b]))) / 100


def edges(c, i, j):
    ws = W[c]["words"]
    s, e = ws[i]["s"], ws[j]["e"]
    prev_e = ws[i - 1]["e"] if i > 0 else 0.0
    next_s = ws[j + 1]["s"] if j + 1 < len(ws) else W[c]["dur"]
    # Whisper's word edges drift (a first word can be smeared over a second of silence), so the real
    # voice onset/offset is measured in the 10 ms loudness: 5 frames in a row above noise floor + 18 dB.
    d = db[c]
    thr = float(np.percentile(d, 10)) + 18
    loud = lambda k: k + 5 <= len(d) and (d[k:k + 5] > thr).all()
    a0, a1 = int(max(prev_e, s - QUIET_SEARCH) * 100), int(e * 100)
    onset = next((k / 100 for k in range(a0, a1) if loud(k)), s)
    b0, b1 = int(s * 100), int(min(next_s, e + QUIET_SEARCH) * 100)
    offset = next(((k + 5) / 100 for k in range(b1 - 5, b0, -1) if loud(k)), e)
    t_in = quiet_point(c, max(prev_e, onset - QUIET_SEARCH), onset - MIN_PAD)
    t_out = quiet_point(c, offset + MIN_PAD, min(next_s, offset + QUIET_SEARCH))
    return round(max(0.0, t_in), 3), round(min(W[c]["dur"], t_out), 3)


def hand_marked(x):
    """True when a hand covered the lens after this take: between its last word and the next spoken word in the clip
    (or within 10 s when nothing is said after it)."""
    ws = W[x["clip"]]["words"]
    e = ws[x["j"]]["e"]
    nxt = ws[x["j"] + 1]["s"] if x["j"] + 1 < len(ws) else e + 10
    return any(e - 0.3 <= a <= max(nxt, e + 1.0) for a, b in HAND.get(x["clip"], []))


report, cuts = [], []
def sentences(text):
    return [s.strip() for s in re.split(r"(?<=[.?!])\s+", text) if s.strip()]


for line in video["lines"]:
    lt = norm_tokens(line["text"])
    cands = candidates(lt)
    full_best = max((x["score"] for x in cands), default=0)
    if full_best < 0.8 and len(sentences(line["text"])) > 1:
        # fallback: a sentence of this shot was cut on set -> match only the sentences that WERE said
        found = [s for s in sentences(line["text"])
                 if max((x["score"] for x in candidates(norm_tokens(s))), default=0) >= 0.8]
        if found:
            alt = candidates(norm_tokens(" ".join(found)))
            if alt and max(x["score"] for x in alt) > full_best:
                cands, lt = alt, norm_tokens(" ".join(found))
                line = dict(line, dropped=[x for x in sentences(line["text"]) if x not in found])
    if not cands:
        cuts.append({"n": line["n"], "status": "NIET GEVONDEN", "script": line["text"]})
        continue
    best = max(x["score"] for x in cands)
    good = [x for x in cands if x["score"] >= best - TAKE_MARGIN]
    pick = max(good, key=lambda x: (order[x["clip"]], x["i"]))
    marked = [x for x in cands if hand_marked(x) and x["score"] >= max(0.8, best - HAND_MARGIN)]
    if marked:
        pick = max(marked, key=lambda x: (order[x["clip"]], x["i"]))
    t_in, t_out = edges(pick["clip"], pick["i"], pick["j"])
    said = " ".join(w["w"] for w in W[pick["clip"]]["words"][pick["i"]:pick["j"] + 1])
    cuts.append({
        "n": line["n"], "status": "OK" if pick["score"] >= 0.8 else "CHECK",
        "hand": hand_marked(pick),
        "clip": pick["clip"], "in": t_in, "out": t_out, "dur": round(t_out - t_in, 3),
        "score": round(pick["score"], 3), "best_score": round(best, 3),
        "takes": sorted({(x["clip"], round(x["score"], 2)) for x in cands}, key=lambda x: (order[x[0]])),
        "script": line["text"], "said": said, "dropped_on_set": line.get("dropped", []),
        "_cands": cands, "_i": pick["i"],
        "words": [dict(w, s=round(w["s"] - t_in, 3), e=round(w["e"] - t_in, 3))
                  for w in W[pick["clip"]]["words"][pick["i"]:pick["j"] + 1]],
    })

# continuity: two consecutive shots spoken in one breath in the same take stay together (no jump cut mid-flow).
# Among clips where both shots have a good take back to back, take the LATEST clip.
CONT_MARGIN = 0.08
for k in range(len(cuts) - 1):
    a, b = cuts[k], cuts[k + 1]
    if "clip" not in a or "clip" not in b or a["clip"] == b["clip"] or a.get("hand") or b.get("hand"):
        continue
    ca, cb = a["_cands"], b["_cands"]
    ba, bb = max(x["score"] for x in ca), max(x["score"] for x in cb)
    pairs = [(x, y) for x in ca if x["score"] >= ba - CONT_MARGIN
             for y in cb if y["score"] >= bb - CONT_MARGIN and y["clip"] == x["clip"] and 0 < y["i"] - x["j"] <= 3]
    if pairs:
        # script fidelity first (summed score), then the latest take
        top = max(p[0]["score"] + p[1]["score"] for p in pairs)
        x, y = max([p for p in pairs if p[0]["score"] + p[1]["score"] >= top - 2 * TAKE_MARGIN],
                   key=lambda p: (order[p[0]["clip"]], p[0]["i"]))
        for cut, pick in ((a, x), (b, y)):
            if (cut["clip"], cut["_i"]) != (pick["clip"], pick["i"]):
                cut["note"] = f"continuïteit: take gewisseld van {cut['clip']} naar {pick['clip']}"
                t_in, t_out = edges(pick["clip"], pick["i"], pick["j"])
                cut.update(clip=pick["clip"], _i=pick["i"], **{"in": t_in, "out": t_out}, dur=round(t_out - t_in, 3),
                           score=round(pick["score"], 3),
                           said=" ".join(w["w"] for w in W[pick["clip"]]["words"][pick["i"]:pick["j"] + 1]),
                           words=[dict(w, s=round(w["s"] - t_in, 3), e=round(w["e"] - t_in, 3))
                                  for w in W[pick["clip"]]["words"][pick["i"]:pick["j"] + 1]])

for c in cuts:
    c.pop("_cands", None); c.pop("_i", None)

t = 0.0
for c in cuts:
    if "clip" in c:
        c["tl"] = round(t, 3)
        t += c["dur"]
json.dump({"video": VID, "total": round(t, 3), "cuts": cuts}, open(f"{WORK}/cuts_{VID}.json", "w"),
          ensure_ascii=False, indent=1)

print(f"{VID}: totaal {t:.2f}s")
for c in cuts:
    if "clip" not in c:
        print(f"\nshot {c['n']}: NIET GEVONDEN\n  script: {c['script']}")
        continue
    print(f"\nshot {c['n']} [{c['status']}] {c['clip']} {c['in']:.2f}-{c['out']:.2f} ({c['dur']:.2f}s) score {c['score']}"
          f" | takes: " + ", ".join(f"{k}:{s}" for k, s in c["takes"]))
    print(f"  script: {c['script']}")
    print(f"  gezegd: {c['said']}")
    if c.get("note"):
        print(f"  {c['note']}")
    if c.get("hand"):
        print("  hand-teken op de set: deze take werd als beste aangeduid")
    if c["dropped_on_set"]:
        print(f"  niet gezegd op set: {' '.join(c['dropped_on_set'])}")
```

### `recut.py`
```python
"""Set one shot to an exact spoken span, for lines that were reworded on set (the matcher looks for the SCRIPT wording,
so a fully rephrased line comes back partial or as CHECK). Edges are snapped to the quietest 10 ms point next to the
words, the same way match.py does it.

usage: python3 recut.py CUTS.json WORK_DIR SHOT CLIP "first words" "last words" [OCCURRENCE=last]
  e.g. recut.py work/cuts_t5.json work 2 IMG_0365 "Er komt namelijk" "voor je toe"
  SHOT "2"  replaces shot 2;  SHOT "2+" appends an extra piece to shot 2 (a line spoken in two takes / two clips)
  OCCURRENCE: 1, 2, ... or "last" (default: the last take of that span in the clip)
"""
import json, re, sys, unicodedata
import numpy as np

CUTS, WORK, SHOT, CLIP, A, B = sys.argv[1:7]
OCC = sys.argv[7] if len(sys.argv) > 7 else "last"
QUIET = 0.35
nz = lambda w: re.sub(r"[^a-z0-9]", "", "".join(c for c in unicodedata.normalize("NFKD", w.lower()) if not unicodedata.combining(c)))
W = json.load(open(f"{WORK}/words.json"))[CLIP]["words"]
db = np.load(f"{WORK}/{CLIP}_db.npy")
toks = [nz(w["w"]) for w in W]
a = [nz(x) for x in A.split()]; b = [nz(x) for x in B.split()]
spans = []
for i in range(len(toks) - len(a) + 1):
    if toks[i:i + len(a)] != a:
        continue
    for j in range(i + len(a) - 1, len(toks) - len(b) + 1):
        if toks[j:j + len(b)] == b:
            spans.append((i, j + len(b) - 1)); break
assert spans, f"'{A}' ... '{B}' niet gevonden in {CLIP}"
i, j = spans[-1] if OCC == "last" else spans[int(OCC) - 1]


def quiet(t0, t1):
    f0, f1 = max(0, int(t0 * 100)), min(len(db), int(t1 * 100))
    return (f0 + int(np.argmin(db[f0:f1]))) / 100 if f1 > f0 else (t0 + t1) / 2


# same edge logic as match.py: Whisper's word edges drift, so the real voice onset/offset is measured in the
# loudness (5 frames in a row above noise floor + 18 dB) and the cut goes in the quietest point just outside it
MIN_PAD = 0.04
s, e = W[i]["s"], W[j]["e"]
prev_e = W[i - 1]["e"] if i else 0.0
next_s = W[j + 1]["s"] if j + 1 < len(W) else len(db) / 100
thr = float(np.percentile(db, 10)) + 18
loud = lambda k: k + 5 <= len(db) and (db[k:k + 5] > thr).all()
onset = next((k / 100 for k in range(int(max(prev_e, s - QUIET) * 100), int(e * 100)) if loud(k)), s)
offset = next(((k + 5) / 100 for k in range(int(min(next_s, e + QUIET) * 100) - 5, int(s * 100), -1) if loud(k)), e)
t_in = round(max(0.0, quiet(max(prev_e, onset - QUIET), onset - MIN_PAD)), 3)
t_out = round(min(len(db) / 100, quiet(offset + MIN_PAD, min(next_s, offset + QUIET))), 3)
C = json.load(open(CUTS))
add = SHOT.endswith("+"); SHOT = SHOT.rstrip("+")
if add:
    k = max(q for q, c in enumerate(C["cuts"]) if c["n"] == int(SHOT))
    C["cuts"].insert(k + 1, {"n": int(SHOT), "script": C["cuts"][k].get("script", ""), "part": True})
    targets = [C["cuts"][k + 1]]
else:
    targets = [c for c in C["cuts"] if c["n"] == int(SHOT) and not c.get("part")]
    C["cuts"] = [c for c in C["cuts"] if not (c["n"] == int(SHOT) and c.get("part"))]
for c in targets:
    if True:
        c.update(status="OK", clip=CLIP, **{"in": t_in, "out": t_out}, dur=round(t_out - t_in, 3),
                 words=W[i:j + 1], said=" ".join(w["w"] for w in W[i:j + 1]), dropped_on_set=[],
                 manual=f"recut: '{A}' .. '{B}' (herformuleerd op de set)")
        print(f"shot {SHOT}: {CLIP} {t_in:.2f}-{t_out:.2f} | {c['said']}")
json.dump(C, open(CUTS, "w"), ensure_ascii=False, indent=1)
```

### `qa.py`
```python
"""Read-back check: glue the chosen audio together, re-transcribe it, compare with what each take said.
usage: qa.py CUTS.json AUDIO_DIR OUT.wav [lang=nl] [model=medium]"""
import json, sys, wave
import numpy as np
from difflib import SequenceMatcher
from faster_whisper import WhisperModel
C = json.load(open(sys.argv[1])); ad = sys.argv[2]
parts, marks = [], []
for c in C["cuts"]:
    if "clip" not in c: continue
    wf = wave.open(f"{ad}/{c['clip']}.wav"); a = np.frombuffer(wf.readframes(wf.getnframes()), np.int16)
    seg = a[int(c["in"]*16000):int(c["out"]*16000)]
    marks.append((sum(len(p) for p in parts)/16000, c["n"], c["said"])); parts.append(seg)
x = np.concatenate(parts)
w = wave.open(sys.argv[3], "wb"); w.setnchannels(1); w.setsampwidth(2); w.setframerate(16000); w.writeframes(x.tobytes()); w.close()
LANG = sys.argv[4] if len(sys.argv) > 4 else "nl"
m = WhisperModel(sys.argv[5] if len(sys.argv) > 5 else "medium", device="cpu", compute_type="int8")
segs, _ = m.transcribe(x.astype(np.float32)/32768, language=LANG, word_timestamps=True, condition_on_previous_text=False, beam_size=5, temperature=0)
words = [w for s in segs for w in s.words]
ok = True
for k, (t0, n, exp) in enumerate(marks):
    t1 = marks[k+1][0] if k+1 < len(marks) else len(x)/16000
    got = " ".join(w.word.strip() for w in words if t0 - 0.05 <= (w.start+w.end)/2 < t1)
    # Whisper invents these on silence at the end of audio (training-data subtitles); they are not in the take
    for junk in ("Bedankt voor het kijken!", "Bedankt voor het kijken.", "Bedankt voor het kijken", "Ondertiteld door de Amara.org gemeenschap",
                 "Ondertitels ingediend door de Amara.org gemeenschap", "Tot de volgende keer.", "Dank u wel voor het kijken."):
        got = got.replace(junk, "").strip()
    import re, unicodedata
    nz = lambda t: re.sub(r"[^a-z0-9%]", "", "".join(c for c in unicodedata.normalize("NFKD", t.lower()) if not unicodedata.combining(c)))
    r = SequenceMatcher(None, nz(exp), nz(got), autojunk=False).ratio()   # letters only: punctuation/spacing don't count
    flag = "OK " if r >= 0.9 else "CHECK"
    ok &= r >= 0.9
    print(f"{flag} shot {n} @{t0:6.2f}s  gelijkenis {r:.2f}\n      verwacht: {exp}\n      gehoord : {got}")
print("ALLES OK" if ok else "ER ZIJN SHOTS OM NA TE KIJKEN", f"| totaal {len(x)/16000:.2f}s")
```

### `plan_edit.py`
```python
"""Turn the matched cuts into the edit list: merge shots that run on in the same take, add push-ins / punch-ins.

usage: python3 plan_edit.py CUTS.json PLAN.json VIDEO_ID OUT_edit.json [FRAMESIM.json]
  CUTS.json      from match.py
  FRAMESIM.json  optional, from framesim.py: {"<k>": similarity between end of item k-1 and start of item k}
writes OUT_edit.json: {"items": [{"clip","in","out","s0","s1","shots":[n..],"why"}], "total": s}

Rules (measured on real shoots):
  - two shots from the same clip with < MERGE_GAP s between them play as ONE piece (no jump cut mid-flow)
  - slow push-in on the hook (first item), on items whose script text asks a question, and on the last item;
    amount = min(9%, max(4%, 1.2% per second))
  - punch-in to 112% when an item has the SAME framing as the one before (similarity > 0.57) = a jump cut;
    the item before it then gets no push-in so the punch reads
"""
import json, sys

CUTS, PLAN, VID, OUT = sys.argv[1:5]
SIM = json.load(open(sys.argv[5])) if len(sys.argv) > 5 else {}
MERGE_GAP = 1.0
PUNCH = 1.12
SAME_FRAMING = 0.57   # measured: different setups up to 0.553, same setup from 0.589 (EnergieGenie V5/V6)

C = [c for c in json.load(open(CUTS))["cuts"] if "clip" in c]
plan = json.load(open(PLAN)); video = next(v for v in plan["videos"] if v["id"] == VID)
# client profile "montage": {"push_in": true/false, "punch_in": 1.12}; plan.json "profile" points to it
if plan.get("profile"):
    _m = json.load(open(plan["profile"])).get("montage", {})
    PUNCH = _m.get("punch_in", PUNCH)
    PUSH_IN = _m.get("push_in", True)
else:
    PUSH_IN = True
script = {l["n"]: l["text"] for l in video["lines"]}

items = []
for c in C:
    p = items[-1] if items else None
    if p and p["clip"] == c["clip"] and 0 <= c["in"] - p["out"] < MERGE_GAP:
        p["out"] = c["out"]; p["shots"].append(c["n"]); p["why"].append(f"shot {c['n']} loopt door in dezelfde take")
    else:
        items.append({"clip": c["clip"], "in": c["in"], "out": c["out"], "shots": [c["n"]], "why": []})

push = lambda d: round(1 + min(0.09, max(0.04, 0.012 * d)), 4)
for k, it in enumerate(items):
    d = it["out"] - it["in"]
    it["s0"] = it["s1"] = 1.0
    question = any("?" in script.get(n, "") for n in it["shots"])
    if PUSH_IN and (k == 0 or question or k == len(items) - 1):
        it["s1"] = push(d)
        it["why"].append("push-in: " + ("hook" if k == 0 else "vraag" if question else "slot"))
for k, it in enumerate(items):
    if k and PUNCH > 1.0 and SIM.get(str(k), 0) > SAME_FRAMING:
        it["s0"], it["s1"] = PUNCH, round(PUNCH + (it["s1"] - 1.0 if it["s1"] > 1 else 0.03), 4)
        prev = items[k - 1]
        prev["s1"] = prev["s0"]
        it["why"].append(f"punch-in: zelfde kadrering als vorige ({SIM[str(k)]:.2f})")

t = 0.0
for it in items:
    it["tl"] = round(t, 3); t += it["out"] - it["in"]
json.dump({"video": VID, "total": round(t, 3), "items": items}, open(OUT, "w"), ensure_ascii=False, indent=1)
print(f"{len(items)} stukken, {t:.2f}s")
for it in items:
    print(f"  {it['tl']:6.2f}s {it['clip']} {it['in']:.2f}-{it['out']:.2f} schaal {it['s0']}->{it['s1']} shots {it['shots']} | {'; '.join(it['why'])}")
```

### `hand_marks.py`
```python
"""Find "best take" signals: a hand held over the lens after a take (runs on the Mac, in device_bash).

usage: python3 hand_marks.py CLIPS_DIR OUT_hand.json [ext=MOV]
On set: after a take that was really good, hold a hand fully over the lens for about 2 seconds, then stop recording.
A covered lens = a frame with almost no detail. Measured on 111 real clips (2583 keyframes, EnergieGenie Q4 2026):
the lowest detail in a normal frame was 12.06 (a dark room), median 23.1. Threshold COVERED = 6.0 (half the lowest
normal value). Only keyframes are decoded (iPhone: about 1 per second), so the hand must stay >= 1.5 s.
writes OUT_hand.json: {"IMG_0323": [[t_start, t_end], ...]}  (seconds in the clip)
"""
import glob, json, os, subprocess, sys
import numpy as np

CLIPS, OUT = sys.argv[1], sys.argv[2]
EXT = sys.argv[3] if len(sys.argv) > 3 else "MOV"
COVERED = 6.0
res = json.load(open(OUT)) if os.path.exists(OUT) else {}
for f in sorted(glob.glob(f"{CLIPS}/*.{EXT}")):
    clip = os.path.basename(f).rsplit(".", 1)[0]
    if clip in res:
        continue
    ts = subprocess.run(["ffprobe", "-v", "error", "-skip_frame", "nokey", "-select_streams", "v:0", "-show_entries",
                         "frame=pts_time", "-of", "csv=p=0", f], capture_output=True, text=True).stdout.split()
    ts = [float(t.strip(",")) for t in ts if t.strip(",")]
    b = subprocess.run(["ffmpeg", "-v", "error", "-skip_frame", "nokey", "-i", f, "-vsync", "vfr",
                        "-vf", "scale=90:160,format=gray", "-f", "rawvideo", "-"], capture_output=True).stdout
    a = np.frombuffer(b, np.uint8).reshape(-1, 160, 90).astype(np.float32)
    det = np.abs(np.diff(a, axis=2)).mean((1, 2)) + np.abs(np.diff(a, axis=1)).mean((1, 2))
    n = min(len(ts), len(det)); cov = det[:n] < COVERED
    iv, k = [], 0
    while k < n:
        if not cov[k]:
            k += 1; continue
        j = k
        while j + 1 < n and cov[j + 1]:
            j += 1
        iv.append([round(ts[k], 2), round(ts[j], 2)])
        k = j + 1
    res[clip] = iv
    print(clip, f"min detail {det.min():.2f}", "HAND " + str(iv) if iv else "", flush=True)
    json.dump(res, open(OUT, "w"), indent=0)
print(f"{sum(1 for v in res.values() if v)} van {len(res)} clips met een hand-teken")
```

### `framesim.py`
```python
"""Runs ON THE USER'S COMPUTER (device_bash; needs ffmpeg + numpy).
For every edit item: grab its first and last frame. Writes
  - framesim.json: {"<k>": correlation between the last frame of item k-1 and the first frame of item k}
    (> 0.57 = same framing = jump cut -> punch-in, same threshold as plan_edit.py)
  - with --kop: head_f<k>_<j>.jpg = first frame, one per second, last frame of every item, 360 px wide, for headroom.py
  - sheet.jpg: first|last frame of every item side by side, to LOOK at (empty frame, someone walking in,
    eyes closed, looking away...). Text checks can't see these.

usage: python3 framesim.py EDIT.json CLIPS_DIR OUT_DIR [EXT=MOV] [--kop]     (--kop: also the frames for a feed post)
"""
import json, os, subprocess, sys
from concurrent.futures import ThreadPoolExecutor
import numpy as np

E, CL, OUT = sys.argv[1:4]
KOP = "--kop" in sys.argv
ARGS = [x for x in sys.argv if x != "--kop"]
EXT = ARGS[4] if len(ARGS) > 4 else "MOV"
os.makedirs(OUT, exist_ok=True)
items = json.load(open(E))["items"]


def grab(clip, t, name, w=108, h=192):
    raw = subprocess.run(["ffmpeg", "-v", "error", "-ss", f"{max(0, t):.3f}", "-i", f"{CL}/{clip}.{EXT}", "-frames:v", "1",
                          "-vf", f"scale={w}:{h},format=gray", "-f", "rawvideo", "-"], capture_output=True).stdout
    subprocess.run(["ffmpeg", "-v", "error", "-y", "-ss", f"{max(0, t):.3f}", "-i", f"{CL}/{clip}.{EXT}", "-frames:v", "1",
                    "-vf", "scale=96:-2", f"{OUT}/{name}.jpg"])
    return np.frombuffer(raw, np.uint8).reshape(h, w).astype(float)


def corr(a, b):
    a = (a - a.mean()) / (a.std() + 1e-6); b = (b - b.mean()) / (b.std() + 1e-6)
    return float((a * b).mean())


first, last = [], []
for k, it in enumerate(items):
    first.append(grab(it["clip"], it["in"] + 0.04, f"f{k:02d}a"))
    last.append(grab(it["clip"], it["out"] - 0.07, f"f{k:02d}b"))
# frames for headroom.py (where is the head, for feed posts): first frame, every second, last frame, 360 px wide.
# A speaker moves during a take: first + last frame alone missed up to 2.2% of the height (measured on the export).
# Only with --kop (feed post). 4 at a time: 86 frames for an 80 s video took 95 s one by one (measured).
def head_frame(job):
    clip, t, f = job
    if not os.path.exists(f):
        subprocess.run(["ffmpeg", "-v", "error", "-y", "-ss", f"{max(0, t):.3f}", "-i", f"{CL}/{clip}.{EXT}",
                        "-frames:v", "1", "-vf", "scale=360:-2", f])


if KOP:
    jobs = []
    for k, it in enumerate(items):
        ts = [it["in"] + 0.04] + [it["in"] + j for j in range(1, int(it["out"] - it["in"]))] + [it["out"] - 0.07]
        jobs += [(it["clip"], t, f"{OUT}/head_f{k:02d}_{j:02d}.jpg") for j, t in enumerate(ts)]
    with ThreadPoolExecutor(4) as ex:
        list(ex.map(head_frame, jobs))
    print(f"kop: {len(jobs)} frames voor headroom.py")
sim = {str(k): round(corr(last[k - 1], first[k]), 3) for k in range(1, len(items))}
json.dump(sim, open(f"{OUT}/framesim.json", "w"), indent=1)
ins = sum([["-i", f"{OUT}/f{k:02d}{s}.jpg"] for k in range(len(items)) for s in "ab"], [])
subprocess.run(["ffmpeg", "-v", "error", "-y", *ins, "-filter_complex", f"hstack={2 * len(items)}", f"{OUT}/sheet.jpg"])
for k, v in sim.items():
    print(f"knip {int(k)}->{int(k) + 1}: {v}" + ("  ZELFDE KADRERING" if v > 0.57 else ""))
print("sheet:", f"{OUT}/sheet.jpg")
```

### `headroom.py`
```python
"""Runs in the CLOUD. Where is the head in every edit item? Needed for a feed post (4:5, 1:1): the 9:16 clip is cut
top and bottom there, and build_capcut.py moves the picture down when the head would be cut off.
Input: the head_f<k>_<j>.jpg frames framesim.py writes (first frame, one per second, last frame of each item, 360 px wide).
Face detector: OpenCV YuNet (needs opencv-python >= 4.8 and the model file, see SKILL.md step 5).
Measured on 72 frames of the EnergieGenie shoot: a face in every frame, no false hits; the top of the head lies
0.25 x the face-box height above the box (hair and bald heads, checked by eye).

usage: python3 headroom.py EDIT.json FS_DIR MODEL.onnx    -> FS_DIR/heads.json {"<k>": {"top": .., "chin": .., "frames": n}}
top / chin = fraction of the clip height (0 = top). Frames where the face touches the top edge of the clip itself
(walking in or out, box top < 0.02) are skipped: nothing to frame there. Items without a usable face get no entry.
"""
import glob, json, os, sys
import cv2

E, FS, MODEL = sys.argv[1:4]
HAIR = 0.25          # head top = face-box top - 0.25 x box height (measured)
items = json.load(open(E))["items"]
det = cv2.FaceDetectorYN.create(MODEL, "", (360, 640), 0.7)
out = {}
for k in range(len(items)):
    tops, chins = [], []
    for p in sorted(glob.glob(os.path.join(FS, f"head_f{k:02d}_*.jpg"))):
        im = cv2.imread(p); H, W = im.shape[:2]; det.setInputSize((W, H))
        _, d = det.detect(im)
        if d is None:
            continue
        x, y, w, h = max(d.tolist(), key=lambda r: r[2] * r[3])[:4]   # biggest face = the speaker
        if y / H < 0.02:
            continue
        tops.append((y - HAIR * h) / H); chins.append((y + h) / H)
    if tops:
        out[str(k)] = {"top": round(min(tops), 4), "chin": round(max(chins), 4), "frames": len(tops)}
json.dump(out, open(os.path.join(FS, "heads.json"), "w"), indent=1)
print(f"heads.json: {len(out)} van {len(items)} items met een gezicht")
for k in range(len(items)):
    v = out.get(str(k))
    print(f"  {k:2d} {items[k]['clip']}: " + (f"hoofd vanaf {v['top']:.3f}, kin {v['chin']:.3f} van de cliphoogte" if v else "geen bruikbaar gezicht"))
```

### `render_preview.py`
```python
"""Runs ON THE USER'S COMPUTER (device_bash; needs ffmpeg). Low-res 540x960 preview of an edit list,
with HDR (iPhone HLG / PQ) tone-mapped to SDR and the push-ins/punch-ins rendered.
Renders N items per call so each call stays under the 180 s device_bash limit; call again until it prints DONE.

usage: python3 render_preview.py EDIT.json CLIPS_DIR OUT_DIR OUT.mp4 [EXT=MOV] [PER_CALL=4] [FORMAAT=9:16|4:5|1:1]
FORMAAT 4:5 / 1:1 (feedpost): centre crop of the 9:16 preview, like CapCut fills the feed canvas (540x675 / 540x540).
"""
import json, os, subprocess, sys
from concurrent.futures import ThreadPoolExecutor

E, CL, OD, OUTMP4 = sys.argv[1:5]
EXT = sys.argv[5] if len(sys.argv) > 5 else "MOV"
PER = int(sys.argv[6]) if len(sys.argv) > 6 else 4
CROP = {"9:16": "", "4:5": ",crop=540:674", "1:1": ",crop=540:540"}[sys.argv[7] if len(sys.argv) > 7 else "9:16"]
os.makedirs(OD, exist_ok=True)
items = json.load(open(E))["items"]


def transfer(path):
    r = subprocess.run(["ffprobe", "-v", "error", "-select_streams", "v:0", "-show_entries", "stream=color_transfer",
                        "-of", "csv=p=0", path], capture_output=True, text=True).stdout.strip()
    return r


def render(k):
    it = items[k]; src = f"{CL}/{it['clip']}.{EXT}"; out = f"{OD}/p{k:02d}.mp4"
    if os.path.exists(out):
        return out
    dur = it["out"] - it["in"]; n = max(2, round(dur * 30))
    tr = transfer(src)
    if tr in ("arib-std-b67", "smpte2084"):
        base = (f"zscale=w=1080:h=1920:t=linear:npl=203:tin={tr}:pin=bt2020:min=bt2020nc,format=gbrpf32le,"
                "zscale=p=bt709,tonemap=hable:desat=0,zscale=t=bt709:m=bt709:r=tv,format=yuv420p")
    else:
        base = "scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920,format=yuv420p"
    s0, s1 = it["s0"], it["s1"]
    vf = (f"{base},fps=30,zoompan=z='{s0}+({s1}-{s0})*on/({n}-1)':d=1:x='(iw-iw/zoom)/2':y='(ih-ih/zoom)*0.35'"
          f":s=540x960:fps=30{CROP}")
    tmp = out + ".part.mp4"
    subprocess.run(["ffmpeg", "-v", "error", "-y", "-ss", f"{it['in']:.3f}", "-i", src, "-t", f"{dur:.3f}", "-vf", vf,
                    "-frames:v", str(n), "-c:v", "libx264", "-preset", "ultrafast", "-crf", "22", "-c:a", "aac",
                    "-ar", "48000", "-ac", "2", tmp], check=True)
    os.replace(tmp, out)
    return out


todo = [k for k in range(len(items)) if not os.path.exists(f"{OD}/p{k:02d}.mp4")][:PER]
with ThreadPoolExecutor(PER) as ex:
    list(ex.map(render, todo))
left = [k for k in range(len(items)) if not os.path.exists(f"{OD}/p{k:02d}.mp4")]
if left:
    print(f"nog {len(left)} te renderen: roep opnieuw aan")
else:
    open(f"{OD}/list.txt", "w").write("".join(f"file 'p{k:02d}.mp4'\n" for k in range(len(items))))
    subprocess.run(["ffmpeg", "-v", "error", "-y", "-f", "concat", "-safe", "0", "-i", f"{OD}/list.txt", "-c", "copy", OUTMP4],
                   check=True)
    d = subprocess.run(["ffprobe", "-v", "error", "-show_entries", "format=duration", "-of", "csv=p=0", OUTMP4],
                       capture_output=True, text=True).stdout.strip()
    print(f"DONE {OUTMP4} {d}s (verwacht {json.load(open(E))['total']}s)")
```

### `harvest_caption_style.py`
```python
"""Harvest a CapCut caption STYLE (word-by-word highlight template) from a draft the user styled in CapCut.

usage: python3 harvest_caption_style.py DRAFT_INFO.json TEMPLATE_RESOURCE_ID OUT_style.json [--color #FAFF96] [--font /path/font.otf]
  Finds the first caption segment made from text_template TEMPLATE_RESOURCE_ID and stores: its track attrs, one
  segment, the text_template material, the inner text material, and every referenced material (effects,
  material_animations), so build_capcut.py can clone them for every subtitle block.
  --color  text color. For the word-highlight templates measured so far ("逐词高亮-发光黄", resource 7543210783027547409):
           white text -> active word pure yellow #FFFF00; any OTHER text color -> active word gets that color and the
           rest is shown white. So --color #FAFF96 = active word in that yellow, other words white.
  --background  box color. Box templates ("边框弹出Ⅰ", resource 7535406282551971088) draw the box behind the active word
           in the text's BACKGROUND color when the background is on (background_style 1 + check_flag bit 16), else purple.
  --stroke  outline "#RRGGBB:width" (e.g. "#221334:0.08") so white text stays readable on the light box and on video.
  --font   font file CapCut uses (CapCut copies system fonts into ~/Library/Containers/com.lemon.lvoverseas/Data/Library/Fonts/)
"""
import json, sys

src, rid, out = sys.argv[1:4]
args = sys.argv[4:]
color = args[args.index("--color") + 1] if "--color" in args else None
font = args[args.index("--font") + 1] if "--font" in args else None
background = args[args.index("--background") + 1] if "--background" in args else None   # box color for box-highlight templates
stroke = args[args.index("--stroke") + 1] if "--stroke" in args else None               # "#221334:0.08" = color:width
d = json.load(open(src))
M = {o["id"]: (k, o) for k, v in d["materials"].items() if isinstance(v, list) for o in v if isinstance(o, dict) and "id" in o}
for t in d["tracks"]:
    s = t["segments"][0]
    k, m = M[s["material_id"]]
    if k == "text_templates" and m.get("resource_id") == rid:
        break
else:
    raise SystemExit(f"geen caption met template {rid} gevonden")
tir = m["text_info_resources"][0]
inner = M[tir["text_material_id"]][1]
refs = {r: list(M[r]) for r in s["extra_material_refs"] + tir["extra_material_refs"]}
style = {"template_resource_id": rid, "template_name": m.get("name"), "track": {k: v for k, v in t.items() if k != "segments"},
         "segment": s, "template": m, "inner_text": inner, "refs": refs, "color": color, "font": font,
         "background": background, "stroke": stroke}
json.dump(style, open(out, "w"), ensure_ascii=False, indent=1)
print(f"stijl '{m.get('name')}' bewaard: {len(refs)} gekoppelde materialen, kleur {color}, font {font}")
```

### `build_capcut.py`
```python
"""Build a NEW CapCut project folder from an approved edit list, cloning every object from one of the
user's OWN CapCut projects (so every field is one their CapCut version wrote itself).

usage: python3 build_capcut.py JOB.json OUT_DIR
JOB.json:
  {"name": "Project name shown in CapCut",
   "projects_root": "/Users/<user>/Movies/CapCut/User Data/Projects/com.lveditor.draft",   # real path on the Mac
   "template_dir": "optional: another template project (draft_info.json + draft_meta_info.json); default sjabloon/",
   "edit": "edit.json (plan_edit.py)", "words": "words.json (transcribe.py)", "plan": "plan.json", "video": "t1",
   "clips": {"IMG_0317": {"dur": 22.1, "w": 2160, "h": 3840, "ext": "MOV"}, ...},   # ffprobe-measured, display w/h
   "fps": 30, "sub_max": 42,
   "caption_style": "style.json (harvest_caption_style.py), optional: word-by-word highlight captions",
   "profile": "klanten/<klant>/profiel.json (optional): fills caption_style, caption_y, caption_max_width, sub_max,
               letters_per_unit, colours, fps and fixed corrections; a value in the job itself always wins",
   "font_path": "font file in CapCut's own font folder on THIS Mac (resolved per Mac)",
   "platform": "tiktok | reel | story | website | feed45 | feed11 (default tiktok): sets canvas, caption position and
               width, see PLATFORMS",
  "heads": "fs/heads.json (headroom.py), needed for feed45/feed11: moves the picture down where the head would be cut",
  "kopruimte": 0.07}   # feed: minimum space above the head, fraction of the frame height (profile montage.kopruimte or 0.07)
Media are referenced INSIDE the project folder (video/<clip>.<ext>) through CapCut's draft-path placeholder,
because the sandboxed Mac CapCut can't read files elsewhere (e.g. Downloads) -> copy the clips there on install.
Writes OUT_DIR/{draft_info.json,.bak,template-2.tmp, Timelines/project.json, Timelines/<id>/(3 copies),
draft_meta_info.json, draft_settings, build_report.json}.
"""
import copy, hashlib, json, os, re, sys, time, uuid, unicodedata
from difflib import SequenceMatcher
import numpy as np

JOB = json.load(open(sys.argv[1])); OUT = sys.argv[2]
# platform presets for the caption position (CapCut transform y: -1 = bottom edge, +1 = top edge) and max width.
# Measured on our TikTok exports: the caption block is TOP-anchored, y -0.349 -> block from 1191 px down to at most
# 1438 px (of 1920). So block top = 960 - y*960 - 104 px, block height up to 247 px.
#   tiktok : bottom 350-480 px covered (measured on exports)      -> y -0.349, 482-522 px free below the block
#   story  : about 250 px covered at the bottom (inro.social)     -> y -0.349 fits
#   reel   : about 670 px covered at the bottom (hopperhq.com)    -> y -0.15: top 1000 px, bottom <= 1247 px
#            (computed from the measured anchor, measure on the first real Reel export)
#   website: nothing covered                                      -> y -0.349
#   Measured on 9:16, 4:5 and 1:1 exports (2-11-2026): y counts in half canvas heights, caption size follows the canvas
#   WIDTH (identical px width in all three), so block top = H/2*(1-y) - a*(W/1080), a = 35..104 px depending on the line.
#   feed45 / feed11: Instagram feed post 4:5 or 1:1. Canvas as CapCut writes it itself (measured: Custom 1080x1350 ->
#            {"ratio": "custom", 1080, 1350}; preset 1:1 -> {"ratio": "1:1", 1920, 1920}). CapCut fits a clip INSIDE the
#            canvas at scale 1 (measured: pillarbox), so the clips get a fill factor (see FILL below).
PLATFORMS = {"tiktok": {"caption_y": -0.349, "caption_max_width": 0.77},
             "story": {"caption_y": -0.349, "caption_max_width": 0.77},
             "reel": {"caption_y": -0.15, "caption_max_width": 0.77},
             "website": {"caption_y": -0.349, "caption_max_width": 0.77},
             "feed45": {"caption_y": -0.349, "caption_max_width": 0.77},
             "feed11": {"caption_y": -0.349, "caption_max_width": 0.77}}
CANVAS = {"feed45": {"ratio": "custom", "width": 1080, "height": 1350},
          "feed11": {"ratio": "1:1", "width": 1920, "height": 1920}}
PLATFORM = JOB.get("platform", "tiktok").lower()
PLATFORM = {"feed": "feed45", "feedpost": "feed45", "post": "feed45", "4:5": "feed45", "1:1": "feed11",
            "square": "feed11", "vierkant": "feed11"}.get(PLATFORM, PLATFORM)
if PLATFORM not in PLATFORMS:
    raise SystemExit(f"STOP: onbekend platform '{PLATFORM}'. Kies uit: {', '.join(PLATFORMS)}")
# client profile (klanten/<klant>/profiel.json): house style + fixed corrections per client. Job values win.
PROFILE_SUBS = []
if JOB.get("profile"):
    _pp = JOB["profile"]; _pd = os.path.dirname(os.path.abspath(_pp)); P = json.load(open(_pp))
    _o = P.get("ondertitels", {})
    for _k, _v in (("caption_style", os.path.join(_pd, _o["style"]) if _o.get("style") else None),
                   ("sub_max", _o.get("max_tekens")), ("letters_per_unit", _o.get("letters_per_eenheid")),
                   ("caption_scale", _o.get("schaal")), ("caption_background", _o.get("blok_kleur")),
                   ("caption_stroke", _o.get("rand")), ("caption_color", _o.get("tekst_kleur")),
                   ("fps", P.get("fps")), ("kopruimte", P.get("montage", {}).get("kopruimte"))):
        if _v is not None and _k not in JOB:
            JOB[_k] = _v
    PROFILE_SUBS = P.get("vaste_correcties", [])
    for _k, _v in P.get("platformen", {}).get(PLATFORM, {}).items():   # client-specific platform override
        JOB.setdefault({"y": "caption_y", "max_breedte": "caption_max_width"}.get(_k, _k), _v)
for _k, _v in PLATFORMS[PLATFORM].items():
    JOB.setdefault(_k, _v)
print(f"platform {PLATFORM}: caption_y {JOB['caption_y']}, max breedte {JOB['caption_max_width']}")
NAME, ROOT = JOB["name"], JOB["projects_root"]
# template: the one built into the skill (sjabloon/, stripped from the approved EnergieGenie project: iPhone HDR clip,
# HDR mode 4, noise reduction 85%) unless the job names another. Other projects carry other settings (measured on this
# Mac: HDR mode 1, no noise reduction, muted clips), so do not pick a random project of the user.
TPL = JOB.get("template_dir") or os.path.join(os.path.dirname(os.path.dirname(os.path.abspath(__file__))), "sjabloon")
_m_home = re.match(r"(/Users/[^/]+)/", ROOT)
_tpl_load = lambda n: open(f"{TPL}/{n}", encoding="utf-8").read().replace("/Users/<user>/", (_m_home.group(1) if _m_home else "/Users/<user>") + "/")
WORDS, PLAN, VID = JOB["words"], JOB["plan"], JOB["video"]
FPS = JOB.get("fps", 30)
HEADS = json.load(open(JOB["heads"])) if JOB.get("heads") else {}
HEADROOM = JOB.get("kopruimte", 0.07)   # 7%: head detection varies up to 2.2% of the height on the export (measured)
CHIN_MAX = 0.60      # feed: caption block starts at 61-62% of the height (measured on the 4:5 and 1:1 exports)
if PLATFORM in CANVAS and not HEADS:
    print("LET OP: feedpost zonder heads.json (headroom.py): het beeld wordt in het midden bijgesneden, hoofden kunnen "
          "afgesneden worden. Draai framesim.py + headroom.py en geef \"heads\" mee.")
REFRAME = []
SUB_MAX = JOB.get("sub_max", 42)
CLIPS = JOB["clips"]
EDIT = [(it["clip"], it["in"], it["out"], it["s0"], it["s1"]) for it in json.load(open(JOB["edit"]))["items"]]
_tpl_raw = _tpl_load("draft_info.json")
_ph = re.findall(r"##_draftpath_placeholder_[0-9A-F-]+_##", _tpl_raw)
CLIPDIR = (_ph[0] if _ph else "##_draftpath_placeholder_0E685133-18CE-45ED-8CB8-2904A212EC80_##") + "/video"

U = lambda: str(uuid.uuid4()).upper()
fr = lambda t: int(round(t * FPS))
us = lambda f: int(f * 1000000 // FPS)

D0 = json.loads(_tpl_raw)
META = json.loads(_tpl_load("draft_meta_info.json"))
ALL = {o["id"]: (k, o) for k, v in D0["materials"].items() if isinstance(v, list) for o in v if isinstance(o, dict) and "id" in o}
T_VSEG = D0["tracks"][0]["segments"][0]
T_VMAT = ALL[T_VSEG["material_id"]][1]
# plain subtitle template (only needed when no caption_style is given)
ttrack = next((t for t in D0["tracks"] if t["type"] == "text" and t["segments"]
               and ALL[t["segments"][0]["material_id"]][1].get("type") == "subtitle"), None)
if ttrack is None and not JOB.get("caption_style"):
    raise SystemExit("template project has no subtitle track: pick another template or give a caption_style")
T_TSEG = ttrack["segments"][0] if ttrack else None
T_TTRACK = {k: v for k, v in ttrack.items() if k != "segments"} if ttrack else None
T_TMAT = ALL[T_TSEG["material_id"]][1] if ttrack else None

M = {k: ([] if isinstance(v, list) else copy.deepcopy(v)) for k, v in D0["materials"].items()}


def add(kind, tmpl, **kw):
    o = copy.deepcopy(tmpl); o["id"] = U(); o.update(kw); M.setdefault(kind, []).append(o); return o["id"]


def clone_refs(seg):
    out = []
    for r in seg["extra_material_refs"]:
        kind, o = ALL[r]
        kw = {"animations": []} if kind == "material_animations" else ({"speed": 1.0} if kind == "speeds" else {})
        out.append(add(kind, o, **kw))
    return out


def kf(prop, pts):  # exact structure CapCut wrote in the user's own project (KFTypeGlobalAlpha), other property
    return {"id": U(), "material_id": "", "property_type": prop,
            "keyframe_list": [{"id": U(), "curveType": "Line", "time_offset": int(t), "left_control": {"x": 0.0, "y": 0.0},
                               "right_control": {"x": 0.0, "y": 0.0}, "values": [float(v)], "string_value": "", "graphID": ""}
                              for t, v in pts]}


# ---------- 1 video track
vtrack = {"id": U(), "type": "video", "flag": 0, "attribute": 0, "name": "", "is_default_name": True, "segments": []}
tl = 0
placed = []
for idx, (clip, a, b, s0, s1) in enumerate(EDIT):
    f0, f1 = fr(a), fr(b)
    src_us, dur_us = us(f0), us(tl + f1 - f0) - us(tl)   # timeline-exact length -> no 1 us gaps between clips
    file_us = int(round(CLIPS[clip]["dur"] * 1e6))
    ext = CLIPS[clip].get("ext", "MOV")
    assert src_us + dur_us <= file_us, (clip, "runs past the end of the file")
    mid = add("videos", T_VMAT, path=f"{CLIPDIR}/{clip}.{ext}", material_name=f"{clip}.{ext}",
              width=CLIPS[clip].get("w", 2160), height=CLIPS[clip].get("h", 3840),
              duration=file_us, has_audio=True, unique_id="", local_id="", media_path="", local_material_id="")
    s = copy.deepcopy(T_VSEG)
    s.update({"id": U(), "material_id": mid, "extra_material_refs": clone_refs(T_VSEG),
              "source_timerange": {"start": src_us, "duration": dur_us},
              "target_timerange": {"start": us(tl), "duration": us(tl + f1 - f0) - us(tl)},
              "speed": 1.0, "volume": 1.0, "last_nonzero_volume": 1.0, "keyframe_refs": [], "render_index": 0, "track_render_index": 0})
    if PLATFORM in CANVAS:   # fill the canvas: CapCut shows a clip at scale 1 fitted INSIDE the canvas (measured)
        _cw, _ch = CANVAS[PLATFORM]["width"], CANVAS[PLATFORM]["height"]
        _w, _h = CLIPS[clip].get("w", 2160), CLIPS[clip].get("h", 3840)
        _fill = max(_cw / _w, _ch / _h) / min(_cw / _w, _ch / _h)
        s0, s1 = round(s0 * _fill, 4), round(s1 * _fill, 4)
        # headroom: picture height on the canvas at scale 1 = fit_h; CapCut scales around the clip centre, so a point at
        # v (0 = top of the clip) lands at  H/2 + shift + (v - 0.5) * fit_h * scale.  Shift down (px) only as far as needed
        # to keep the head top >= kopruimte * H at the largest scale, and never so far that the top edge of the clip
        # comes into view at the smallest scale (room = (fit_h * scale_min - H) / 2).
        _fit_h = min(_cw / _w, _ch / _h) * _h
        _hd = HEADS.get(str(idx)); _shift = 0.0
        if _hd:
            _need = HEADROOM * _ch - _ch / 2 - (_hd["top"] - 0.5) * _fit_h * max(s0, s1)      # min shift: headroom
            _lift = CHIN_MAX * _ch - _ch / 2 - (_hd["chin"] - 0.5) * _fit_h * max(s0, s1)     # max shift: chin above captions
            _room = (_fit_h * min(s0, s1) - _ch) / 2
            _shift = max(_need, min(0.0, _lift))          # down for the head; up for a low chin, never past the headroom
            _shift = max(-_room, min(_shift, _room))
            _top = (_ch / 2 + _shift + (_hd["top"] - 0.5) * _fit_h * max(s0, s1)) / _ch
            _chin = (_ch / 2 + _shift + (_hd["chin"] - 0.5) * _fit_h * max(s0, s1)) / _ch
            REFRAME.append((idx, clip, round(_shift / _ch * 1080), round(_top, 3), round(_chin, 3), _need > _room + 0.5))
        _ty = round(-_shift / (_ch / 2), 4) + 0.0   # no -0.0; CapCut transform y: half canvas heights, + = up (measured on export)
    _ty = _ty if PLATFORM in CANVAS else 0.0
    s["clip"] = {"scale": {"x": s0, "y": s0}, "rotation": 0.0, "transform": {"x": 0.0, "y": _ty},
                 "flip": {"vertical": False, "horizontal": False}, "alpha": 1.0}
    s["uniform_scale"] = {"on": True, "value": 1.0}
    if abs(s1 - s0) > 1e-4:   # keyframe time = SOURCE time (clip source start + offset); last key 1 frame before the end
        k0, k1 = src_us, src_us + dur_us - us(1)
        s["common_keyframes"] = [kf("KFTypePositionX", [(k0, 0), (k1, 0)]), kf("KFTypePositionY", [(k0, _ty), (k1, _ty)]),
                                 kf("KFTypeScaleX", [(k0, s0), (k1, s1)]), kf("KFTypeRotation", [(k0, 0), (k1, 0)])]
    else:
        s["common_keyframes"] = []
    vtrack["segments"].append(s)
    placed.append((clip, a, b, us(tl) / 1e6))
    tl += f1 - f0
TOTAL = us(tl)

# ---------- 2 subtitles: spoken words (timing) with script spelling where it is clearly the same word
plan = json.load(open(PLAN)); video = next(v for v in plan["videos"] if v["id"] == VID)
script_words = " ".join(l["text"] for l in video["lines"]).split()
_nz0 = lambda w: re.sub(r"[^a-z0-9%]", "", "".join(c for c in unicodedata.normalize("NFKD", w.lower()) if not unicodedata.combining(c)))
NUMW = dict(enumerate("nul een twee drie vier vijf zes zeven acht negen tien elf twaalf".split()))
nz = lambda w: NUMW.get(int(_nz0(w)), _nz0(w)) if _nz0(w).isdigit() and int(_nz0(w)) in NUMW else _nz0(w)   # "3." == "Drie:"
W = json.load(open(WORDS))
DB = {}
voiced = lambda seg, thr: any((seg[k:k + 5] > thr).all() for k in range(len(seg) - 4))   # 5 frames in a row = voice
for _c in {p[0] for p in placed}:
    _f = os.path.join(os.path.dirname(WORDS), f"{_c}_db.npy")
    if os.path.exists(_f):
        _d = np.load(_f); DB[_c] = (_d, float(np.percentile(_d, 10)) + 18)
spoken = []
for clip, a, b, tl0 in placed:
    for w in W[clip]["words"]:
        # a word belongs to the cut when it overlaps it (Whisper can smear a first word over the silence before it)
        inside = w["e"] > a + 0.05 and w["s"] < b - 0.05
        if not inside and clip in DB and w.get("p", 1) >= 0.5:   # low-confidence words are often invented on silence
            # Whisper sometimes places a whole word just OUTSIDE the cut ("Op" at 13.04-13.60, voice really at 13.90);
            # if the audio between that word and the cut edge is silent, the word is in fact spoken inside the cut
            d, thr = DB[clip]
            if a - 0.8 < w["e"] <= a + 0.05:
                inside = not voiced(d[int(w["s"] * 100):int(a * 100) + 1], thr)
            elif b - 0.05 <= w["s"] < b + 0.8:
                inside = not voiced(d[int(b * 100):int(w["e"] * 100) + 1], thr)
        if inside:
            ws_, we_ = max(a, min(w["s"], b - 0.2)), min(b, max(w["e"], a + 0.2))   # pulled-in words get >= 0.2 s
            ww = {"w": w["w"], "s": ws_ - a + tl0, "e": we_ - a + tl0}
            if ww["w"].startswith("-") and spoken and spoken[-1]["e"] >= ww["s"] - 0.05:   # "QR -code" -> "QR-code"
                spoken[-1]["w"] += ww["w"]; spoken[-1]["e"] = ww["e"]
            else:
                spoken.append(ww)
sm = SequenceMatcher(None, [nz(x) for x in script_words], [nz(x["w"]) for x in spoken], autojunk=False)
fixes, out = [], []
for op, i1, i2, j1, j2 in sm.get_opcodes():
    seg = spoken[j1:j2]
    if op == "equal":
        out += [dict(w, w=script_words[i], si=i, sp=w["w"]) for i, w in zip(range(i1, i2), seg)]
    elif op == "replace" and "".join(nz(x) for x in script_words[i1:i2]) == "".join(nz(w["w"]) for w in seg):
        # same letters, different word split ("vanaf" / "van af", "West -Europa" / "West-Europa"): script words,
        # time span shared out evenly
        s0, e0, n = seg[0]["s"], seg[-1]["e"], i2 - i1
        out += [{"w": script_words[i], "s": s0 + (e0 - s0) * k / n, "e": s0 + (e0 - s0) * (k + 1) / n}
                for k, i in enumerate(range(i1, i2))]
        fixes.append(f"{' '.join(w['w'] for w in seg)} -> {' '.join(script_words[i1:i2])}")
    elif op == "replace" and i2 - i1 == j2 - j1:
        for i, w in zip(range(i1, i2), seg):
            a_, b_ = nz(script_words[i]), nz(w["w"])
            if SequenceMatcher(None, a_, b_).ratio() >= 0.8 and not (a_ != b_ and (a_.startswith(b_) or b_.startswith(a_))):
                # Whisper typo of the same word (but "voorschot" vs "voorschoten" is a different word form: keep spoken)
                if nz(script_words[i]) != nz(w["w"]):
                    fixes.append(f"{w['w']} -> {script_words[i]}")
                out.append(dict(w, w=script_words[i]))
            else:
                out.append(w)                                                          # really said differently: keep
    elif op in ("replace", "insert"):
        out += seg                                                                     # said, not in script: keep
# script punctuation only where the next spoken word is also the next script word; otherwise the speaker went another
# way ("vertrouwd. aan." / swapped clauses) and the spoken punctuation is the right one
PUN = re.compile(r"[.,:;?!]+$")
for k, w in enumerate(out):
    if "si" in w and k + 1 < len(out) and out[k + 1].get("si") != w["si"] + 1:
        core = PUN.sub("", w["w"]); a_p = PUN.search(w["w"]); sp_p = PUN.search(w["sp"])
        a_p, sp_p = a_p.group(0) if a_p else "", sp_p.group(0) if sp_p else ""
        if not (a_p in ("", ",") and sp_p in ("", ",")):     # commas from the script always stay
            w["w"] = core + sp_p
# per-project corrections of what Whisper misheard, measured on the audio: job "sub_phrases": [["from words", "to words"]]
for src, dst in [tuple(x) for x in JOB.get("sub_phrases", [])] + [tuple(x) for x in PROFILE_SUBS]:   # job first: it wins
    sw, dw = src.split(), dst.split()
    for k in range(len(out) - len(sw) + 1):
        if [nz(x["w"]) for x in out[k:k + len(sw)]] == [nz(x) for x in sw]:
            s0, e0 = out[k]["s"], out[k + len(sw) - 1]["e"]
            _p = PUN.search(out[k + len(sw) - 1]["w"])            # keep the spoken punctuation ("fluvjes." -> "Fluvius.")
            if _p and not PUN.search(dw[-1]):
                dw = dw[:-1] + [dw[-1] + _p.group(0)]
            out[k:k + len(sw)] = [{"w": d, "s": s0 + (e0 - s0) * q / len(dw), "e": s0 + (e0 - s0) * (q + 1) / len(dw)}
                                  for q, d in enumerate(dw)]
            fixes.append(f"{src} -> {dst}")
            break
    else:
        if [src, dst] not in [list(x) for x in PROFILE_SUBS]:     # profile corrections are optional per video
            print("LET OP: sub_phrase niet gevonden:", src)
# capitals: a sentence starts with a capital; a word the script only capitalised because it started a script sentence
# is lowercased mid-sentence ("en Zo sluit" -> "en zo sluit")
for k, w in enumerate(out):
    start = k == 0 or out[k - 1]["w"].endswith((".", "?", "!"))
    if start:
        w["w"] = w["w"][:1].upper() + w["w"][1:]
    elif "si" in w and w["w"][:1].isupper() and (w["si"] == 0 or script_words[w["si"] - 1].endswith((".", "?", "!"))):
        w["w"] = w["w"][:1].lower() + w["w"][1:]
spoken = out

# sentences -> blocks of <= SUB_MAX chars, split into EVEN parts (no orphan words), preferring a comma near the target
sents, cur = [], []
for w in spoken:
    cur.append(w)
    if w["w"].endswith((".", "?", "!")):
        sents.append(cur); cur = []
if cur:
    sents.append(cur)
L = lambda ws: len(" ".join(x["w"] for x in ws))
def best_split(snt):
    """Optimal subtitle split of one sentence: parts <= SUB_MAX chars, as even as possible,
    a bonus for breaking after a comma/colon, a big penalty for breaking inside a name ("Verenigd | Koninkrijk")."""
    n = len(snt)
    cost = {0: (0.0, None)}
    for j in range(1, n + 1):
        for i in range(j):
            if i not in cost:
                continue
            part = snt[i:j]; ln = L(part)
            if ln > SUB_MAX and j - i > 1:
                continue
            c = cost[i][0] + (SUB_MAX - ln) ** 2 / 10 + 25          # short parts and many parts cost
            if j < n:
                w, nxt = snt[j - 1]["w"], snt[j]["w"]
                if w.endswith((",", ":")):
                    c -= 30
                if w[:1].isupper() and nxt[:1].isupper() and not w.endswith((",", ":")):
                    c += 500
            if j not in cost or c < cost[j][0]:
                cost[j] = (c, i)
    cuts, j = [], n
    while j:
        i = cost[j][1]; cuts.append(snt[i:j]); j = i
    return cuts[::-1]


blocks = []
for snt in sents:
    blocks += best_split(snt)

STYLE = json.load(open(JOB["caption_style"])) if JOB.get("caption_style") else None
if STYLE:
    # the harvested style holds CapCut cache paths of the Mac it was made on (effects, animations, font):
    # point them to THIS user's CapCut container (home taken from projects_root). CapCut must have used the caption
    # template once on this Mac so the effect files exist there (SKILL.md, step 8).
    _home = re.match(r"(/Users/[^/]+)/", ROOT)
    if _home:
        STYLE = json.loads(re.sub(r"/Users/[^/\"]+/Library/Containers/com\.lemon\.lvoverseas",
                                  _home.group(1) + "/Library/Containers/com.lemon.lvoverseas", json.dumps(STYLE)))
    # per-Mac font path (CapCut copies fonts to its own folder with a hash: resolve it on the Mac, see SKILL.md)
    if JOB.get("font_path"):
        STYLE["font"] = JOB["font_path"]
    for _k, _s in (("caption_background", "background"), ("caption_stroke", "stroke"), ("caption_color", "color")):
        if JOB.get(_k):
            STYLE[_s] = JOB[_k]
    if JOB.get("caption_scale"):
        STYLE["segment"]["clip"]["scale"] = {"x": JOB["caption_scale"], "y": JOB["caption_scale"]}
    if not JOB.get("font_path"):
        raise SystemExit("STOP: geen font_path in de job. Zoek het lettertype van de klant op in CapCut's fontmap op deze "
                         "Mac (SKILL.md stap 8) en gebruik voor publicatie de gelicentieerde versie, geen Trial.")
    if "trial" in os.path.basename(JOB["font_path"]).lower():
        print("LET OP: font_path is een Trial-lettertype; niet publiceren zonder de gelicentieerde versie")


def rgb(hexcol):
    h = hexcol.lstrip("#")[:6]
    return [int(h[i:i + 2], 16) / 255 for i in (0, 2, 4)]


def styled_caption(text, bl, a_f, b_f):
    """One caption from a harvested CapCut caption template (word-by-word highlight).
    words: CapCut format, ms relative to the caption start, spaces as zero-length tokens at the end of the previous word."""
    st = STYLE
    t0 = us(a_f) / 1e6
    dur_us = us(b_f) - us(a_f)
    toks = {"start_time": [], "end_time": [], "text": []}
    for i, w in enumerate(bl):
        s_ms = max(0, int(round((w["s"] - t0) * 1000))); e_ms = max(s_ms, int(round((w["e"] - t0) * 1000)))
        toks["start_time"].append(s_ms); toks["end_time"].append(e_ms); toks["text"].append(w["w"])
        if i + 1 < len(bl):
            toks["start_time"].append(e_ms); toks["end_time"].append(e_ms); toks["text"].append(" ")
    # shared refs (effects + caption animation) are referenced by both the segment and the inner text
    newref = {}
    for rid_old, (kind, o) in st["refs"].items():
        kw = {}
        if kind == "material_animations":
            anims = copy.deepcopy(o["animations"])
            for an in anims:
                an["start"], an["duration"] = 0, dur_us
            kw["animations"] = anims
        newref[rid_old] = add(kind, o, **kw)
    content = json.loads(st["inner_text"]["content"])
    content["text"] = text
    for sty in content["styles"]:
        sty["range"] = [0, len(text)]
        if st.get("font"):
            sty["font"] = {"path": st["font"], "id": ""}
        if st.get("color"):
            sty["fill"] = {"content": {"solid": {"color": rgb(st["color"])}, "render_type": "solid"}}
        if st.get("stroke"):
            sc, sw = st["stroke"].split(":")
            sty["strokes"] = [{"alpha": 1.0, "content": {"render_type": "solid", "solid": {"alpha": 1.0, "color": rgb(sc)}},
                               "width": float(sw)}]
    tkw = dict(content=json.dumps(content, ensure_ascii=False), words=toks,
               current_words={"start_time": [], "end_time": [], "text": []}, recognize_task_id="", group_id=GROUP)
    if st.get("color"):
        tkw["text_color"] = st["color"]
    if st.get("font"):
        tkw.update(font_path=st["font"], font_id="", font_resource_id="")
    if st.get("background"):   # measured: CapCut's Background toggle = these three fields + check_flag bit 16
        tkw.update(background_color=st["background"], background_style=1, background_alpha=1.0,
                   check_flag=st["inner_text"].get("check_flag", 47) | 16)
    if st.get("stroke"):
        sc, sw = st["stroke"].split(":")
        tkw.update(border_color=sc, border_width=float(sw), border_alpha=1.0)
    # long words: the caption template wraps INSIDE a word when it does not fit a line ("ELEKTRICITEITSMA / RKT").
    # Measured in CapCut (FT Aktual Bold, upper case): line_max_width is in the text's own units, so it scales WITH
    # the caption. 16 capitals fill line_max_width 0.82 (19.5 per unit; 18.3 used, with margin), and on screen
    # line_max_width x scale x 0.976 = fraction of the frame width (0.82 x 0.69 -> 0.55, measured).
    # So: widen the line until the longest word fits; shrink the caption only when it would pass caption_max_width.
    lmw0 = st["inner_text"].get("line_max_width", 0.82); sc0 = st["segment"]["clip"]["scale"]["x"]
    L = max(len(w["w"]) for w in bl) + 1
    lmw = max(lmw0, round(L / JOB.get("letters_per_unit", 18.3), 4))
    # max width on screen: job "caption_max_width" (default 0.77 = 120 px free left and right on 1080, TikTok buttons);
    # measured: on-screen width = line_max_width x scale x 0.976
    shrink = min(1.0, JOB.get("caption_max_width", 0.77) / (lmw * sc0 * 0.976)) if sc0 else 1.0
    if lmw != lmw0:
        tkw["line_max_width"] = round(lmw, 4)
    inner_id = add("texts", st["inner_text"], **tkw)
    tpl = copy.deepcopy(st["template"])
    tir = tpl["text_info_resources"][0]
    tir.update(id=U(), text_material_id=inner_id, extra_material_refs=[newref[r] for r in tir["extra_material_refs"]])
    tir["attach_info"].update(start_time=0, duration=dur_us)
    tpl["text_info_resources"] = [tir]
    tpl_id = add("text_templates", tpl)
    seg = copy.deepcopy(st["segment"])
    seg.update({"id": U(), "material_id": tpl_id, "extra_material_refs": [newref[r] for r in seg["extra_material_refs"]],
                "target_timerange": {"start": us(a_f), "duration": dur_us}, "keyframe_refs": [], "common_keyframes": []})
    if shrink < 1.0:
        seg["clip"]["scale"] = {"x": round(sc0 * shrink, 4), "y": round(sc0 * shrink, 4)}
    # position: default for 9:16 social (TikTok/Reels/Shorts). CapCut: -1 = bottom edge, +1 = top edge.
    # -0.349 puts the caption block at 62%-75% of the height (measured on export: 1191-1438 px of 1920), clear of the
    # ~480 px bottom zone where TikTok shows description, hashtags and music. Override with job "caption_y".
    seg["clip"]["transform"]["y"] = JOB.get("caption_y", -0.349)
    return seg


GROUP = U()
group = GROUP
sub_track = dict(copy.deepcopy(STYLE["track"] if STYLE else T_TTRACK), id=U(), segments=[])
subs = []
for k, bl in enumerate(blocks):
    text = " ".join(x["w"] for x in bl).replace(" :", ":")
    a = fr(bl[0]["s"])
    # stays until the next block starts, but never longer than 0.6 s after its last word; last one until the end
    b = min(fr(blocks[k + 1][0]["s"]), fr(bl[-1]["e"] + 0.6)) if k + 1 < len(blocks) else tl
    if STYLE:
        sub_track["segments"].append(styled_caption(text, bl, a, b))
        subs.append((us(a) / 1e6, us(b) / 1e6, text))
        continue
    content = json.loads(T_TMAT["content"])
    content["text"] = text
    for st in content["styles"]:
        st["range"] = [0, len(text)]
    mid = add("texts", T_TMAT, content=json.dumps(content, ensure_ascii=False), group_id=group, recognize_task_id="",
              recognize_text="", words={"start_time": [], "end_time": [], "text": []},
              current_words={"start_time": [], "end_time": [], "text": []}, base_content="")
    s = copy.deepcopy(T_TSEG)
    s.update({"id": U(), "material_id": mid, "extra_material_refs": clone_refs(T_TSEG),
              "target_timerange": {"start": us(a), "duration": us(b) - us(a)}, "keyframe_refs": [], "common_keyframes": []})
    sub_track["segments"].append(s)
    subs.append((us(a) / 1e6, us(b) / 1e6, text))

tracks = [vtrack, sub_track]
for i, t in enumerate(tracks):
    for s in t["segments"]:
        s["track_render_index"] = i
        if t["type"] == "text":
            s["render_index"] = 14000 + i

D = copy.deepcopy(D0)
DID = U()
now = int(time.time() * 1e6)
D.update(id=DID, name="", tracks=tracks, materials=M, duration=TOTAL, fps=float(FPS), create_time=now,
         update_time=now, path="")
D["keyframes"] = {k: [] for k in (D0.get("keyframes") or {})}
if PLATFORM in CANVAS:
    D["canvas_config"] = dict(D0.get("canvas_config") or {}, **CANVAS[PLATFORM])

# ---------- validate (dangling refs, duplicate ids, overlaps, gaps on the main track, keyframes inside their clip)
ids = set()
for k, v in D["materials"].items():
    if isinstance(v, list):
        for x in v:
            assert x["id"] not in ids, "duplicate id"; ids.add(x["id"])
for t in D["tracks"]:
    prev = -1
    for s in t["segments"]:
        assert s["material_id"] in ids and all(r in ids for r in s["extra_material_refs"]), "missing material"
        st, du = s["target_timerange"]["start"], s["target_timerange"]["duration"]
        assert du > 0 and st >= prev and st + du <= TOTAL + 1, (t["type"], "overlap/empty/past end", st)
        prev = st + du
        src = s.get("source_timerange")
        for c in s.get("common_keyframes") or []:
            for p in c["keyframe_list"]:
                assert src["start"] <= p["time_offset"] < src["start"] + src["duration"], "keyframe outside clip"
main = D["tracks"][0]["segments"]
assert main[0]["target_timerange"]["start"] == 0
for x, y in zip(main, main[1:]):
    assert x["target_timerange"]["start"] + x["target_timerange"]["duration"] == y["target_timerange"]["start"], "gap"

# ---------- write project folder
F = f"{ROOT}/{NAME}"
os.makedirs(f"{OUT}/Timelines", exist_ok=True)
blob = json.dumps(D, ensure_ascii=False, separators=(",", ":"))
# CapCut 9.x reads the timeline from Timelines/<main_timeline_id>/ -> write all six copies (root + timeline dir)
os.makedirs(f"{OUT}/Timelines/{DID}", exist_ok=True)
for n in ("draft_info.json", "draft_info.json.bak", "template-2.tmp"):
    open(f"{OUT}/{n}", "w", encoding="utf-8").write(blob)
    open(f"{OUT}/Timelines/{DID}/{n}", "w", encoding="utf-8").write(blob)
json.dump({"config": {"color_space": -1, "mixed_track_mode_on": False, "render_index_track_mode_on": True, "use_float_render": False},
           "create_time": now, "id": DID, "main_timeline_id": DID,
           "timelines": [{"create_time": now, "id": DID, "is_marked_delete": False, "name": "Timeline 01", "update_time": now}],
           "update_time": now, "version": 0}, open(f"{OUT}/Timelines/project.json", "w"), separators=(",", ":"))

meta = copy.deepcopy(META)
mtpl = META["draft_materials"][0]["value"][0]
media = []
for clip in sorted({e[0] for e in EDIT}):
    e = copy.deepcopy(mtpl)
    ext = CLIPS[clip].get("ext", "MOV")
    e.update(id=U(), file_Path=f"./video/{clip}.{ext}", metetype="video", extra_info=f"{clip}.{ext}",
             duration=int(round(CLIPS[clip]["dur"] * 1e6)), width=CLIPS[clip].get("w", 2160), height=CLIPS[clip].get("h", 3840), import_time=now // 10**6, type=0, md5="")
    media.append(e)
for g in meta["draft_materials"]:
    g["value"] = media if g["type"] == 0 else []
meta.update({"draft_fold_path": F, "draft_root_path": ROOT, "draft_name": NAME, "draft_id": DID, "draft_cover": "",
             "draft_materials_copied_info": [], "draft_segment_extra_info": [], "draft_timeline_materials_size_": 0,
             "tm_draft_create": now, "tm_draft_modified": now, "tm_duration": TOTAL, "tm_draft_removed": 0,
             # local project, not a cloud copy (values as in the user's own local projects 0604/0609)
             "cloud_draft_cover": False, "cloud_draft_sync": False, "cloud_package_completed_time": "",
             "draft_cloud_capcut_purchase_info": "", "draft_cloud_last_action_download": False, "draft_cloud_package_type": "",
             "draft_cloud_template_id": "", "draft_cloud_tutorial_info": "", "draft_cloud_videocut_purchase_info": "",
             "draft_is_cloud_temp_draft": False, "tm_draft_cloud_completed": "", "tm_draft_cloud_entry_id": -1,
             "tm_draft_cloud_modified": 0, "tm_draft_cloud_parent_entry_id": -1, "tm_draft_cloud_space_id": -1,
             "tm_draft_cloud_user_id": -1})
json.dump(meta, open(f"{OUT}/draft_meta_info.json", "w"), ensure_ascii=False, separators=(",", ":"))
open(f"{OUT}/draft_settings", "w").write(f"[General]\ncloud_last_modify_platform=mac\ndraft_create_time={now // 10**6}\n"
                                          f"draft_last_edit_time={now // 10**6}\nreal_edit_keys=0\nreal_edit_seconds=0\n")
md5 = hashlib.md5(blob.encode()).hexdigest()
json.dump({"md5": md5, "folder": F, "draft_id": DID, "total_s": TOTAL / 1e6,
           "media": sorted({f"video/{e[0]}.{CLIPS[e[0]].get('ext', 'MOV')}" for e in EDIT})}, open(f"{OUT}/build_report.json", "w"), indent=1)

print(f"project '{NAME}': {len(main)} clips, {len(subs)} ondertitels, {TOTAL / 1e6:.3f}s, md5 {md5}")
for clip, a, b, t0 in placed:
    print(f"  {t0:7.3f}s  {clip}  {a:.2f}-{b:.2f}")
if REFRAME:
    print(f"kadrering feedpost (kopruimte {HEADROOM:.0%}; verschuiving in px op 1080 breed, hoofd/kin als deel van de hoogte):")
    for idx, clip, px, top, chin, short in REFRAME:
        print(f"  {idx:2d} {clip}: {abs(px):4d} px {'omlaag' if px >= 0 else 'omhoog'}, hoofd vanaf {top:.3f}, kin {chin:.3f}"
              + ("  LET OP: kan niet verder zakken, hoofd blijft krap" if short else "")
              + ("  LET OP: kin onder 0,60, kan niet hoger zonder het hoofd af te snijden: ondertitels dicht bij de mond" if chin > CHIN_MAX + 0.005 else ""))
print("spelling uit script:", fixes)
for a, b, t in subs:
    print(f"  {a:6.2f}-{b:6.2f}  {t}")
```

### `install_capcut.py`
```python
"""Runs ON THE USER'S COMPUTER (device_bash). Installs a project built by build_capcut.py into CapCut.
CapCut MUST be fully quit (it rewrites root_meta_info.json and open drafts when it quits).

usage: python3 install_capcut.py BUILD_DIR CLIPS_DIR PROJECTS_DIR BACKUP_DIR
  BUILD_DIR     the OUT_DIR of build_capcut.py, committed to the Mac
  CLIPS_DIR     where the source clips are on the Mac (mount path inside device_bash)
  PROJECTS_DIR  .../Movies/CapCut/User Data/Projects/com.lveditor.draft (mount path inside device_bash)
Steps: back up root_meta_info.json -> create the project folder -> copy the clips into <project>/video/ ->
copy the 6 timeline copies + meta + settings -> add the project to root_meta_info.json -> verify (md5 of all
6 timeline copies, every media file present with the right size, index entry present).
"""
import copy, glob, hashlib, json, os, shutil, sys, time

B, CL, P, BK = sys.argv[1:5]
rep = json.load(open(f"{B}/build_report.json"))
meta = json.load(open(f"{B}/draft_meta_info.json"))
name, did = meta["draft_name"], meta["draft_id"]
F = os.path.join(P, name)
assert not os.path.exists(F), f"bestaat al: {F} (kies een andere naam)"
os.makedirs(BK, exist_ok=True)
stamp = time.strftime("%Y%m%d-%H%M%S")
shutil.copy2(f"{P}/root_meta_info.json", f"{BK}/root_meta_info_{stamp}.json")

os.makedirs(f"{F}/video"); os.makedirs(f"{F}/Timelines/{did}")
for m in rep["media"]:                                     # "video/IMG_0317.MOV"
    src = os.path.join(CL, os.path.basename(m)); dst = os.path.join(F, m)
    shutil.copy2(src, dst)
    assert os.path.getsize(src) == os.path.getsize(dst), ("kopie mislukt", m)
for n in ("draft_info.json", "draft_info.json.bak", "template-2.tmp", "draft_meta_info.json", "draft_settings",
          "Timelines/project.json", f"Timelines/{did}/draft_info.json", f"Timelines/{did}/draft_info.json.bak",
          f"Timelines/{did}/template-2.tmp"):
    shutil.copy2(f"{B}/{n}", f"{F}/{n}")

R = json.load(open(f"{P}/root_meta_info.json"))
R["all_draft_store"] = [x for x in R["all_draft_store"] if x.get("draft_name") != name]
pattern = next((x for x in R["all_draft_store"] if x.get("tm_draft_cloud_entry_id", -1) == -1), R["all_draft_store"][0])
e = copy.deepcopy(pattern)
real_root = R.get("root_path") or os.path.dirname(meta["draft_fold_path"])
e.update(draft_cover="", draft_fold_path=meta["draft_fold_path"], draft_json_file=meta["draft_fold_path"] + "/draft_info.json",
         draft_id=did, draft_name=name, draft_root_path=real_root, tm_duration=meta["tm_duration"],
         tm_draft_create=meta["tm_draft_create"], tm_draft_modified=meta["tm_draft_modified"],
         draft_timeline_materials_size=0, tm_draft_removed=0, tm_draft_cloud_entry_id=-1, tm_draft_cloud_user_id=-1,
         tm_draft_cloud_space_id=-1, tm_draft_cloud_parent_entry_id=-1, tm_draft_cloud_modified=0, tm_draft_cloud_completed="",
         cloud_draft_sync=False, cloud_draft_cover=False, draft_cloud_last_action_download=False)
R["all_draft_store"].insert(0, e)
if isinstance(R.get("draft_ids"), int):
    R["draft_ids"] = len(R["all_draft_store"])
json.dump(R, open(f"{P}/root_meta_info.json", "w"), ensure_ascii=False, separators=(",", ":"))

# verify
md5s = {hashlib.md5(open(x, "rb").read()).hexdigest() for x in
        [f"{F}/{n}" for n in ("draft_info.json", "draft_info.json.bak", "template-2.tmp")] +
        glob.glob(f"{F}/Timelines/{did}/*")}
assert md5s == {rep["md5"]}, ("tijdlijnkopieen verschillen", md5s)
d = json.load(open(f"{F}/draft_info.json"))
for o in d["materials"]["videos"]:
    p = os.path.join(F, "video", os.path.basename(o["path"]))
    assert os.path.exists(p), ("media ontbreekt", p)
assert any(x["draft_name"] == name for x in json.load(open(f"{P}/root_meta_info.json"))["all_draft_store"])
print(f"OK: '{name}' geinstalleerd | 6 tijdlijnkopieen md5 {rep['md5']} | {len(rep['media'])} clips in project | "
      f"{rep['total_s']:.3f}s | index-backup {BK}/root_meta_info_{stamp}.json")
```

### `test_skill.py`
```python
"""Regression test for build_capcut.py: builds the mini test set (tests/fixture) for every platform and compares
the result with the approved reference (tests/golden_<platform>.json). Runs in the cloud container, takes ~5 s.

usage: python3 scripts/test_skill.py                       # test: prints OK or the first differences, exit 1 on a fail
       python3 scripts/test_skill.py --update              # ONLY after the user approved a changed result
       python3 scripts/test_skill.py --compare A_dir B_dir # compare two build folders (e.g. old vs new script)
       python3 scripts/test_skill.py TEMPLATE_DIR          # optional: test with another template than sjabloon/
What is compared: per clip (file, source in/length, timeline position/length, scale, push-in keyframes), per caption
(text, timing, position, scale, highlighted words), the canvas. Random ids, timestamps and paths are ignored.
"""
import json, os, subprocess, sys, tempfile

HERE = os.path.dirname(os.path.abspath(__file__)); SKILL = os.path.dirname(HERE)
T = os.path.join(SKILL, "tests"); FX = os.path.join(T, "fixture")
PLATFORMS = ["tiktok", "reel", "story", "website", "feed45", "feed11"]


def summary(build_dir):
    d = json.load(open(os.path.join(build_dir, "draft_info.json")))
    mats = {o["id"]: o for v in d["materials"].values() if isinstance(v, list) for o in v if isinstance(o, dict) and "id" in o}
    out = {"canvas": {k: d["canvas_config"].get(k) for k in ("ratio", "width", "height")}, "duration": d["duration"],
           "fps": d["fps"], "tracks": []}
    for t in d["tracks"]:
        segs = []
        for s in t["segments"]:
            m = mats[s["material_id"]]
            e = {"target": [s["target_timerange"]["start"], s["target_timerange"]["duration"]],
                 "scale": round(s["clip"]["scale"]["x"], 4), "y": round(s["clip"]["transform"]["y"], 4)}
            if t["type"] == "video":
                e.update(clip=m["material_name"], source=[s["source_timerange"]["start"], s["source_timerange"]["duration"]],
                         keys={c["property_type"]: [[k["time_offset"], round(k["values"][0], 4)] for k in c["keyframe_list"]]
                               for c in s.get("common_keyframes") or []})
            else:
                inner = [mats[r["text_material_id"]] for r in m.get("text_info_resources") or []] or [m]
                tm = inner[0]   # styled caption (text_template) -> its inner text material; plain subtitle -> itself
                c = json.loads(tm["content"]) if tm.get("content") else {}
                e.update(text=c.get("text"), words=tm.get("words"), lmw=tm.get("line_max_width"), styles=[[st.get("range"), st.get("fill", {}).get("content", {}).get("solid", {}).get("color")]
                                                      for st in c.get("styles", [])],
                         anims=[[a.get("type"), a.get("start"), a.get("duration")] for r in s["extra_material_refs"]
                                for a in (mats.get(r, {}).get("animations") or [])])
            segs.append(e)
        out["tracks"].append({"type": t["type"], "segments": segs})
    return out


def diff(a, b, path="", out=None, limit=8):
    out = [] if out is None else out
    if len(out) >= limit: return out
    if isinstance(a, dict) and isinstance(b, dict):
        for k in sorted(set(a) | set(b)):
            diff(a.get(k), b.get(k), f"{path}.{k}", out, limit)
    elif isinstance(a, list) and isinstance(b, list) and len(a) == len(b):
        for i, (x, y) in enumerate(zip(a, b)):
            diff(x, y, f"{path}[{i}]", out, limit)
    elif isinstance(a, list) and isinstance(b, list):
        out.append(f"{path}: aantal verwacht {len(a)} | nu {len(b)}")
        for i, (x, y) in enumerate(zip(a, b)):
            if x != y:
                diff(x, y, f"{path}[{i}] (eerste verschil)", out, limit); break
    elif a != b:
        out.append(f"{path}: verwacht {json.dumps(a, ensure_ascii=False)[:120]} | nu {json.dumps(b, ensure_ascii=False)[:120]}")
    return out


def build(tpl, platform, outdir):
    job = json.load(open(os.path.join(FX, "job.json")))
    if tpl: job["template_dir"] = os.path.abspath(tpl)
    job.update(platform=platform, name=f"test {platform}",
               edit=os.path.join(FX, "edit.json"), words=os.path.join(FX, "words.json"), plan=os.path.join(FX, "plan.json"),
               profile=os.path.join(SKILL, "klanten", "energiegenie", "profiel.json"))
    if os.path.exists(os.path.join(FX, "heads.json")):
        job["heads"] = os.path.join(FX, "heads.json")   # head positions measured on the real frames (headroom.py)
    jp = os.path.join(outdir, f"job_{platform}.json"); json.dump(job, open(jp, "w"))
    r = subprocess.run([sys.executable, os.path.join(HERE, "build_capcut.py"), jp, os.path.join(outdir, platform)],
                       capture_output=True, text=True)
    if r.returncode:
        raise SystemExit(f"FAIL {platform}: build_capcut stopte\n{r.stdout[-800:]}{r.stderr[-800:]}")
    return summary(os.path.join(outdir, platform))


def checks(s, platform):
    """Rules that must hold whatever the golden file says."""
    bad = []
    v = s["tracks"][0]["segments"]
    if v[0]["target"][0] != 0: bad.append("eerste clip start niet op 0")
    for a, b in zip(v, v[1:]):
        if a["target"][0] + a["target"][1] != b["target"][0]: bad.append(f"gat of overlap voor {b['clip']}")
    cv = s["canvas"]
    if platform == "feed45" and (cv["width"], cv["height"]) != (1080, 1350): bad.append(f"canvas 4:5 fout: {cv}")
    if platform == "feed11" and (cv["width"], cv["height"]) != (1920, 1920): bad.append(f"canvas 1:1 fout: {cv}")
    if platform not in ("feed45", "feed11") and (cv["width"], cv["height"]) != (1080, 1920): bad.append(f"canvas 9:16 fout: {cv}")
    fill = {"feed45": 1.4222, "feed11": 1.7778}.get(platform, 1.0)   # 9:16 clip filling the canvas (measured on export)
    if any(x["scale"] < fill - 1e-3 for x in v): bad.append(f"clip kleiner dan het kader (min schaal {fill})")
    return bad


def main():
    if len(sys.argv) > 1 and sys.argv[1] == "--compare":
        d = diff(summary(sys.argv[2]), summary(sys.argv[3]), limit=20)
        print("IDENTIEK" if not d else "VERSCHIL:\n  " + "\n  ".join(d)); sys.exit(1 if d else 0)
    args = [a for a in sys.argv[1:] if not a.startswith("--")]
    tpl, update = (args[0] if args else None), "--update" in sys.argv
    fails = 0
    with tempfile.TemporaryDirectory() as tmp:
        texts = {}
        for p in PLATFORMS:
            s = build(tpl, p, tmp)
            g = os.path.join(T, f"golden_{p}.json")
            bad = checks(s, p)
            texts[p] = [(x["text"], x["target"]) for t in s["tracks"][1:] for x in t["segments"]]
            if update:
                if bad: print(f"NIET bijgewerkt {p}: {bad}"); fails += 1; continue
                json.dump(s, open(g, "w"), ensure_ascii=False, indent=0); print(f"bijgewerkt {p}"); continue
            d = diff(json.load(open(g)), s) if os.path.exists(g) else ["geen golden-bestand"]
            if d or bad:
                fails += 1; print(f"FAIL {p}:\n  " + "\n  ".join(bad + d))
            else:
                print(f"OK   {p}: {len(s['tracks'][0]['segments'])} clips, {sum(len(t['segments']) for t in s['tracks'][1:])} ondertitels")
        if len({json.dumps(v) for v in texts.values()}) != 1:
            fails += 1; print("FAIL: ondertitels (tekst/timing) verschillen tussen platformen")
    print("ALLES OK" if not fails else f"{fails} FOUT(EN): niet verspreiden voor dit opgelost is")
    sys.exit(1 if fails else 0)


main()
```

## Testset

Zet elk bestand letterlijk in `tests/` (de golden-bestanden zijn de goedgekeurde referentie, zie `test_skill.py`).

`tests/golden_feed11.json`
```json
{"canvas":{"ratio":"1:1","width":1920,"height":1920},"duration":18833333,"fps":30.0,"tracks":[{"type":"video","segments":[{"target":[0,5533333],"scale":1.7778,"y":0.1937,"clip":"IMG_0405.MOV","source":[1066666,5533333],"keys":{"KFTypePositionX":[[1066666,0.0],[6566666,0.0]],"KFTypePositionY":[[1066666,0.1937],[6566666,0.1937]],"KFTypeScaleX":[[1066666,1.7778],[6566666,1.8962]],"KFTypeRotation":[[1066666,0.0],[6566666,0.0]]}},{"target":[5533333,3733333],"scale":1.7778,"y":-0.3997,"clip":"IMG_0408.MOV","source":[11400000,3733333],"keys":{"KFTypePositionX":[[11400000,0.0],[15100000,0.0]],"KFTypePositionY":[[11400000,-0.3997],[15100000,-0.3997]],"KFTypeScaleX":[[11400000,1.7778],[15100000,1.8574]],"KFTypeRotation":[[11400000,0.0],[15100000,0.0]]}},{"target":[9266666,9566667],"scale":1.7778,"y":-0.191,"clip":"IMG_0418.MOV","source":[166666,9566667],"keys":{}}]},{"type":"text","segments":[{"target":[433333,1233333],"scale":0.6925,"y":-0.349,"text":"De elektriciteitsprijs","words":{"start_time":[7,627,627],"end_time":[627,627,1247],"text":["De"," ","elektriciteitsprijs"]},"lmw":1.0929,"styles":[[[0,22],[1,1,1]]],"anims":[["caption",0,1233333]]},{"target":[1666666,1433334],"scale":0.6925,"y":-0.349,"text":"verandert voortdurend.","words":{"start_time":[13,413,413],"end_time":[413,413,1193],"text":["verandert"," ","voortdurend."]},"lmw":0.82,"styles":[[[0,22],[1,1,1]]],"anims":[["caption",0,1433334]]},{"target":[3100000,1133333],"scale":0.6925,"y":-0.349,"text":"Soms is stroom goedkoop,","words":{"start_time":[0,240,240,440,440,600,600],"end_time":[240,240,440,440,600,600,1120],"text":["Soms"," ","is"," ","stroom"," ","goedkoop,"]},"lmw":0.82,"styles":[[[0,24],[1,1,1]]],"anims":[["caption",0,1133333]]},{"target":[4233333,1300000],"scale":0.6925,"y":-0.349,"text":"en soms heel duur.","words":{"start_time":[0,227,227,567,567,747,747],"end_time":[227,227,567,567,747,747,1027],"text":["en"," ","soms"," ","heel"," ","duur."]},"lmw":0.82,"styles":[[[0,18],[1,1,1]]],"anims":[["caption",0,1300000]]},{"target":[5533333,1500000],"scale":0.6925,"y":-0.349,"text":"Maar hoe wordt die prijs bepaald?","words":{"start_time":[0,360,360,600,600,700,700,880,880,1080,1080],"end_time":[360,360,600,600,700,700,880,880,1080,1080,1500],"text":["Maar"," ","hoe"," ","wordt"," ","die"," ","prijs"," ","bepaald?"]},"lmw":0.82,"styles":[[[0,33],[1,1,1]]],"anims":[["caption",0,1500000]]},{"target":[7033333,2600000],"scale":0.6925,"y":-0.349,"text":"En waarom stijgt ze mee met de gasprijzen?","words":{"start_time":[0,240,240,600,600,800,800,960,960,1080,1080,1240,1240,1400,1400],"end_time":[240,240,600,600,800,800,960,960,1080,1080,1240,1240,1400,1400,2000],"text":["En"," ","waarom"," ","stijgt"," ","ze"," ","mee"," ","met"," ","de"," ","gasprijzen?"]},"lmw":0.82,"styles":[[[0,42],[1,1,1]]],"anims":[["caption",0,2600000]]},{"target":[9766666,1333334],"scale":0.6925,"y":-0.349,"text":"Elektriciteit voeren we niet van","words":{"start_time":[0,480,480,760,760,920,920,1140,1140],"end_time":[480,480,760,760,920,920,1140,1140,1340],"text":["Elektriciteit"," ","voeren"," ","we"," ","niet"," ","van"]},"lmw":0.82,"styles":[[[0,32],[1,1,1]]],"anims":[["caption",0,1333334]]},{"target":[11100000,1700000],"scale":0.6925,"y":-0.349,"text":"ver in en wordt vooral bij onze","words":{"start_time":[7,227,227,607,607,847,847,1027,1027,1287,1287,1407,1407],"end_time":[227,227,607,607,847,847,1027,1027,1287,1287,1407,1407,1687],"text":["ver"," ","in"," ","en"," ","wordt"," ","vooral"," ","bij"," ","onze"]},"lmw":0.82,"styles":[[[0,31],[1,1,1]]],"anims":[["caption",0,1700000]]},{"target":[12800000,1733333],"scale":0.6925,"y":-0.349,"text":"buurlanden en bij ons geproduceerd.","words":{"start_time":[0,607,607,667,667,827,827,1067,1067],"end_time":[607,607,667,667,827,827,1067,1067,1747],"text":["buurlanden"," ","en"," ","bij"," ","ons"," ","geproduceerd."]},"lmw":0.82,"styles":[[[0,35],[1,1,1]]],"anims":[["caption",0,1733333]]},{"target":[14533333,2066667],"scale":0.6925,"y":-0.349,"text":"Met kern- en gascentrales en de","words":{"start_time":[13,293,293,733,733,833,833,1693,1693,1873,1873],"end_time":[293,293,733,733,833,833,1693,1693,1873,1873,2053],"text":["Met"," ","kern-"," ","en"," ","gascentrales"," ","en"," ","de"]},"lmw":0.82,"styles":[[[0,31],[1,1,1]]],"anims":[["caption",0,2066667]]},{"target":[16600000,2233333],"scale":0.6925,"y":-0.349,"text":"laatste tijd ook veel meer zon en wind.","words":{"start_time":[0,247,247,367,367,727,727,887,887,1087,1087,1427,1427,1687,1687],"end_time":[247,247,367,367,727,727,887,887,1087,1087,1427,1427,1687,1687,1847],"text":["laatste"," ","tijd"," ","ook"," ","veel"," ","meer"," ","zon"," ","en"," ","wind."]},"lmw":0.82,"styles":[[[0,39],[1,1,1]]],"anims":[["caption",0,2233333]]}]}]}
```

`tests/golden_feed45.json`
```json
{"canvas":{"ratio":"custom","width":1080,"height":1350},"duration":18833333,"fps":30.0,"tracks":[{"type":"video","segments":[{"target":[0,5533333],"scale":1.4222,"y":0.1149,"clip":"IMG_0405.MOV","source":[1066666,5533333],"keys":{"KFTypePositionX":[[1066666,0.0],[6566666,0.0]],"KFTypePositionY":[[1066666,0.1149],[6566666,0.1149]],"KFTypeScaleX":[[1066666,1.4222],[6566666,1.5169]],"KFTypeRotation":[[1066666,0.0],[6566666,0.0]]}},{"target":[5533333,3733333],"scale":1.4222,"y":-0.1477,"clip":"IMG_0408.MOV","source":[11400000,3733333],"keys":{"KFTypePositionX":[[11400000,0.0],[15100000,0.0]],"KFTypePositionY":[[11400000,-0.1477],[15100000,-0.1477]],"KFTypeScaleX":[[11400000,1.4222],[15100000,1.4859]],"KFTypeRotation":[[11400000,0.0],[15100000,0.0]]}},{"target":[9266666,9566667],"scale":1.4222,"y":0.0,"clip":"IMG_0418.MOV","source":[166666,9566667],"keys":{}}]},{"type":"text","segments":[{"target":[433333,1233333],"scale":0.6925,"y":-0.349,"text":"De elektriciteitsprijs","words":{"start_time":[7,627,627],"end_time":[627,627,1247],"text":["De"," ","elektriciteitsprijs"]},"lmw":1.0929,"styles":[[[0,22],[1,1,1]]],"anims":[["caption",0,1233333]]},{"target":[1666666,1433334],"scale":0.6925,"y":-0.349,"text":"verandert voortdurend.","words":{"start_time":[13,413,413],"end_time":[413,413,1193],"text":["verandert"," ","voortdurend."]},"lmw":0.82,"styles":[[[0,22],[1,1,1]]],"anims":[["caption",0,1433334]]},{"target":[3100000,1133333],"scale":0.6925,"y":-0.349,"text":"Soms is stroom goedkoop,","words":{"start_time":[0,240,240,440,440,600,600],"end_time":[240,240,440,440,600,600,1120],"text":["Soms"," ","is"," ","stroom"," ","goedkoop,"]},"lmw":0.82,"styles":[[[0,24],[1,1,1]]],"anims":[["caption",0,1133333]]},{"target":[4233333,1300000],"scale":0.6925,"y":-0.349,"text":"en soms heel duur.","words":{"start_time":[0,227,227,567,567,747,747],"end_time":[227,227,567,567,747,747,1027],"text":["en"," ","soms"," ","heel"," ","duur."]},"lmw":0.82,"styles":[[[0,18],[1,1,1]]],"anims":[["caption",0,1300000]]},{"target":[5533333,1500000],"scale":0.6925,"y":-0.349,"text":"Maar hoe wordt die prijs bepaald?","words":{"start_time":[0,360,360,600,600,700,700,880,880,1080,1080],"end_time":[360,360,600,600,700,700,880,880,1080,1080,1500],"text":["Maar"," ","hoe"," ","wordt"," ","die"," ","prijs"," ","bepaald?"]},"lmw":0.82,"styles":[[[0,33],[1,1,1]]],"anims":[["caption",0,1500000]]},{"target":[7033333,2600000],"scale":0.6925,"y":-0.349,"text":"En waarom stijgt ze mee met de gasprijzen?","words":{"start_time":[0,240,240,600,600,800,800,960,960,1080,1080,1240,1240,1400,1400],"end_time":[240,240,600,600,800,800,960,960,1080,1080,1240,1240,1400,1400,2000],"text":["En"," ","waarom"," ","stijgt"," ","ze"," ","mee"," ","met"," ","de"," ","gasprijzen?"]},"lmw":0.82,"styles":[[[0,42],[1,1,1]]],"anims":[["caption",0,2600000]]},{"target":[9766666,1333334],"scale":0.6925,"y":-0.349,"text":"Elektriciteit voeren we niet van","words":{"start_time":[0,480,480,760,760,920,920,1140,1140],"end_time":[480,480,760,760,920,920,1140,1140,1340],"text":["Elektriciteit"," ","voeren"," ","we"," ","niet"," ","van"]},"lmw":0.82,"styles":[[[0,32],[1,1,1]]],"anims":[["caption",0,1333334]]},{"target":[11100000,1700000],"scale":0.6925,"y":-0.349,"text":"ver in en wordt vooral bij onze","words":{"start_time":[7,227,227,607,607,847,847,1027,1027,1287,1287,1407,1407],"end_time":[227,227,607,607,847,847,1027,1027,1287,1287,1407,1407,1687],"text":["ver"," ","in"," ","en"," ","wordt"," ","vooral"," ","bij"," ","onze"]},"lmw":0.82,"styles":[[[0,31],[1,1,1]]],"anims":[["caption",0,1700000]]},{"target":[12800000,1733333],"scale":0.6925,"y":-0.349,"text":"buurlanden en bij ons geproduceerd.","words":{"start_time":[0,607,607,667,667,827,827,1067,1067],"end_time":[607,607,667,667,827,827,1067,1067,1747],"text":["buurlanden"," ","en"," ","bij"," ","ons"," ","geproduceerd."]},"lmw":0.82,"styles":[[[0,35],[1,1,1]]],"anims":[["caption",0,1733333]]},{"target":[14533333,2066667],"scale":0.6925,"y":-0.349,"text":"Met kern- en gascentrales en de","words":{"start_time":[13,293,293,733,733,833,833,1693,1693,1873,1873],"end_time":[293,293,733,733,833,833,1693,1693,1873,1873,2053],"text":["Met"," ","kern-"," ","en"," ","gascentrales"," ","en"," ","de"]},"lmw":0.82,"styles":[[[0,31],[1,1,1]]],"anims":[["caption",0,2066667]]},{"target":[16600000,2233333],"scale":0.6925,"y":-0.349,"text":"laatste tijd ook veel meer zon en wind.","words":{"start_time":[0,247,247,367,367,727,727,887,887,1087,1087,1427,1427,1687,1687],"end_time":[247,247,367,367,727,727,887,887,1087,1087,1427,1427,1687,1687,1847],"text":["laatste"," ","tijd"," ","ook"," ","veel"," ","meer"," ","zon"," ","en"," ","wind."]},"lmw":0.82,"styles":[[[0,39],[1,1,1]]],"anims":[["caption",0,2233333]]}]}]}
```

`tests/golden_reel.json`
```json
{"canvas":{"ratio":"9:16","width":1080,"height":1920},"duration":18833333,"fps":30.0,"tracks":[{"type":"video","segments":[{"target":[0,5533333],"scale":1.0,"y":0.0,"clip":"IMG_0405.MOV","source":[1066666,5533333],"keys":{"KFTypePositionX":[[1066666,0.0],[6566666,0.0]],"KFTypePositionY":[[1066666,0.0],[6566666,0.0]],"KFTypeScaleX":[[1066666,1.0],[6566666,1.0666]],"KFTypeRotation":[[1066666,0.0],[6566666,0.0]]}},{"target":[5533333,3733333],"scale":1.0,"y":0.0,"clip":"IMG_0408.MOV","source":[11400000,3733333],"keys":{"KFTypePositionX":[[11400000,0.0],[15100000,0.0]],"KFTypePositionY":[[11400000,0.0],[15100000,0.0]],"KFTypeScaleX":[[11400000,1.0],[15100000,1.0448]],"KFTypeRotation":[[11400000,0.0],[15100000,0.0]]}},{"target":[9266666,9566667],"scale":1.0,"y":0.0,"clip":"IMG_0418.MOV","source":[166666,9566667],"keys":{}}]},{"type":"text","segments":[{"target":[433333,1233333],"scale":0.6925,"y":-0.15,"text":"De elektriciteitsprijs","words":{"start_time":[7,627,627],"end_time":[627,627,1247],"text":["De"," ","elektriciteitsprijs"]},"lmw":1.0929,"styles":[[[0,22],[1,1,1]]],"anims":[["caption",0,1233333]]},{"target":[1666666,1433334],"scale":0.6925,"y":-0.15,"text":"verandert voortdurend.","words":{"start_time":[13,413,413],"end_time":[413,413,1193],"text":["verandert"," ","voortdurend."]},"lmw":0.82,"styles":[[[0,22],[1,1,1]]],"anims":[["caption",0,1433334]]},{"target":[3100000,1133333],"scale":0.6925,"y":-0.15,"text":"Soms is stroom goedkoop,","words":{"start_time":[0,240,240,440,440,600,600],"end_time":[240,240,440,440,600,600,1120],"text":["Soms"," ","is"," ","stroom"," ","goedkoop,"]},"lmw":0.82,"styles":[[[0,24],[1,1,1]]],"anims":[["caption",0,1133333]]},{"target":[4233333,1300000],"scale":0.6925,"y":-0.15,"text":"en soms heel duur.","words":{"start_time":[0,227,227,567,567,747,747],"end_time":[227,227,567,567,747,747,1027],"text":["en"," ","soms"," ","heel"," ","duur."]},"lmw":0.82,"styles":[[[0,18],[1,1,1]]],"anims":[["caption",0,1300000]]},{"target":[5533333,1500000],"scale":0.6925,"y":-0.15,"text":"Maar hoe wordt die prijs bepaald?","words":{"start_time":[0,360,360,600,600,700,700,880,880,1080,1080],"end_time":[360,360,600,600,700,700,880,880,1080,1080,1500],"text":["Maar"," ","hoe"," ","wordt"," ","die"," ","prijs"," ","bepaald?"]},"lmw":0.82,"styles":[[[0,33],[1,1,1]]],"anims":[["caption",0,1500000]]},{"target":[7033333,2600000],"scale":0.6925,"y":-0.15,"text":"En waarom stijgt ze mee met de gasprijzen?","words":{"start_time":[0,240,240,600,600,800,800,960,960,1080,1080,1240,1240,1400,1400],"end_time":[240,240,600,600,800,800,960,960,1080,1080,1240,1240,1400,1400,2000],"text":["En"," ","waarom"," ","stijgt"," ","ze"," ","mee"," ","met"," ","de"," ","gasprijzen?"]},"lmw":0.82,"styles":[[[0,42],[1,1,1]]],"anims":[["caption",0,2600000]]},{"target":[9766666,1333334],"scale":0.6925,"y":-0.15,"text":"Elektriciteit voeren we niet van","words":{"start_time":[0,480,480,760,760,920,920,1140,1140],"end_time":[480,480,760,760,920,920,1140,1140,1340],"text":["Elektriciteit"," ","voeren"," ","we"," ","niet"," ","van"]},"lmw":0.82,"styles":[[[0,32],[1,1,1]]],"anims":[["caption",0,1333334]]},{"target":[11100000,1700000],"scale":0.6925,"y":-0.15,"text":"ver in en wordt vooral bij onze","words":{"start_time":[7,227,227,607,607,847,847,1027,1027,1287,1287,1407,1407],"end_time":[227,227,607,607,847,847,1027,1027,1287,1287,1407,1407,1687],"text":["ver"," ","in"," ","en"," ","wordt"," ","vooral"," ","bij"," ","onze"]},"lmw":0.82,"styles":[[[0,31],[1,1,1]]],"anims":[["caption",0,1700000]]},{"target":[12800000,1733333],"scale":0.6925,"y":-0.15,"text":"buurlanden en bij ons geproduceerd.","words":{"start_time":[0,607,607,667,667,827,827,1067,1067],"end_time":[607,607,667,667,827,827,1067,1067,1747],"text":["buurlanden"," ","en"," ","bij"," ","ons"," ","geproduceerd."]},"lmw":0.82,"styles":[[[0,35],[1,1,1]]],"anims":[["caption",0,1733333]]},{"target":[14533333,2066667],"scale":0.6925,"y":-0.15,"text":"Met kern- en gascentrales en de","words":{"start_time":[13,293,293,733,733,833,833,1693,1693,1873,1873],"end_time":[293,293,733,733,833,833,1693,1693,1873,1873,2053],"text":["Met"," ","kern-"," ","en"," ","gascentrales"," ","en"," ","de"]},"lmw":0.82,"styles":[[[0,31],[1,1,1]]],"anims":[["caption",0,2066667]]},{"target":[16600000,2233333],"scale":0.6925,"y":-0.15,"text":"laatste tijd ook veel meer zon en wind.","words":{"start_time":[0,247,247,367,367,727,727,887,887,1087,1087,1427,1427,1687,1687],"end_time":[247,247,367,367,727,727,887,887,1087,1087,1427,1427,1687,1687,1847],"text":["laatste"," ","tijd"," ","ook"," ","veel"," ","meer"," ","zon"," ","en"," ","wind."]},"lmw":0.82,"styles":[[[0,39],[1,1,1]]],"anims":[["caption",0,2233333]]}]}]}
```

`tests/golden_story.json`
```json
{"canvas":{"ratio":"9:16","width":1080,"height":1920},"duration":18833333,"fps":30.0,"tracks":[{"type":"video","segments":[{"target":[0,5533333],"scale":1.0,"y":0.0,"clip":"IMG_0405.MOV","source":[1066666,5533333],"keys":{"KFTypePositionX":[[1066666,0.0],[6566666,0.0]],"KFTypePositionY":[[1066666,0.0],[6566666,0.0]],"KFTypeScaleX":[[1066666,1.0],[6566666,1.0666]],"KFTypeRotation":[[1066666,0.0],[6566666,0.0]]}},{"target":[5533333,3733333],"scale":1.0,"y":0.0,"clip":"IMG_0408.MOV","source":[11400000,3733333],"keys":{"KFTypePositionX":[[11400000,0.0],[15100000,0.0]],"KFTypePositionY":[[11400000,0.0],[15100000,0.0]],"KFTypeScaleX":[[11400000,1.0],[15100000,1.0448]],"KFTypeRotation":[[11400000,0.0],[15100000,0.0]]}},{"target":[9266666,9566667],"scale":1.0,"y":0.0,"clip":"IMG_0418.MOV","source":[166666,9566667],"keys":{}}]},{"type":"text","segments":[{"target":[433333,1233333],"scale":0.6925,"y":-0.349,"text":"De elektriciteitsprijs","words":{"start_time":[7,627,627],"end_time":[627,627,1247],"text":["De"," ","elektriciteitsprijs"]},"lmw":1.0929,"styles":[[[0,22],[1,1,1]]],"anims":[["caption",0,1233333]]},{"target":[1666666,1433334],"scale":0.6925,"y":-0.349,"text":"verandert voortdurend.","words":{"start_time":[13,413,413],"end_time":[413,413,1193],"text":["verandert"," ","voortdurend."]},"lmw":0.82,"styles":[[[0,22],[1,1,1]]],"anims":[["caption",0,1433334]]},{"target":[3100000,1133333],"scale":0.6925,"y":-0.349,"text":"Soms is stroom goedkoop,","words":{"start_time":[0,240,240,440,440,600,600],"end_time":[240,240,440,440,600,600,1120],"text":["Soms"," ","is"," ","stroom"," ","goedkoop,"]},"lmw":0.82,"styles":[[[0,24],[1,1,1]]],"anims":[["caption",0,1133333]]},{"target":[4233333,1300000],"scale":0.6925,"y":-0.349,"text":"en soms heel duur.","words":{"start_time":[0,227,227,567,567,747,747],"end_time":[227,227,567,567,747,747,1027],"text":["en"," ","soms"," ","heel"," ","duur."]},"lmw":0.82,"styles":[[[0,18],[1,1,1]]],"anims":[["caption",0,1300000]]},{"target":[5533333,1500000],"scale":0.6925,"y":-0.349,"text":"Maar hoe wordt die prijs bepaald?","words":{"start_time":[0,360,360,600,600,700,700,880,880,1080,1080],"end_time":[360,360,600,600,700,700,880,880,1080,1080,1500],"text":["Maar"," ","hoe"," ","wordt"," ","die"," ","prijs"," ","bepaald?"]},"lmw":0.82,"styles":[[[0,33],[1,1,1]]],"anims":[["caption",0,1500000]]},{"target":[7033333,2600000],"scale":0.6925,"y":-0.349,"text":"En waarom stijgt ze mee met de gasprijzen?","words":{"start_time":[0,240,240,600,600,800,800,960,960,1080,1080,1240,1240,1400,1400],"end_time":[240,240,600,600,800,800,960,960,1080,1080,1240,1240,1400,1400,2000],"text":["En"," ","waarom"," ","stijgt"," ","ze"," ","mee"," ","met"," ","de"," ","gasprijzen?"]},"lmw":0.82,"styles":[[[0,42],[1,1,1]]],"anims":[["caption",0,2600000]]},{"target":[9766666,1333334],"scale":0.6925,"y":-0.349,"text":"Elektriciteit voeren we niet van","words":{"start_time":[0,480,480,760,760,920,920,1140,1140],"end_time":[480,480,760,760,920,920,1140,1140,1340],"text":["Elektriciteit"," ","voeren"," ","we"," ","niet"," ","van"]},"lmw":0.82,"styles":[[[0,32],[1,1,1]]],"anims":[["caption",0,1333334]]},{"target":[11100000,1700000],"scale":0.6925,"y":-0.349,"text":"ver in en wordt vooral bij onze","words":{"start_time":[7,227,227,607,607,847,847,1027,1027,1287,1287,1407,1407],"end_time":[227,227,607,607,847,847,1027,1027,1287,1287,1407,1407,1687],"text":["ver"," ","in"," ","en"," ","wordt"," ","vooral"," ","bij"," ","onze"]},"lmw":0.82,"styles":[[[0,31],[1,1,1]]],"anims":[["caption",0,1700000]]},{"target":[12800000,1733333],"scale":0.6925,"y":-0.349,"text":"buurlanden en bij ons geproduceerd.","words":{"start_time":[0,607,607,667,667,827,827,1067,1067],"end_time":[607,607,667,667,827,827,1067,1067,1747],"text":["buurlanden"," ","en"," ","bij"," ","ons"," ","geproduceerd."]},"lmw":0.82,"styles":[[[0,35],[1,1,1]]],"anims":[["caption",0,1733333]]},{"target":[14533333,2066667],"scale":0.6925,"y":-0.349,"text":"Met kern- en gascentrales en de","words":{"start_time":[13,293,293,733,733,833,833,1693,1693,1873,1873],"end_time":[293,293,733,733,833,833,1693,1693,1873,1873,2053],"text":["Met"," ","kern-"," ","en"," ","gascentrales"," ","en"," ","de"]},"lmw":0.82,"styles":[[[0,31],[1,1,1]]],"anims":[["caption",0,2066667]]},{"target":[16600000,2233333],"scale":0.6925,"y":-0.349,"text":"laatste tijd ook veel meer zon en wind.","words":{"start_time":[0,247,247,367,367,727,727,887,887,1087,1087,1427,1427,1687,1687],"end_time":[247,247,367,367,727,727,887,887,1087,1087,1427,1427,1687,1687,1847],"text":["laatste"," ","tijd"," ","ook"," ","veel"," ","meer"," ","zon"," ","en"," ","wind."]},"lmw":0.82,"styles":[[[0,39],[1,1,1]]],"anims":[["caption",0,2233333]]}]}]}
```

`tests/golden_tiktok.json`
```json
{"canvas":{"ratio":"9:16","width":1080,"height":1920},"duration":18833333,"fps":30.0,"tracks":[{"type":"video","segments":[{"target":[0,5533333],"scale":1.0,"y":0.0,"clip":"IMG_0405.MOV","source":[1066666,5533333],"keys":{"KFTypePositionX":[[1066666,0.0],[6566666,0.0]],"KFTypePositionY":[[1066666,0.0],[6566666,0.0]],"KFTypeScaleX":[[1066666,1.0],[6566666,1.0666]],"KFTypeRotation":[[1066666,0.0],[6566666,0.0]]}},{"target":[5533333,3733333],"scale":1.0,"y":0.0,"clip":"IMG_0408.MOV","source":[11400000,3733333],"keys":{"KFTypePositionX":[[11400000,0.0],[15100000,0.0]],"KFTypePositionY":[[11400000,0.0],[15100000,0.0]],"KFTypeScaleX":[[11400000,1.0],[15100000,1.0448]],"KFTypeRotation":[[11400000,0.0],[15100000,0.0]]}},{"target":[9266666,9566667],"scale":1.0,"y":0.0,"clip":"IMG_0418.MOV","source":[166666,9566667],"keys":{}}]},{"type":"text","segments":[{"target":[433333,1233333],"scale":0.6925,"y":-0.349,"text":"De elektriciteitsprijs","words":{"start_time":[7,627,627],"end_time":[627,627,1247],"text":["De"," ","elektriciteitsprijs"]},"lmw":1.0929,"styles":[[[0,22],[1,1,1]]],"anims":[["caption",0,1233333]]},{"target":[1666666,1433334],"scale":0.6925,"y":-0.349,"text":"verandert voortdurend.","words":{"start_time":[13,413,413],"end_time":[413,413,1193],"text":["verandert"," ","voortdurend."]},"lmw":0.82,"styles":[[[0,22],[1,1,1]]],"anims":[["caption",0,1433334]]},{"target":[3100000,1133333],"scale":0.6925,"y":-0.349,"text":"Soms is stroom goedkoop,","words":{"start_time":[0,240,240,440,440,600,600],"end_time":[240,240,440,440,600,600,1120],"text":["Soms"," ","is"," ","stroom"," ","goedkoop,"]},"lmw":0.82,"styles":[[[0,24],[1,1,1]]],"anims":[["caption",0,1133333]]},{"target":[4233333,1300000],"scale":0.6925,"y":-0.349,"text":"en soms heel duur.","words":{"start_time":[0,227,227,567,567,747,747],"end_time":[227,227,567,567,747,747,1027],"text":["en"," ","soms"," ","heel"," ","duur."]},"lmw":0.82,"styles":[[[0,18],[1,1,1]]],"anims":[["caption",0,1300000]]},{"target":[5533333,1500000],"scale":0.6925,"y":-0.349,"text":"Maar hoe wordt die prijs bepaald?","words":{"start_time":[0,360,360,600,600,700,700,880,880,1080,1080],"end_time":[360,360,600,600,700,700,880,880,1080,1080,1500],"text":["Maar"," ","hoe"," ","wordt"," ","die"," ","prijs"," ","bepaald?"]},"lmw":0.82,"styles":[[[0,33],[1,1,1]]],"anims":[["caption",0,1500000]]},{"target":[7033333,2600000],"scale":0.6925,"y":-0.349,"text":"En waarom stijgt ze mee met de gasprijzen?","words":{"start_time":[0,240,240,600,600,800,800,960,960,1080,1080,1240,1240,1400,1400],"end_time":[240,240,600,600,800,800,960,960,1080,1080,1240,1240,1400,1400,2000],"text":["En"," ","waarom"," ","stijgt"," ","ze"," ","mee"," ","met"," ","de"," ","gasprijzen?"]},"lmw":0.82,"styles":[[[0,42],[1,1,1]]],"anims":[["caption",0,2600000]]},{"target":[9766666,1333334],"scale":0.6925,"y":-0.349,"text":"Elektriciteit voeren we niet van","words":{"start_time":[0,480,480,760,760,920,920,1140,1140],"end_time":[480,480,760,760,920,920,1140,1140,1340],"text":["Elektriciteit"," ","voeren"," ","we"," ","niet"," ","van"]},"lmw":0.82,"styles":[[[0,32],[1,1,1]]],"anims":[["caption",0,1333334]]},{"target":[11100000,1700000],"scale":0.6925,"y":-0.349,"text":"ver in en wordt vooral bij onze","words":{"start_time":[7,227,227,607,607,847,847,1027,1027,1287,1287,1407,1407],"end_time":[227,227,607,607,847,847,1027,1027,1287,1287,1407,1407,1687],"text":["ver"," ","in"," ","en"," ","wordt"," ","vooral"," ","bij"," ","onze"]},"lmw":0.82,"styles":[[[0,31],[1,1,1]]],"anims":[["caption",0,1700000]]},{"target":[12800000,1733333],"scale":0.6925,"y":-0.349,"text":"buurlanden en bij ons geproduceerd.","words":{"start_time":[0,607,607,667,667,827,827,1067,1067],"end_time":[607,607,667,667,827,827,1067,1067,1747],"text":["buurlanden"," ","en"," ","bij"," ","ons"," ","geproduceerd."]},"lmw":0.82,"styles":[[[0,35],[1,1,1]]],"anims":[["caption",0,1733333]]},{"target":[14533333,2066667],"scale":0.6925,"y":-0.349,"text":"Met kern- en gascentrales en de","words":{"start_time":[13,293,293,733,733,833,833,1693,1693,1873,1873],"end_time":[293,293,733,733,833,833,1693,1693,1873,1873,2053],"text":["Met"," ","kern-"," ","en"," ","gascentrales"," ","en"," ","de"]},"lmw":0.82,"styles":[[[0,31],[1,1,1]]],"anims":[["caption",0,2066667]]},{"target":[16600000,2233333],"scale":0.6925,"y":-0.349,"text":"laatste tijd ook veel meer zon en wind.","words":{"start_time":[0,247,247,367,367,727,727,887,887,1087,1087,1427,1427,1687,1687],"end_time":[247,247,367,367,727,727,887,887,1087,1087,1427,1427,1687,1687,1847],"text":["laatste"," ","tijd"," ","ook"," ","veel"," ","meer"," ","zon"," ","en"," ","wind."]},"lmw":0.82,"styles":[[[0,39],[1,1,1]]],"anims":[["caption",0,2233333]]}]}]}
```

`tests/golden_website.json`
```json
{"canvas":{"ratio":"9:16","width":1080,"height":1920},"duration":18833333,"fps":30.0,"tracks":[{"type":"video","segments":[{"target":[0,5533333],"scale":1.0,"y":0.0,"clip":"IMG_0405.MOV","source":[1066666,5533333],"keys":{"KFTypePositionX":[[1066666,0.0],[6566666,0.0]],"KFTypePositionY":[[1066666,0.0],[6566666,0.0]],"KFTypeScaleX":[[1066666,1.0],[6566666,1.0666]],"KFTypeRotation":[[1066666,0.0],[6566666,0.0]]}},{"target":[5533333,3733333],"scale":1.0,"y":0.0,"clip":"IMG_0408.MOV","source":[11400000,3733333],"keys":{"KFTypePositionX":[[11400000,0.0],[15100000,0.0]],"KFTypePositionY":[[11400000,0.0],[15100000,0.0]],"KFTypeScaleX":[[11400000,1.0],[15100000,1.0448]],"KFTypeRotation":[[11400000,0.0],[15100000,0.0]]}},{"target":[9266666,9566667],"scale":1.0,"y":0.0,"clip":"IMG_0418.MOV","source":[166666,9566667],"keys":{}}]},{"type":"text","segments":[{"target":[433333,1233333],"scale":0.6925,"y":-0.349,"text":"De elektriciteitsprijs","words":{"start_time":[7,627,627],"end_time":[627,627,1247],"text":["De"," ","elektriciteitsprijs"]},"lmw":1.0929,"styles":[[[0,22],[1,1,1]]],"anims":[["caption",0,1233333]]},{"target":[1666666,1433334],"scale":0.6925,"y":-0.349,"text":"verandert voortdurend.","words":{"start_time":[13,413,413],"end_time":[413,413,1193],"text":["verandert"," ","voortdurend."]},"lmw":0.82,"styles":[[[0,22],[1,1,1]]],"anims":[["caption",0,1433334]]},{"target":[3100000,1133333],"scale":0.6925,"y":-0.349,"text":"Soms is stroom goedkoop,","words":{"start_time":[0,240,240,440,440,600,600],"end_time":[240,240,440,440,600,600,1120],"text":["Soms"," ","is"," ","stroom"," ","goedkoop,"]},"lmw":0.82,"styles":[[[0,24],[1,1,1]]],"anims":[["caption",0,1133333]]},{"target":[4233333,1300000],"scale":0.6925,"y":-0.349,"text":"en soms heel duur.","words":{"start_time":[0,227,227,567,567,747,747],"end_time":[227,227,567,567,747,747,1027],"text":["en"," ","soms"," ","heel"," ","duur."]},"lmw":0.82,"styles":[[[0,18],[1,1,1]]],"anims":[["caption",0,1300000]]},{"target":[5533333,1500000],"scale":0.6925,"y":-0.349,"text":"Maar hoe wordt die prijs bepaald?","words":{"start_time":[0,360,360,600,600,700,700,880,880,1080,1080],"end_time":[360,360,600,600,700,700,880,880,1080,1080,1500],"text":["Maar"," ","hoe"," ","wordt"," ","die"," ","prijs"," ","bepaald?"]},"lmw":0.82,"styles":[[[0,33],[1,1,1]]],"anims":[["caption",0,1500000]]},{"target":[7033333,2600000],"scale":0.6925,"y":-0.349,"text":"En waarom stijgt ze mee met de gasprijzen?","words":{"start_time":[0,240,240,600,600,800,800,960,960,1080,1080,1240,1240,1400,1400],"end_time":[240,240,600,600,800,800,960,960,1080,1080,1240,1240,1400,1400,2000],"text":["En"," ","waarom"," ","stijgt"," ","ze"," ","mee"," ","met"," ","de"," ","gasprijzen?"]},"lmw":0.82,"styles":[[[0,42],[1,1,1]]],"anims":[["caption",0,2600000]]},{"target":[9766666,1333334],"scale":0.6925,"y":-0.349,"text":"Elektriciteit voeren we niet van","words":{"start_time":[0,480,480,760,760,920,920,1140,1140],"end_time":[480,480,760,760,920,920,1140,1140,1340],"text":["Elektriciteit"," ","voeren"," ","we"," ","niet"," ","van"]},"lmw":0.82,"styles":[[[0,32],[1,1,1]]],"anims":[["caption",0,1333334]]},{"target":[11100000,1700000],"scale":0.6925,"y":-0.349,"text":"ver in en wordt vooral bij onze","words":{"start_time":[7,227,227,607,607,847,847,1027,1027,1287,1287,1407,1407],"end_time":[227,227,607,607,847,847,1027,1027,1287,1287,1407,1407,1687],"text":["ver"," ","in"," ","en"," ","wordt"," ","vooral"," ","bij"," ","onze"]},"lmw":0.82,"styles":[[[0,31],[1,1,1]]],"anims":[["caption",0,1700000]]},{"target":[12800000,1733333],"scale":0.6925,"y":-0.349,"text":"buurlanden en bij ons geproduceerd.","words":{"start_time":[0,607,607,667,667,827,827,1067,1067],"end_time":[607,607,667,667,827,827,1067,1067,1747],"text":["buurlanden"," ","en"," ","bij"," ","ons"," ","geproduceerd."]},"lmw":0.82,"styles":[[[0,35],[1,1,1]]],"anims":[["caption",0,1733333]]},{"target":[14533333,2066667],"scale":0.6925,"y":-0.349,"text":"Met kern- en gascentrales en de","words":{"start_time":[13,293,293,733,733,833,833,1693,1693,1873,1873],"end_time":[293,293,733,733,833,833,1693,1693,1873,1873,2053],"text":["Met"," ","kern-"," ","en"," ","gascentrales"," ","en"," ","de"]},"lmw":0.82,"styles":[[[0,31],[1,1,1]]],"anims":[["caption",0,2066667]]},{"target":[16600000,2233333],"scale":0.6925,"y":-0.349,"text":"laatste tijd ook veel meer zon en wind.","words":{"start_time":[0,247,247,367,367,727,727,887,887,1087,1087,1427,1427,1687,1687],"end_time":[247,247,367,367,727,727,887,887,1087,1087,1427,1427,1687,1687,1847],"text":["laatste"," ","tijd"," ","ook"," ","veel"," ","meer"," ","zon"," ","en"," ","wind."]},"lmw":0.82,"styles":[[[0,39],[1,1,1]]],"anims":[["caption",0,2233333]]}]}]}
```

`tests/fixture/edit.json`
```json
{"video":"t2","total":18.84,"items":[{"clip":"IMG_0405","in":1.06,"out":6.61,"shots":[1],"why":["push-in: hook"],"s0":1.0,"s1":1.0666,"tl":0.0},{"clip":"IMG_0408","in":11.4,"out":15.13,"shots":[1],"why":["push-in: vraag"],"s0":1.0,"s1":1.0448,"tl":5.55},{"clip":"IMG_0418","in":0.16,"out":9.72,"shots":[2],"why":[],"s0":1.0,"s1":1.0,"tl":9.28}]}
```

`tests/fixture/heads.json`
```json
{"0":{"top":0.3416,"chin":0.6038,"frames":6},"1":{"top":0.1609,"chin":0.4204,"frames":4},"2":{"top":0.2044,"chin":0.3671,"frames":10}}
```

`tests/fixture/job.json`
```json
{"name":"test","projects_root":"/Users/test/Movies/CapCut/User Data/Projects/com.lveditor.draft","video":"t2","clips":{"IMG_0405":{"dur":7.631667,"w":2160,"h":3840,"ext":"MOV"},"IMG_0408":{"dur":16.666667,"w":2160,"h":3840,"ext":"MOV"},"IMG_0418":{"dur":10.9,"w":2160,"h":3840,"ext":"MOV"}},"font_path":"/Users/test/Library/Containers/com.lemon.lvoverseas/Data/Library/Fonts/FTAktual-Bold.otf"}
```

`tests/fixture/plan.json`
```json
{"project":"EnergieGenie topics deel 3 (2026)","source":"Energiegenie_topics__script_deel_3_2026_final.docx","format":{"width":1080,"height":1920,"fps":null,"aspect":"9:16","note":"fps meten op de clips"},"videos":[{"id":"t2","title":"Wat bepaalt de elektriciteitsprijs die jij betaalt?","language":"NL","lines":[{"n":1,"role":"hook","text":"Iedere dag verandert de elektriciteitsprijs. Soms is stroom goedkoop, soms extreem duur. Maar hoe wordt die prijs bepaald? En waarom stijgt ze zo vaak mee met de gasprijs?"},{"n":2,"text":"Elektriciteit voeren we niet van ver in. Ze wordt vooral bij ons en in onze buurlanden geproduceerd. Met kerncentrales, gascentrales en steeds meer zon en wind."},{"n":3,"text":"Op de energiemarkt geldt een slim principe om de vraag naar energie te dekken:"},{"n":4,"text":"Al die bronnen worden gerangschikt van goedkoop naar duur en zo ingeschakeld tot de vraag gedekt is. Zon en wind eerst, dan kernenergie, als laatste de duurdere gascentrales. En de laatste centrale die nog nodig is, bepaalt de prijs voor alle stroom. Dat heet de merit order."},{"n":5,"text":"Is er veel zon en wind? Dan is de vraag al grotendeels gedekt met goedkope stroom, en zakt de prijs. Op topmomenten soms zelfs tot onder nul."},{"n":6,"text":"Is er weinig zon en wind, of net veel vraag? Dan moet een gascentrale bijspringen om alles te dekken. En die zet dan de prijs. Daarom beweegt je stroomprijs mee met gas."},{"n":7,"text":"Ook andere zaken spelen mee: een koude winter die de vraag opdrijft, onderhoud aan een kerncentrale, of de CO2-prijs die gas duurder maakt."},{"n":8,"role":"end","text":"Dus: je energieprijs hangt af van vraag, aanbod en de duurste centrale die moet draaien. Zo eenvoudig kan energie zijn."}],"speaker":null}]}
```

`tests/fixture/words.json`
```json
{"IMG_0405":{"dur":7.636,"words":[{"w":"De","s":1.5,"e":2.12,"p":0.841},{"w":"elektriciteitsprijs","s":2.12,"e":2.74,"p":0.976},{"w":"verandert","s":2.74,"e":3.14,"p":0.984},{"w":"voortdurend.","s":3.14,"e":3.92,"p":0.91},{"w":"Soms","s":4.16,"e":4.4,"p":0.997},{"w":"is","s":4.4,"e":4.6,"p":0.948},{"w":"stroom","s":4.6,"e":4.76,"p":0.638},{"w":"goedkoop","s":4.76,"e":5.28,"p":0.624},{"w":"en","s":5.28,"e":5.52,"p":0.814},{"w":"soms","s":5.52,"e":5.86,"p":0.996},{"w":"heel","s":5.86,"e":6.04,"p":0.936},{"w":"duur.","s":6.04,"e":6.32,"p":0.988}]},"IMG_0408":{"dur":16.681,"words":[{"w":"Maar","s":0.0,"e":0.74,"p":0.217},{"w":"hoe","s":0.74,"e":1.0,"p":0.967},{"w":"wordt","s":1.0,"e":1.1,"p":0.995},{"w":"die","s":1.1,"e":1.3,"p":0.891},{"w":"prijs","s":1.3,"e":1.52,"p":0.991},{"w":"bepaald","s":1.52,"e":2.04,"p":0.994},{"w":"en","s":2.04,"e":2.38,"p":0.828},{"w":"waarom","s":2.38,"e":2.78,"p":0.999},{"w":"stijgt","s":2.78,"e":3.04,"p":0.641},{"w":"ze","s":3.04,"e":3.12,"p":0.927},{"w":"vaak","s":3.12,"e":3.34,"p":0.969},{"w":"mee","s":3.34,"e":3.6,"p":0.994},{"w":"met","s":3.6,"e":3.78,"p":0.984},{"w":"de","s":3.78,"e":3.92,"p":0.981},{"w":"gasprijzen?","s":3.92,"e":4.64,"p":0.906},{"w":"Dat","s":6.58,"e":7.06,"p":0.691},{"w":"klopt,","s":7.06,"e":7.54,"p":0.997},{"w":"hè?","s":7.88,"e":7.88,"p":0.828},{"w":"Ja,","s":8.4,"e":8.58,"p":0.328},{"w":"en","s":8.74,"e":8.84,"p":0.707},{"w":"waarom","s":8.84,"e":9.2,"p":0.999},{"w":"stijgt","s":9.2,"e":9.44,"p":0.986},{"w":"ze","s":9.44,"e":9.54,"p":0.982},{"w":"mee","s":9.54,"e":9.64,"p":0.747},{"w":"met","s":9.64,"e":9.82,"p":0.994},{"w":"de","s":9.82,"e":9.92,"p":0.985},{"w":"gasprijzen?","s":9.92,"e":10.2,"p":0.987},{"w":"Maar","s":11.28,"e":11.76,"p":0.352},{"w":"hoe","s":11.76,"e":12.0,"p":0.986},{"w":"wordt","s":12.0,"e":12.1,"p":0.997},{"w":"die","s":12.1,"e":12.28,"p":0.967},{"w":"prijs","s":12.28,"e":12.48,"p":0.998},{"w":"bepaald","s":12.48,"e":12.9,"p":0.998},{"w":"en","s":12.9,"e":13.14,"p":0.993},{"w":"waarom","s":13.14,"e":13.5,"p":0.999},{"w":"stijgt","s":13.5,"e":13.7,"p":0.998},{"w":"ze","s":13.7,"e":13.86,"p":0.99},{"w":"mee","s":13.86,"e":13.98,"p":0.968},{"w":"met","s":13.98,"e":14.14,"p":0.998},{"w":"de","s":14.14,"e":14.3,"p":0.996},{"w":"gasprijzen?","s":14.3,"e":14.9,"p":0.998}]},"IMG_0418":{"dur":10.9,"words":[{"w":"Electriciteit","s":0.66,"e":1.14,"p":0.845},{"w":"voeren","s":1.14,"e":1.42,"p":0.922},{"w":"we","s":1.42,"e":1.58,"p":0.989},{"w":"niet","s":1.58,"e":1.8,"p":0.992},{"w":"van","s":1.8,"e":2.0,"p":0.979},{"w":"ver","s":2.0,"e":2.22,"p":0.95},{"w":"in","s":2.22,"e":2.6,"p":0.971},{"w":"en","s":2.6,"e":2.84,"p":0.191},{"w":"wordt","s":2.84,"e":3.02,"p":0.799},{"w":"vooral","s":3.02,"e":3.28,"p":0.963},{"w":"bij","s":3.28,"e":3.4,"p":0.984},{"w":"onze","s":3.4,"e":3.68,"p":0.992},{"w":"buurlanden","s":3.68,"e":4.3,"p":0.987},{"w":"en","s":4.3,"e":4.36,"p":0.988},{"w":"bij","s":4.36,"e":4.52,"p":0.956},{"w":"ons","s":4.52,"e":4.76,"p":0.999},{"w":"geproduceerd","s":4.76,"e":5.44,"p":0.839},{"w":"met","s":5.44,"e":5.72,"p":0.959},{"w":"kern","s":5.72,"e":6.06,"p":0.849},{"w":"-","s":6.06,"e":6.16,"p":0.908},{"w":"en","s":6.16,"e":6.26,"p":0.995},{"w":"gascentrales","s":6.26,"e":7.12,"p":0.951},{"w":"en","s":7.12,"e":7.3,"p":0.828},{"w":"de","s":7.3,"e":7.48,"p":0.964},{"w":"laatste","s":7.48,"e":7.74,"p":0.803},{"w":"tijd","s":7.74,"e":7.86,"p":0.998},{"w":"ook","s":7.86,"e":8.22,"p":0.926},{"w":"veel","s":8.22,"e":8.38,"p":0.983},{"w":"meer","s":8.38,"e":8.58,"p":0.994},{"w":"zon","s":8.58,"e":8.92,"p":0.998},{"w":"en","s":8.92,"e":9.18,"p":0.997},{"w":"wind.","s":9.18,"e":9.34,"p":0.994}]}}
```

## Sjabloon

Zet beide bestanden letterlijk in `sjabloon/`: het CapCut-sjabloon waaruit de bouwer elk object kloont (stap 8). `<user>` vult de bouwer zelf in.

`sjabloon/draft_info.json`
```json
{"id": "1F5CAEAF-F8C2-413C-8620-5871E22D29FE", "version": 360000, "new_version": "183.0.0", "name": "", "duration": 62933333, "create_time": 1774554852992457, "update_time": 1781590758434092, "fps": 30.0, "is_drop_frame_timecode": false, "color_space": -1, "config": {"video_mute": false, "record_audio_last_index": 1, "extract_audio_last_index": 1, "original_sound_last_index": 10, "subtitle_recognition_id": "", "subtitle_taskinfo": [], "lyrics_recognition_id": "", "lyrics_taskinfo": [], "subtitle_sync": true, "lyrics_sync": true, "voice_change_sync": false, "sticker_max_index": 1, "adjust_max_index": 1, "material_save_mode": 0, "export_range": null, "maintrack_adsorb": true, "combination_max_index": 1, "attachment_info": [{"class_name": "AttachmentBusinessControl", "file_name": "draft.extra"}], "zoom_info_params": {"zoom_ratio": 1.0, "offset_x": 0.0, "offset_y": 0.0}, "system_font_list": ["7148719615961469441", "7167023658689368577", "6936059986288054786", "6936059969326289410", "6936059954742694402", "6936059869082423810", "6936059834206786049", "6936059789155766785"], "multi_language_mode": "none", "multi_language_main": "none", "multi_language_current": "none", "multi_language_list": [], "subtitle_keywords_config": null, "use_float_render": false}, "canvas_config": {"ratio": "9:16", "width": 1080, "height": 1920, "background": null}, "tracks": [{"id": "16E00272-DC93-4169-A333-0CF10D0E7AED", "type": "video", "segments": [{"id": "42B1DE9A-529D-4D7D-9B0E-A865C911E953", "source_timerange": {"start": 1566666, "duration": 1100000}, "target_timerange": {"start": 0, "duration": 1100000}, "render_timerange": {"start": 0, "duration": 0}, "desc": "", "state": 0, "speed": 1.0, "is_loop": false, "is_tone_modify": false, "reverse": false, "intensifies_audio": false, "cartoon": false, "volume": 1.0, "last_nonzero_volume": 1.0, "clip": {"scale": {"x": 1.343231543112485, "y": 1.343231543112485}, "rotation": 0.768207874442913, "transform": {"x": 0.0, "y": 0.0}, "flip": {"vertical": false, "horizontal": false}, "alpha": 1.0}, "uniform_scale": {"on": true, "value": 1.0}, "material_id": "1D30B6F4-1AB3-4639-9AF9-5AFA51F9B223", "extra_material_refs": ["8FF5525A-21A3-464F-82DC-ADF1938A2EF4", "E7B7D32A-2E0B-4F34-913C-20F8BBE8D724", "7215EA30-E6ED-4D0E-827C-B3068283B1F3", "34332116-F54E-4E42-BC96-9C019462E7F2", "0F513612-FE67-4860-BFFC-A6A4D397A100", "1E464D2E-5738-4D49-8ECA-B9C4C295078D", "D95E0B66-2C95-4725-AF1E-CF54335285F3", "83E3E969-6DC4-43EE-88E1-2A54258CD926"], "render_index": 0, "keyframe_refs": [], "enable_lut": true, "enable_adjust": true, "enable_hsl": false, "visible": true, "group_id": "", "enable_color_curves": true, "enable_hsl_curves": true, "track_render_index": 0, "hdr_settings": {"mode": 4, "intensity": 1.0, "nits": 1000}, "enable_color_wheels": true, "track_attribute": 0, "is_placeholder": false, "template_id": "", "enable_smart_color_adjust": false, "template_scene": "default", "common_keyframes": [], "caption_info": null, "responsive_layout": {"enable": false, "target_follow": "", "size_layout": 0, "horizontal_pos_layout": 0, "vertical_pos_layout": 0}, "enable_color_match_adjust": false, "enable_color_correct_adjust": false, "enable_adjust_mask": false, "raw_segment_id": "", "lyric_keyframes": null, "enable_video_mask": true, "digital_human_template_group_id": "", "color_correct_alg_result": "", "source": "segmentsourcenormal", "enable_mask_stroke": false, "enable_mask_shadow": false, "enable_color_adjust_pro": false, "segment_color_tag": ""}], "flag": 0, "attribute": 0, "name": "", "is_default_name": true}, {"id": "5B6147EC-8EF0-4D2B-B551-F2CCB28E708E", "type": "text", "segments": [{"id": "1514D113-F669-4D0B-A86B-3EABA91F6A08", "source_timerange": null, "target_timerange": {"start": 166666, "duration": 3633334}, "render_timerange": {"start": 0, "duration": 0}, "desc": "", "state": 0, "speed": 1.0, "is_loop": false, "is_tone_modify": false, "reverse": false, "intensifies_audio": false, "cartoon": false, "volume": 1.0, "last_nonzero_volume": 1.0, "clip": {"scale": {"x": 0.692481160163879, "y": 0.692481160163879}, "rotation": 0.0, "transform": {"x": 0.0, "y": -0.625}, "flip": {"vertical": false, "horizontal": false}, "alpha": 1.0}, "uniform_scale": {"on": true, "value": 1.0}, "material_id": "09244860-005F-4992-8D86-3FFA39862889", "extra_material_refs": ["CAE7B03C-1C16-4D7B-A976-AAB5A9514E7A"], "render_index": 14052, "keyframe_refs": [], "enable_lut": false, "enable_adjust": false, "enable_hsl": false, "visible": true, "group_id": "", "enable_color_curves": true, "enable_hsl_curves": true, "track_render_index": 3, "hdr_settings": null, "enable_color_wheels": true, "track_attribute": 0, "is_placeholder": false, "template_id": "", "enable_smart_color_adjust": false, "template_scene": "default", "common_keyframes": [], "caption_info": null, "responsive_layout": {"enable": false, "target_follow": "", "size_layout": 0, "horizontal_pos_layout": 0, "vertical_pos_layout": 0}, "enable_color_match_adjust": false, "enable_color_correct_adjust": false, "enable_adjust_mask": false, "raw_segment_id": "", "lyric_keyframes": null, "enable_video_mask": true, "digital_human_template_group_id": "", "color_correct_alg_result": "", "source": "segmentsourcenormal", "enable_mask_stroke": false, "enable_mask_shadow": false, "enable_color_adjust_pro": false, "segment_color_tag": ""}], "flag": 0, "attribute": 0, "name": "", "is_default_name": true}], "group_container": null, "materials": {"flowers": [], "videos": [{"id": "1D30B6F4-1AB3-4639-9AF9-5AFA51F9B223", "unique_id": "0400263F-B969-4E5E-8099-2E9C134C9149/L0/001", "type": "video", "duration": 102800000, "path": "##_draftpath_placeholder_0E685133-18CE-45ED-8CB8-2904A212EC80_##/video/IOS_ALBUM@0400263F-B969-4E5E-8099-2E9C134C9149_L0_001@1774522305000.mov", "media_path": "/var/mobile/Media/PhotoData/CPLAssets/group222/E10C6A5D-CD78-4D13-A894-AECCB9FB86D8.MOV", "local_id": "0400263F-B969-4E5E-8099-2E9C134C9149/L0/001", "has_audio": true, "reverse_path": "", "intensifies_path": "", "reverse_intensifies_path": "", "intensifies_audio_path": "", "cartoon_path": "", "width": 2160, "height": 3840, "category_id": "", "category_name": "", "material_id": "", "material_name": "", "material_url": "", "crop": {"upper_left_x": 0.0, "upper_left_y": 0.0, "upper_right_x": 1.0, "upper_right_y": 0.0, "lower_left_x": 0.0, "lower_left_y": 1.0, "lower_right_x": 1.0, "lower_right_y": 1.0}, "crop_ratio": "free", "audio_fade": null, "crop_scale": 1.0, "extra_type_option": 0, "stable": {"stable_level": 0, "matrix_path": "", "time_range": {"start": 0, "duration": 0}}, "matting": {"flag": 0, "path": "", "interactiveTime": [], "has_use_quick_brush": false, "strokes": [], "has_use_quick_eraser": false, "expansion": 0, "feather": 0, "reverse": false, "custom_matting_id": "", "enable_matting_stroke": false, "is_clould": false, "mask_video_path": "", "cloud_product_fps": 0.0}, "source": 0, "source_platform": 0, "formula_id": "", "check_flag": 63502335, "video_algorithm": {"algorithms": [], "time_range": null, "path": "", "gameplay_configs": [], "ai_in_painting_config": [], "complement_frame_config": null, "motion_blur_config": null, "deflicker": null, "noise_reduction": null, "quality_enhance": null, "super_resolution": null, "ai_background_configs": [], "smart_complement_frame": null, "aigc_generate": null, "aigc_generate_list": [], "mouth_shape_driver": null, "ai_expression_driven": null, "ai_motion_driven": null, "image_interpretation": null, "story_video_modify_video_config": {"task_id": "", "is_overwrite_last_video": false, "tracker_task_id": "", "generate_id": "", "generate_card_id": ""}, "skip_algorithm_index": []}, "is_unified_beauty_mode": false, "is_set_beauty_mode": false, "object_locked": null, "smart_motion": null, "multi_camera_info": null, "freeze": null, "picture_from": "none", "picture_set_category_id": "", "picture_set_category_name": "", "team_id": "", "local_material_id": "", "origin_material_id": "", "request_id": "", "has_sound_separated": false, "is_text_edit_overdub": false, "is_ai_generate_content": false, "aigc_type": "none", "is_copyright": false, "aigc_history_id": "", "aigc_item_id": "", "local_material_from": "", "smart_match_info": null, "beauty_face_preset_infos": [], "beauty_body_preset_id": "", "beauty_face_auto_preset": {"preset_id": "", "name": "", "rate_map": "", "scene": ""}, "beauty_face_auto_preset_infos": [], "beauty_body_auto_preset": null, "live_photo_timestamp": -1, "live_photo_cover_path": "", "content_feature_info": null, "corner_pin": null, "surface_trackings": [], "video_mask_stroke": {"resource_id": "", "path": "", "type": "", "color": "", "size": 0.0, "alpha": 0.0, "distance": 0.0, "texture": 0.0, "horizontal_shift": 0.0, "vertical_shift": 0.0}, "video_mask_shadow": {"resource_id": "", "path": "", "color": "", "alpha": 0.0, "blur": 0.0, "distance": 0.0, "angle": 0.0}, "pre_applied_vip_materials": [], "workflow_node_id": ""}], "tail_leaders": [], "audios": [], "images": [], "texts": [{"recognize_task_id": "32BF7124-1D32-4358-A187-4CF8059D99E8", "id": "09244860-005F-4992-8D86-3FFA39862889", "name": "", "recognize_text": "", "recognize_model": "", "punc_model": "", "type": "subtitle", "content": "{\"styles\": [{\"fill\": {\"alpha\": 1.0, \"content\": {\"render_type\": \"solid\", \"solid\": {\"alpha\": 1.0, \"color\": [1.0, 1.0, 1.0]}}}, \"font\": {\"id\": \"\", \"path\": \"/Users/<user>/Library/Containers/com.lemon.lvoverseas/Data/Library/Fonts/OpenSans-Bold.ttf\"}, \"range\": [0, 10], \"size\": 10, \"strokes\": [{\"alpha\": 1.0, \"content\": {\"render_type\": \"solid\", \"solid\": {\"alpha\": 1.0, \"color\": [0.0, 0.0, 0.0]}}, \"width\": 0.05999999865889549}], \"useLetterColor\": true}], \"text\": \"Ondertitel\"}", "base_content": "", "words": {"start_time": [], "end_time": [], "text": []}, "current_words": {"start_time": [], "end_time": [], "text": []}, "global_alpha": 1.0, "combo_info": {"text_templates": []}, "caption_template_info": {"resource_id": "", "third_resource_id": "", "resource_name": "", "category_id": "", "category_name": "", "effect_id": "", "request_id": "", "path": "", "is_new": false, "source_platform": 0}, "layer_weight": 1, "letter_spacing": 0.0, "text_curve": null, "text_loop_on_path": false, "offset_on_path": 0.0, "enable_path_typesetting": false, "text_exceeds_path_process_type": 0, "text_typesetting_paths": null, "text_typesetting_paths_file": "", "text_typesetting_path_index": 0, "line_spacing": 0.0, "has_shadow": false, "shadow_color": "#000000", "shadow_alpha": 0.649999976158142, "shadow_smoothing": 0.0, "shadow_distance": 8.0, "shadow_point": {"x": 1.018233764908628, "y": -1.018233764908628}, "shadow_angle": -45.0, "shadow_thickness_projection_enable": false, "shadow_thickness_projection_angle": 0.0, "shadow_thickness_projection_distance": 0.0, "border_alpha": 1.0, "border_color": "#000000", "border_width": 0.059999998658895, "border_mode": 0, "style_name": "white_text_black_shadow", "text_color": "#ffffff", "text_alpha": 1.0, "font_name": "", "font_title": "none", "font_size": 8.0, "font_path": "/Users/<user>/Library/Containers/com.lemon.lvoverseas/Data/Library/Fonts/OpenSans-Bold.ttf", "font_id": "", "font_resource_id": "", "initial_scale": 1.0, "font_url": "", "typesetting": 0, "alignment": 1, "line_feed": 1, "use_effect_default_color": true, "is_rich_text": false, "shape_clip_x": false, "shape_clip_y": false, "ktv_color": "", "text_to_audio_ids": [], "bold_width": 0.0, "italic_degree": 0, "underline": false, "underline_width": 0.05, "underline_offset": 0.22, "sub_type": 0, "check_flag": 47, "text_size": 30, "font_category_name": "", "font_source_platform": 0, "font_third_resource_id": "", "font_category_id": "", "add_type": 1, "operation_type": 0, "recognize_type": 0, "fonts": [], "background_color": "#000000", "background_alpha": 1.0, "background_style": 0, "background_round_radius": 0.0, "background_width": 0.14, "background_height": 0.14, "background_vertical_offset": 0.0, "background_horizontal_offset": 0.0, "background_fill": "", "single_char_bg_enable": false, "single_char_bg_color": "", "single_char_bg_alpha": 1.0, "single_char_bg_round_radius": 0.3, "single_char_bg_width": 0.0, "single_char_bg_height": 0.0, "single_char_bg_vertical_offset": 0.0, "single_char_bg_horizontal_offset": 0.0, "font_team_id": "", "tts_auto_update": false, "text_preset_resource_id": "", "group_id": "D99FD29E-7818-451E-8063-DF8C5833AF54", "preset_id": "", "preset_name": "", "preset_category": "", "preset_category_id": "", "preset_index": 0, "preset_has_set_alignment": false, "force_apply_line_max_width": false, "language": "en-US", "relevance_segment": [], "original_size": [], "fixed_width": -1.0, "fixed_height": -1.0, "autoAdaptCanvasEnabled": false, "line_max_width": 0.82, "oneline_cutoff": false, "cutoff_postfix": "", "subtitle_template_original_fontsize": 0.0, "subtitle_keywords": null, "inner_padding": -1.0, "multi_language_current": "none", "source_from": "", "is_lyric_effect": false, "lyric_group_id": "", "lyrics_template": {"resource_id": "", "resource_name": "", "panel": "", "effect_id": "", "path": "", "category_id": "", "category_name": "", "request_id": ""}, "is_batch_replace": false, "is_words_linear": false, "ssml_content": "", "subtitle_keywords_config": null, "sub_template_id": -1, "translate_original_text": ""}], "effects": [], "stickers": [], "canvases": [{"id": "7215EA30-E6ED-4D0E-827C-B3068283B1F3", "type": "canvas_color", "color": "", "blur": 0.0, "image": "", "album_image": "", "image_id": "", "image_name": "", "source_platform": 0, "team_id": ""}], "transitions": [], "audio_effects": [], "audio_fades": [], "beats": [], "material_animations": [{"id": "0F513612-FE67-4860-BFFC-A6A4D397A100", "type": "sticker_animation", "animations": [{"id": "6798320778182922760", "type": "in", "start": 0, "duration": 500000, "path": "/Users/<user>/Library/Containers/com.lemon.lvoverseas/Data/Movies/CapCut/User Data/Cache/effect/6798320778182922760/883ad04bd79b502aaa55b5d9b87175ea", "platform": "all", "resource_id": "6798320778182922760", "third_resource_id": "6798320778182922760", "source_platform": 1, "name": "Einblenden", "category_id": "2037708296", "category_name": "Trending1", "panel": "video", "material_type": "video", "anim_adjust_params": null, "request_id": "2026032701043417C635E29D16E668B028"}], "multi_language_current": "none"}, {"id": "CAE7B03C-1C16-4D7B-A976-AAB5A9514E7A", "type": "sticker_animation", "animations": [], "multi_language_current": "none"}], "placeholders": [], "placeholder_infos": [{"id": "E7B7D32A-2E0B-4F34-913C-20F8BBE8D724", "type": "placeholder_info", "meta_type": "none", "res_path": "", "res_text": "", "error_path": "", "error_text": ""}], "speeds": [{"id": "8FF5525A-21A3-464F-82DC-ADF1938A2EF4", "type": "speed", "mode": 0, "speed": 1.0, "curve_speed": null}], "common_mask": [], "chromas": [], "text_templates": [], "realtime_denoises": [{"id": "34332116-F54E-4E42-BC96-9C019462E7F2", "type": "realtime_denoise", "is_denoise": true, "denoise_mode": 0.0, "denoise_rate": 0.85, "path": "/Applications/CapCut.app/Contents/MacOS/../Resources/audiosami/unet_denoise_44k_music_model_v1.0.model", "sami_name": "denoise_v2", "sami_version": "1.0", "sami_type": 2, "is_from_hd_sounds": false}], "audio_pannings": [], "audio_pitch_shifts": [], "video_trackings": [], "hsl": [], "drafts": [], "color_curves": [], "hsl_curves": [], "primary_color_wheels": [], "log_color_wheels": [], "video_effects": [], "ai_text_effects": [], "audio_balances": [], "handwrites": [], "manual_deformations": [], "manual_beautys": [], "plugin_effects": [], "sound_channel_mappings": [{"id": "1E464D2E-5738-4D49-8ECA-B9C4C295078D", "type": "none", "audio_channel_mapping": 0, "is_config_open": false}], "green_screens": [], "shapes": [], "material_colors": [{"id": "D95E0B66-2C95-4725-AF1E-CF54335285F3", "is_color_clip": false, "is_gradient": false, "solid_color": "", "gradient_colors": [], "gradient_percents": [], "gradient_angle": 90.0, "width": 0.0, "height": 0.0}], "digital_humans": [], "digital_human_model_dressing": [], "smart_crops": [], "ai_translates": [], "audio_track_indexes": [], "loudnesses": [], "vocal_beautifys": [], "vocal_separations": [{"id": "83E3E969-6DC4-43EE-88E1-2A54258CD926", "type": "vocal_separation", "choice": 0, "removed_sounds": [], "time_range": null, "production_path": "", "final_algorithm": "", "enter_from": ""}], "smart_relights": [], "time_marks": [], "multi_language_refs": [], "video_shadows": [], "video_strokes": [], "video_radius": []}, "keyframes": {"videos": [], "audios": [], "texts": [], "stickers": [], "filters": [], "adjusts": [], "handwrites": [], "effects": []}, "keyframe_graph_list": [], "platform": {"os": "iOS", "os_version": "26.1", "app_id": 3006, "app_version": "17.1.0", "app_source": "cc", "device_id": "e4434467ad99b533376c00d46a84f3fd", "hard_disk_id": "f490b1de7e30a0535bc5b6c8d44756c4", "mac_address": "64458b3f0ec35d3886751f96a48aa54d"}, "last_modified_platform": {"os": "mac", "os_version": "26.5.1", "app_id": 359289, "app_version": "9.3.0", "app_source": "cc", "device_id": "b1eeb266a7ef4d8ae55c9fc99bab7547", "hard_disk_id": "", "mac_address": "25b1724f2ccb46177fae33eee989ba35"}, "mutable_config": null, "cover": null, "retouch_cover": null, "extra_info": {"text_to_video": null, "track_info": {"transfer_paths": ["edit"], "item_type": 0, "template_id": "", "tutorial_info": null, "split_screen_info": null, "subtitle_batch_editing_info": null, "smart_match_type": 0, "is_start_from_text": false}, "subtitle_fragment_info_list": []}, "relationships": [], "mixed_track_mode_on": false, "render_index_track_mode_on": true, "free_render_index_mode_on": false, "static_cover_image_path": "", "source": "default", "time_marks": null, "path": "", "lyrics_effects": [], "uneven_animation_template_info": {"composition": "", "content": "", "order": "", "sub_template_info_list": []}, "draft_type": "video", "smart_ads_info": {"page_from": "", "routine": "", "draft_url": ""}, "function_assistant_info": {"smart_rec_applied": false, "fixed_rec_applied": false, "auto_adjust": false, "auto_adjust_segid_list": [], "color_correction": false, "color_correction_segid_list": [], "enhance_quality": false, "smooth_slow_motion": false, "deflicker_segid_list": [], "video_noise_segid_list": [], "enhance_quality_segid_list": [], "smart_segid_list": [], "retouch": false, "retouch_segid_list": [], "enhande_voice": false, "enhance_voice_segid_list": [], "audio_noise_segid_list": [], "auto_caption": false, "auto_caption_segid_list": [], "auto_caption_template_id": "", "caption_opt": false, "caption_opt_segid_list": [], "eye_correction": false, "eye_correction_segid_list": [], "normalize_loudness": false, "normalize_loudness_segid_list": [], "normalize_loudness_audio_denoise_segid_list": [], "auto_adjust_fixed": false, "auto_adjust_fixed_value": 50.0, "color_correction_fixed": false, "color_correction_fixed_value": 50.0, "normalize_loudness_fixed": false, "enhande_voice_fixed": false, "retouch_fixed": false, "enhance_quality_fixed": false, "smooth_slow_motion_fixed": false, "fps": {"num": 0, "den": 1}}}
```

`sjabloon/draft_meta_info.json`
```json
{"cloud_draft_cover": false, "cloud_draft_sync": false, "cloud_package_completed_time": "2026-06-16T06:45:13+00:00", "draft_cloud_capcut_purchase_info": "{}", "draft_cloud_last_action_download": true, "draft_cloud_package_type": "", "draft_cloud_purchase_info": "{\n    \"benefitsPayStrategy\": null,\n    \"currencyCode\": \"\",\n    \"expireTime\": 0,\n    \"extraFreeAssetDetail\": null,\n    \"hasBuy\": false,\n    \"hasPurchased\": false,\n    \"hasUnlockByAd\": false,\n    \"oneOffInfo\": null,\n    \"price\": -1,\n    \"priceType\": -1,\n    \"productId\": \"\",\n    \"productInfos\": null,\n    \"promotionAmount\": 0,\n    \"promotionProductId\": \"\",\n    \"useNewModule\": false\n}\n", "draft_cloud_template_id": "", "draft_cloud_tutorial_info": "{\n}\n", "draft_cloud_videocut_purchase_info": "{\"template_type\":\"\",\"unlock_type\":\"\"}", "draft_cover": "draft_cover.jpg", "draft_deeplink_url": "", "draft_enterprise_info": {"draft_enterprise_extra": "", "draft_enterprise_id": "", "draft_enterprise_name": "", "enterprise_material": []}, "draft_fold_path": "/Users/<user>/Movies/CapCut/User Data/Projects/com.lveditor.draft/sjabloon", "draft_id": "6335D3C3-8483-4388-97C8-08826042FBA9", "draft_is_ae_produce": false, "draft_is_ai_packaging_used": false, "draft_is_ai_shorts": false, "draft_is_ai_translate": false, "draft_is_article_video_draft": false, "draft_is_cloud_temp_draft": false, "draft_is_from_deeplink": "false", "draft_is_infinite_canvas_draft": false, "draft_is_invisible": false, "draft_is_pippit_draft": false, "draft_is_web_article_video": false, "draft_materials": [{"type": 0, "value": [{"ai_group_type": "", "create_time": 0, "duration": 165459592, "enter_from": 0, "extra_info": "paulyudin-corporate-corporate-music-493490", "file_Path": "./audio/877E6D56-DB8A-4622-ACA7-3F6790DE267F.mp3", "height": 0, "id": "75A498E0-377B-4AE0-9E63-CE33DFEDD623", "import_time": 1781697714, "import_time_ms": -1, "item_source": 1, "material_color_tag": "", "md5": "", "metetype": "extract_music", "roughcut_time_range": {"duration": -1, "start": -1}, "sub_time_range": {"duration": -1, "start": -1}, "type": 0, "width": 0}]}, {"type": 1, "value": []}, {"type": 2, "value": []}, {"type": 3, "value": []}, {"type": 6, "value": []}, {"type": 7, "value": []}, {"type": 8, "value": []}], "draft_materials_copied_info": [], "draft_name": "sjabloon", "draft_need_rename_folder": false, "draft_new_version": "164.0.0", "draft_removable_storage_device": "", "draft_root_path": "/Users/<user>/Movies/CapCut/User Data/Projects/com.lveditor.draft", "draft_segment_extra_info": [], "draft_timeline_materials_size_": 430688929, "draft_type": "", "draft_web_article_video_enter_from": "", "pippit_avatar_url": "", "pippit_extra_info": "", "pippit_id": "", "pippit_user_name": "", "tm_draft_cloud_completed": "1781592099000", "tm_draft_cloud_entry_id": 7651879801203982864, "tm_draft_cloud_modified": 1781697714755580, "tm_draft_cloud_parent_entry_id": -1, "tm_draft_cloud_space_id": 7244241542884410369, "tm_draft_cloud_user_id": 6969229428362822657, "tm_draft_create": 1781697714433059, "tm_draft_modified": 1790584281997782, "tm_draft_removed": 0, "tm_duration": 62933333}
```

## Klantprofielen

Zet elk profiel letterlijk in `klanten/<klant>/` (`profiel.json` en `style.json`). Nieuwe klant: zie "Nieuwe klant
opzetten" en voeg het profiel hier toe.

### EnergieGenie (energie.be)

`klanten/energiegenie/profiel.json`
```json
{
 "klant": "energie.be (EnergieGenie)",
 "goedgekeurd": "27-10-2026 (huisstijl ondertitels), 29-10-2026 (positie TikTok-veilig)",
 "taal": "nl",
 "fps": 30,
 "ondertitels": {
  "style": "style.json",
  "beweging": "blok achter het actieve woord (sjabloon 7535406282551971088)",
  "lettertype": "FT Aktual Bold",
  "letters_per_eenheid": 18.3,
  "blok_kleur": "#FAFF96",
  "rand": "#221334:0.08",
  "tekst_kleur": null,
  "schaal": null,
  "max_tekens": 42
 },
 "platformen": {},
 "montage": {
  "push_in": true,
  "punch_in": 1.12
 },
 "expand": {
  "vk": "verenigd koninkrijk",
  "vs": "verenigde staten",
  "co2": "co twee"
 },
 "vaste_correcties": [
  ["voorschoten", "voorschotten"],
  ["fluvjes", "Fluvius"],
  ["al in energieprijs", "all-in energieprijs"],
  ["energieovername document", "energieovernamedocument"]
 ],
 "export": {
  "map": "~/Documents/EnergieGenie Exports",
  "naam": "Video {n} - {titel}",
  "resolutie": "1080p",
  "fps": 30
 },
 "notities": [
  "Lettertype FT Aktual Bold, gemeten op energie.be (FTAktualRegular/FTAktualBold). Op de test-Mac stond enkel 'FT Aktual Trial': voor publicatie de gelicentieerde versie installeren en die als font_path gebruiken.",
  "Blok #FAFF96 (rgb 250, 255, 150) en rand #221334 (tekstkleur energie.be), beide gemeten op energie.be.",
  "Sjabloon 7535406282551971088 (blok, hoofdletters), schaal 0,69 uit style.json."
 ]
}
```

`klanten/energiegenie/style.json` (uit CapCut geoogst). De paden naar de CapCut-map staan hier met `<user>`: de bouwer
vult je eigen gebruiker in (uit `projects_root`) en vervangt het lettertype door `font_path` uit de job.
```json
{"template_resource_id": "7535406282551971088", "template_name": "边框弹出Ⅰ", "track": {"id": "46D69FE4-6A92-4834-B764-99EC1B8A228C", "type": "text", "flag": 1, "attribute": 0, "name": "", "is_default_name": true}, "segment": {"id": "57F78209-D9A2-41B6-AB5E-A8D3B8A82C87", "source_timerange": null, "target_timerange": {"start": 0, "duration": 2166666}, "render_timerange": {"start": 0, "duration": 0}, "desc": "", "state": 0, "speed": 1.0, "is_loop": false, "is_tone_modify": false, "reverse": false, "intensifies_audio": false, "cartoon": false, "volume": 1.0, "last_nonzero_volume": 1.0, "clip": {"scale": {"x": 0.692481160163879, "y": 0.692481160163879}, "rotation": 0.0, "transform": {"x": 0.0, "y": -0.625}, "flip": {"vertical": false, "horizontal": false}, "alpha": 1.0}, "uniform_scale": {"on": true, "value": 1.0}, "material_id": "D5A56775-72B7-45B9-90DE-4A0AB2EAE6A2", "extra_material_refs": ["A9920C9C-7C18-4F71-941C-7C473411D147", "A3B6657D-FD47-4870-94DC-41D005C15903"], "render_index": 14002, "keyframe_refs": [], "enable_lut": false, "enable_adjust": false, "enable_hsl": false, "visible": true, "group_id": "", "enable_color_curves": true, "enable_hsl_curves": true, "track_render_index": 2, "hdr_settings": null, "hdr_vivid_settings": null, "enable_color_wheels": true, "track_attribute": 0, "is_placeholder": false, "template_id": "", "enable_smart_color_adjust": false, "template_scene": "default", "common_keyframes": [], "caption_info": null, "responsive_layout": {"enable": false, "target_follow": "", "size_layout": 0, "horizontal_pos_layout": 0, "vertical_pos_layout": 0}, "enable_color_match_adjust": false, "enable_color_correct_adjust": false, "enable_adjust_mask": false, "raw_segment_id": "", "lyric_keyframes": null, "enable_video_mask": true, "digital_human_template_group_id": "", "color_correct_alg_result": "", "source": "segmentsourcenormal", "enable_mask_stroke": false, "enable_mask_shadow": false, "enable_color_adjust_pro": false, "segment_color_tag": ""}, "template": {"id": "D5A56775-72B7-45B9-90DE-4A0AB2EAE6A2", "version": "1.0.0", "effect_id": "7535406282551971088", "resource_id": "7535406282551971088", "third_resource_id": "0", "name": "边框弹出Ⅰ", "type": "text_template_subtitle", "path": "/Users/<user>/Library/Containers/com.lemon.lvoverseas/Data/Movies/CapCut/User Data/Cache/effect/7535406282551971088/57bf8cbc692b9141d866d501a010f220", "category_id": "", "category_name": "", "platform": "all", "text_to_audio_ids": [], "source_platform": 1, "resources": [{"panel": "fonts", "path": "/Users/<user>/Library/Containers/com.lemon.lvoverseas/Data/Movies/CapCut/User Data/Cache/effect/7506401698856865040/a9335808de0a39843cc63fe75a3baf0d/font.ttf", "resource_id": "7506401698856865040", "source_platform": 1}, {"panel": "flower", "path": "/Users/<user>/Library/Containers/com.lemon.lvoverseas/Data/Movies/CapCut/User Data/Cache/effect/162739917/4ed86fb521450aefdd39eeaa424b5bba", "resource_id": "7317207872985633281", "source_platform": 0}, {"panel": "text", "path": "/Users/<user>/Library/Containers/com.lemon.lvoverseas/Data/Movies/CapCut/User Data/Cache/effect/125673105/296d05f252f6a19fd38f3fb9a69ed03e", "resource_id": "7265163695435551233", "source_platform": 0}], "formula_id": "", "text_info_resources": [{"id": "7DA85CA3-836C-475C-9063-28F544733036", "attach_info": {"start_time": 0, "duration": 2166666, "original_size_width": 365.714111328125, "original_size_height": 232.50003051757812, "clip": {"scale": {"x": 1.0, "y": 1.0}, "rotation": 0.0, "transform": {"x": 0.0, "y": 0.0}, "flip": {"vertical": false, "horizontal": false}, "alpha": 1.0}}, "text_material_id": "3846BB53-24FF-4A74-838E-C179CE2069AC", "extra_material_refs": ["A3B6657D-FD47-4870-94DC-41D005C15903", "A9920C9C-7C18-4F71-941C-7C473411D147"], "clip_type": "", "lyric_keyframes": [], "word_index": [], "order_in_layer": 0, "capital": ""}], "non_text_info_resources": [], "check_flag": 7, "text_template_preset_resource_id": "", "is_3d": false, "is_pre_rendered": false, "aigc_type": "none", "text_template_resource_type": "subtitle_template", "aigc_config": {"prompt": "", "seed": 0, "model": "", "font_item": {"id": "6E81FA15-AE9A-4013-A8CE-909D1E5B1D48", "resource_id": "", "third_resource_id": "", "category_id": "", "category_name": "", "source_platform": 0, "path": "", "effect_id": "", "title": "", "team_id": "", "file_uri": "", "request_id": ""}}, "request_id": "", "origin_word_info": {"text": "", "start_time": 0, "end_time": 0, "words": [], "keyword_ranges": []}, "current_word_info": {"text": "", "start_time": 0, "end_time": 0, "words": [], "keyword_ranges": []}, "is_dynamic_build": false, "is_ai_emoji": false, "is_lyric_effect": false, "lyric_group_id": "", "merge_content": "", "is_uneven_animation": false, "material_text_ranges": [], "ai_emoji_config": null, "preview_time": 0.1, "render_mode": 0}, "inner_text": {"recognize_task_id": "6ae0bc23d06a2801f16bad66_8_0", "id": "3846BB53-24FF-4A74-838E-C179CE2069AC", "name": "A16DDE7A-4F98-4C85-9C71-9E4626CF64E8", "recognize_text": "je gasprijs verandert voordurend", "recognize_model": "", "punc_model": "", "type": "text", "content": "{\"styles\":[{\"fill\":{\"content\":{\"solid\":{\"color\":[1,1,1]},\"render_type\":\"solid\"}},\"range\":[0,32],\"effectStyle\":{\"path\":\"/Users/<user>/Library/Containers/com.lemon.lvoverseas/Data/Movies/CapCut/User Data/Cache/effect/162739917/4ed86fb521450aefdd39eeaa424b5bba\",\"id\":\"7317207872985633281\"},\"size\":15,\"font\":{\"path\":\"/Users/<user>/Library/Containers/com.lemon.lvoverseas/Data/Library/Fonts/FTAktualTrial-Bold-BF65d80f45a9d93.otf\",\"id\":\"\"}}],\"text\":\"je gasprijs verandert voordurend\"}", "base_content": "", "words": {"start_time": [0, 120, 200, 840, 920, 1360, 1480], "end_time": [120, 120, 840, 840, 1360, 1360, 2160], "text": ["je", " ", "gasprijs", " ", "verandert", " ", "voordurend"]}, "current_words": {"start_time": [], "end_time": [], "text": []}, "global_alpha": 1.0, "combo_info": {"text_templates": []}, "caption_template_info": {"resource_id": "", "third_resource_id": "", "resource_name": "", "category_id": "", "category_name": "", "effect_id": "", "request_id": "", "path": "", "is_new": false, "source_platform": 0}, "layer_weight": 1, "letter_spacing": 0.0423806680215372, "text_curve": null, "text_loop_on_path": false, "offset_on_path": 0.0, "enable_path_typesetting": false, "text_exceeds_path_process_type": 0, "text_typesetting_paths": null, "text_typesetting_paths_file": "", "text_typesetting_path_index": 0, "line_spacing": 0.02, "has_shadow": false, "shadow_color": "", "shadow_alpha": 0.0, "shadow_smoothing": 0.0, "shadow_distance": 5.0, "shadow_point": {"x": 0.0, "y": 0.0}, "shadow_angle": -45.0, "shadow_thickness_projection_enable": false, "shadow_thickness_projection_angle": 0.0, "shadow_thickness_projection_distance": 0.0, "border_alpha": 0.0, "border_color": "", "border_width": 0.0, "border_mode": 0, "style_name": "", "text_color": "#ffffffff", "text_alpha": 1.0, "font_name": "", "font_title": "none", "font_size": 15.0, "font_path": "/Users/<user>/Library/Containers/com.lemon.lvoverseas/Data/Library/Fonts/FTAktualTrial-Bold-BF65d80f45a9d93.otf", "font_id": "", "font_resource_id": "", "initial_scale": 1.0, "font_url": "", "typesetting": 0, "alignment": 1, "line_feed": 1, "use_effect_default_color": true, "is_rich_text": false, "shape_clip_x": false, "shape_clip_y": false, "ktv_color": "", "text_to_audio_ids": [], "bold_width": 0.008, "italic_degree": 10, "underline": false, "underline_width": 0.05, "underline_offset": 0.22, "sub_type": 0, "check_flag": 63, "text_size": 30, "font_category_name": "", "font_source_platform": 0, "font_third_resource_id": "", "font_category_id": "", "add_type": 1, "operation_type": 0, "recognize_type": 0, "fonts": [{"id": "9D1E27C1-C97A-49BA-97D1-66531AE0020F", "resource_id": "7506401698856865040", "third_resource_id": "", "category_id": "", "category_name": "", "source_platform": 1, "path": "/Users/<user>/Library/Containers/com.lemon.lvoverseas/Data/Movies/CapCut/User Data/Cache/effect/7506401698856865040/a9335808de0a39843cc63fe75a3baf0d/font.ttf", "effect_id": "", "title": "", "team_id": "", "file_uri": "", "request_id": ""}], "background_color": "#FAFF96", "background_alpha": 1.0, "background_style": 1, "background_round_radius": 0.0, "background_width": 0.14, "background_height": 0.14, "background_vertical_offset": 0.0, "background_horizontal_offset": 0.0, "background_fill": "", "single_char_bg_enable": false, "single_char_bg_color": "", "single_char_bg_alpha": 1.0, "single_char_bg_round_radius": 0.3, "single_char_bg_width": 0.0, "single_char_bg_height": 0.0, "single_char_bg_vertical_offset": 0.0, "single_char_bg_horizontal_offset": 0.0, "font_team_id": "", "tts_auto_update": false, "text_preset_resource_id": "", "group_id": "Auto_1793113134932", "preset_id": "", "preset_name": "", "preset_category": "", "preset_category_id": "", "preset_index": 0, "preset_has_set_alignment": false, "force_apply_line_max_width": true, "language": "nl-NL", "relevance_segment": [], "original_size": [], "fixed_width": -1.0, "fixed_height": -1.0, "autoAdaptCanvasEnabled": false, "line_max_width": 0.82, "oneline_cutoff": false, "cutoff_postfix": "", "subtitle_template_original_fontsize": 0.0, "subtitle_keywords": null, "inner_padding": -1.0, "multi_language_current": "none", "source_from": "", "is_lyric_effect": false, "lyric_group_id": "", "lyrics_template": {"resource_id": "", "resource_name": "", "panel": "", "effect_id": "", "path": "", "category_id": "", "category_name": "", "request_id": ""}, "is_batch_replace": false, "is_words_linear": false, "ssml_content": "", "subtitle_keywords_config": null, "sub_template_id": -1, "translate_original_text": ""}, "refs": {"A9920C9C-7C18-4F71-941C-7C473411D147": ["material_animations", {"id": "A9920C9C-7C18-4F71-941C-7C473411D147", "type": "sticker_animation", "animations": [{"id": "", "type": "caption", "start": 0, "duration": 2166666, "path": "/Users/<user>/Library/Containers/com.lemon.lvoverseas/Data/Movies/CapCut/User Data/Cache/effect/125673105/296d05f252f6a19fd38f3fb9a69ed03e", "platform": "all", "resource_id": "7265163695435551233", "third_resource_id": "", "source_platform": 0, "name": "", "category_id": "", "category_name": "", "panel": "", "material_type": "sticker", "anim_adjust_params": null, "request_id": ""}], "multi_language_current": "none"}], "A3B6657D-FD47-4870-94DC-41D005C15903": ["effects", {"id": "A3B6657D-FD47-4870-94DC-41D005C15903", "effect_id": "", "resource_id": "7317207872985633281", "third_resource_id": "", "name": "", "report_name": "", "type": "text_effect", "sub_type": "none", "path": "/Users/<user>/Library/Containers/com.lemon.lvoverseas/Data/Movies/CapCut/User Data/Cache/effect/162739917/4ed86fb521450aefdd39eeaa424b5bba", "value": 0.0, "visible": true, "item_effect_type": 0, "category_id": "", "category_name": "", "category_key": "", "sub_category_id": "", "sub_category_name": "", "platform": "all", "apply_target_type": 0, "source_platform": 0, "version": "", "adjust_params": [], "time_range": null, "formula_id": "", "enable_skin_tone_correction": false, "algorithm_artifact_path": "", "intensity_key": "", "face_adjust_params": [], "exclusion_group": [], "panel_id": "", "bloom_params": null, "request_id": "", "color_match_info": null, "multi_language_current": "", "lumi_hub_path": "", "covering_relation_change": 0, "beauty_face_auto_preset_id": "", "beauty_body_auto_preset_id": "", "beauty_face_auto_retouch_info": {"face_id": [], "beauty_face_auto_retouch_id": ""}, "smart_color_mode": 0, "is_from_intelligent_quality": false}]}, "color": null, "font": "/Users/<user>/Library/Containers/com.lemon.lvoverseas/Data/Library/Fonts/FTAktualTrial-Bold-BF65d80f45a9d93.otf", "background": "#FAFF96", "stroke": "#221334:0.08"}
```

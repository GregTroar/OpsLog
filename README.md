# OpsLog

A modern, fast ham-radio logger for Windows — Log4OM-style entry, real-time CAT
for **OmniRig**, native **FlexRadio/SmartSDR**, native **Icom CI-V** (USB **and**
remote-over-internet, replacing RS-BA1) and **TCI** (SunSDR / Expert Electronics),
DX cluster with spot alerts, awards tracking, maps, contest logging, QSL
management and a QSL-card designer. Built with **Wails v2** (Go backend +
React/TypeScript frontend), **pure Go** (no CGO): SQLite for configuration,
optional **shared MySQL** for the logbook so several operators can run one log.
Fully themeable and bilingual (English / French).

Developed by **F4BPO**.

---

## Building / developing

- **Dev:** `wails dev` (Vite hot-reload; Go methods reachable at http://localhost:34115).
- **Build:** `wails build` (use the project's wails v2.11 — `~/go/bin/wails.exe`).
- **Regenerate Go↔TS bindings** after changing exported `App` methods:
  `wails generate module`.
- **Release:** `.vscode/release.ps1` (Ctrl+Shift+P → *Tasks: Run Task* →
  *Release OpsLog*) — bumps the version, pushes source to Gitea, builds the exe
  and publishes it to Gitea + GitHub releases.

---

## Logging

- **Log4OM-style entry strip:** callsign, RST tx/rx, name/QTH/grid, band/mode,
  TX/RX frequency (split), start/end time, comment/note. The contacted entity's
  **flag** is shown large next to the RST fields.
- **Callsign lookup** (QRZ.com / HamQTH) with photo, auto-fill of name/QTH/grid
  and the QRZ.com tab.
- **Offline DXCC** resolution from `cty.dat` (country, CQ/ITU zones, continent),
  with `/MM` `/AM` `/B` (beacon) and call-area (`/8`, `/W6`) handling, plus
  ClubLog DXpedition date overrides.
- **Recent QSOs**, **Worked-before** matrix (per band/mode slot), bulk re-resolve
  from cty/QRZ/ClubLog, bulk send to QSL services.
- **Advanced QSO filter builder** (field / operator / value, AND / OR, saved
  presets) with filtered- and selected-row **ADIF export**.
- **Find duplicates** (Tools) — groups QSOs by same call + band + mode (optionally
  same day / minute) and lets you pick which to delete.
- **ADIF 3.1.7 compliant** import/export: a full field dictionary, 30 promoted
  columns, a generic "extra fields" editor and standard/all export modes.
- **Profiles:** every setting is per-profile; each profile can point its logbook
  at the local SQLite file or a **shared MySQL** database (multi-operator).

## Maps & antenna

- **Main view = two configurable panes** (per profile, Settings → General →
  *Main view*): great-circle map, locator (street) map, the cluster grid, the
  worked-before grid, recent QSOs, the **FlexRadio controls**, the **Icom
  console** or the **Net control** panel.
- **Great-circle map** with short/long-path distance & azimuth, selectable
  basemaps (Light / Voyager / Street / Satellite, all key-free and labelled) and
  the **antenna beam lobe(s)** drawn from the rotor azimuth.
- **Rotor compass** (azimuthal-equidistant, click-to-turn) driven by PstRotator.
- **Ultrabeam** support (Normal / 180° reverse / Bidirectional): the radiating
  direction is shown in green and the **mechanical boom** in grey, on both the
  compass and the map, so you never lose track of where the antenna points.

## DX Cluster

- Multiple cluster servers with auto-reconnect, a master for commands.
- **Filter sidebar** (callsign search, hide-worked, group duplicates, band /
  mode / status / source) shared by the Cluster tab and the Main-view cluster
  pane, with a show/hide toggle.
- Per-spot **status** (new / new-band / new-slot / worked), click-to-tune the
  rig, and a multi-band **Band Map** (panadapter-style strips).
- **POTA** spots are tagged with their park reference (via `api.pota.app`).
- **Spot alerts** (Log4OM-style): rules on call / country / band / mode /
  spotter, with sound, visual and e-mail notification (Tools → *Alert
  management*).

## CAT control

Four native backends (Settings → CAT), each with auto-reconnect and a fast,
non-blocking connect so a powered-off radio never freezes the app:

- **OmniRig** (Rig 1/2, hot-swap) — works with any OmniRig-supported rig.
- **FlexRadio (SmartSDR)** over the radio's TCP API — real-time slice freq /
  mode / split, UDP discovery, and **panadapter spots** (cluster spots pushed to
  the Flex display, click → fill the call).
- **Icom CI-V** — native, over the radio's **USB** port *or* over the internet
  via the radio's **built-in LAN server** (see *Remote Icom* below). No RS-BA1 or
  Remote Utility needed.
- **TCI** (WebSocket) — SunSDR / ExpertSDR2 and any TCI-compatible server:
  freq / mode / PTT / split, plus optional panorama spots.

Mode is taken from the radio; the digital sub-mode (FT4 vs FT8) is inferred from
the frequency. **Per-band Flex RX/TX antennas** can be configured and are applied
automatically on band change.

### FlexRadio control tab (SmartSDR-style)

Shown only when the CAT backend is a FlexRadio:

- **Transmit:** RF power, tune power, TUNE, MOX, speech processor (NOR/DX/DX+),
  VOX (+ level + delay), monitor (+ level), mic gain.
- **Receive (active slice):** AGC mode/threshold, audio level, NB / NR / ANF.
- **Antenna tuner (ATU):** tune / bypass / memories.
- **Amplifier:** the amp card follows whichever amplifier is configured —
  **PowerGenius XL** (operate/standby, fan mode, fault) or **SPE Expert**
  (operate/standby, ON/OFF, Low/Mid/High level, output-power bar, band &
  temperature).
- **Live meters** over the UDP VITA-49 stream: S-meter (S-units), forward power
  (W), SWR, ALC, PA temperature, voltage, plus the amplifier's meters.

### Icom control tab

Shown when the CAT backend is Icom (USB or network). A full RS-BA1-style console:

- **Twin VFO readout** (MAIN / SUB) with the big tabular frequency, mode badge,
  band and RIT/ΔTX offset, and a **mode-button row** (SSB / CW / RTTY / PSK /
  AM / FM).
- **Spectrum scope + waterfall** (panadapter): ON/OFF, CTR/FIX, double-click to
  tune, and **◀ ⊙ ▶** buttons to centre the scope on the current frequency
  (±50 kHz) and pan left/right.
- **Live meters** always visible: S-meter (click → fill RST), power in watts, SWR.
- **Receive DSP:** AF / RF gain, squelch, AGC, preamp, attenuator, filter
  (FIL1/2/3), NB, NR, ANF and — **on CW only** — the **APF** (audio peak filter).
- **Passband / notch:** Twin PBT (inner / outer), manual notch + position.
- **Transmit:** RF power, MOX, TUNE, **split with an automatic offset**
  (+5 kHz on SSB, +1 kHz on CW), and monitor. On **voice modes only**: mic gain,
  speech compressor, VOX (+ gain + anti-VOX). Controls that don't apply to the
  current mode are hidden automatically.
- **Bands & antenna:** one-touch band buttons and ANT1/ANT2 selection.
- **Clarifiers:** RIT and ΔTX with wheel / ± tuning (Ctrl+←/→ nudges RIT).
- **Power ON / OFF** buttons (manual by design — the app never wakes the rig on
  connect).
- **CW keying** can run through the radio's own keyer (see *Keyers* below).

### Remote Icom (over the internet, no RS-BA1)

OpsLog speaks the IC-7610's built-in network protocol directly — it **replaces
both the Icom Remote Utility and RS-BA1**. Enter the radio's IP, the Network
User1 name/password and the CI-V address, and the whole Icom console works over
the LAN/internet: login + token (auto-renewed), CI-V tunnel, receive-side
retransmit for a rock-solid link even with the panadapter streaming, and manual
power ON/OFF. (Audio is out of scope — use the radio in USB + a voice link such
as Mumble.)

## Keyers & audio

- **CW keyer** with macros and F-key macros. The keyer engine is selectable:
  **WinKeyer** (K1EL WK1/2/3 over a COM port), **FlexRadio CWX** (the radio's
  built-in keyer over the SmartSDR API — type-ahead and backspace, no WinKeyer or
  SmartCAT needed), **Icom** (the radio's own keyer over CI-V — no extra hardware,
  works over the remote link too) or **TCI**.
- **Digital Voice Keyer** (DVK): record F1–F6 voice messages and transmit them.
- **QSO audio recording:** continuous rolling capture; on *Log QSO* the contact
  is saved to a per-QSO WAV (`CALL_YYYYMMDD_HHMMSS.wav`); mixes RX + mic.

## Amplifiers & switches

- **Amplifiers** (Settings → Amplifier — the control card appears on the
  FlexRadio tab, or on its own when neither Flex nor Icom is active):
  - **PowerGenius XL** (4O3A) over direct TCP — operate/standby, fan-mode
    selector and fault display.
  - **SPE Expert** (1.3K-FA / 1.5K-FA / 2K-FA) over **USB** (virtual COM) or the
    **network** (RS232-to-Ethernet bridge) — operate/standby, ON/OFF,
    Low / Mid / High output level, an output-power bar and live status (band,
    SWR, PA current, temperature, warnings/alarms).
- **Antenna Genius** (4O3A) antenna switch over TCP/GSCP — a docked A/B
  antenna-switch widget.
- **Station Control** panel (dockable, drag-to-reorder widgets): the **rotator**,
  **Ultrabeam** element control and **relay boards** — WebSwitch 1216H, KMTronic,
  **Denkovi** USB (FT245 D2XX bit-bang, 4 or 8 relays) and generic USB-serial
  (CH340 / LCUS, A0 protocol) — for station power, antennas and accessories.
- **Relay auto-control** (Settings): switch Station-Control relays automatically
  from the rig frequency / band (like PstRotator) — per relay, a frequency window
  or a set of bands.

## QSL & awards

- **Awards engine:** built-in + custom award definitions (shared **globally**
  across profiles) — DXCC, WAS / WAZ / WAC, WPX, IOTA / POTA / SOTA / WWFF,
  **DDFM**, worked/confirmed/validated by band & mode, OR rules and manual
  reference assignment, live reference detection on call entry, **reference-list
  import** for totals/names, and a **Rescan** that re-pulls the logbook (picks up
  fresh LoTW/QRZ confirmations).
- **QSL services:** ClubLog (batched ADIF upload), LoTW, QRZ.com, eQSL — upload
  and **confirmation download** (which auto-refreshes the award stats).
- **QSL Card Designer** (see below).
- **E-mail eQSL:** right-click a QSO → *Send eQSL by e-mail* via the configured
  SMTP account. (Outlook/Hotmail disable basic-auth SMTP — use Gmail with an app
  password, or a Microsoft app password.)

## Contest logging

- **Contest tab:** pick a contest (built-in ADIF `CONTEST_ID` list) and an
  exchange (running serial or a fixed exchange). OpsLog auto-fills `CONTEST_ID`
  and the sent/received serials (`STX` / `SRX`), enforces a window start/end,
  flags dupes and keeps a live scoreboard.

## Statistics

- **Logbook statistics dashboard:** headline tiles (QSOs, unique callsigns, DXCC
  entities, continents, % confirmed) plus charts — QSOs **by mode**, a per-band
  **CW / phone / data** split, **activity over time** (rolling day / 7-day /
  30-day / 12-month views), by operator and by continent. Date-range, per-operator
  and per-contest filters, and a **Table** view that mirrors every chart.

## Multi-operator live status (special events)

For a multi-op special-event call on a shared MySQL logbook (e.g. **TM74TFR**):
Settings → General → *Publish live operator status*. Each OpsLog instance
heartbeats its current activity (operator call, band, frequency, mode) into a
`live_status` table every ~15 s. A small PHP renderer
([`docs/livestatus/tm74-status.php`](docs/livestatus/tm74-status.php)) on your
own web server reads that table and produces a live page/image you can embed on
the station's **QRZ.com** bio (`<img src="…/tm74-status.php?img=1">`). OpsLog
only writes to the DB — it is not a web server.

## Net control

- **Directed-net logging** (Tools → Net): a global roster (`nets.json`) plus an
  in-memory active session — check stations in, then log the whole net at once
  using the CAT frequency.

## Appearance & language

- **Themes:** four complete themes (Warm light, Warm dark, Graphite dark, High
  contrast) plus **Auto** (follows the OS light/dark preference), selectable in
  Settings → General. Every panel and every AG-Grid table follows the theme.
- **Bilingual:** full **English / French** UI, with a first-run flag chooser and
  a switcher in Settings → General.

## Security

- **Secret vault:** opt-in passphrase encryption of the stored passwords
  (AES-GCM + PBKDF2). Encrypted values are portable; a single unlock prompt at
  launch decrypts them for the session.

## Integrations (outbound)

- **UDP emitters:** push the current frequency to **PstRotator**, radio info in
  **N1MM `RadioInfo`** format, or an **ADIF record on each logged QSO** — so
  external tools (rotator control, digital apps, other loggers) stay in sync.

## Other

- **Autostart:** launch external programs (WSJT-X, JTAlert, rotator control…) at
  OpsLog startup, skipping any already running.
- **Backup:** optional database + ADIF backup at shutdown.
- **Update check** at startup and every 5 minutes (and on opening Help → About),
  with a toast (toggleable), plus a **What's new** dialog that shows the changelog
  (English / French) on the first launch after an update — reopenable any time
  from the Help menu.
- **Anonymous usage telemetry** (a once-a-day heartbeat: random install ID +
  version + OS — no callsign or QSO data; opt-out in Preferences).

---

## QSL Card Designer

Tools → *QSL Card Designer…* turns a few photos into a polished eQSL card:

1. Pick 1–6 photos (jpeg/png). OpsLog analyzes them offline (detail/luminance
   grid) and proposes **3 designs** — callsign in the calmest zone of the best
   photo, operator name, CQ/ITU zones + locator line, country flag, the other
   photos as bordered inserts, and a per-QSO confirmation box.
2. Pick a proposal and fine-tune it: click an element to select, drag to move,
   change font / style preset (gel gold, gel silver, classic white outline,
   script, flat) and per-preset knobs in the right panel.
3. Save the template (photos are copied into `data/qsl/templates/<id>/`, so the
   originals can move). One template can be the default per profile.

Sending: right-click a QSO → *Send eQSL by e-mail*. The card is rendered with
that QSO's data, rasterized to a ≤ 800 KB JPEG, archived in `data/qsl/outbox/`
and sent through the configured SMTP account to the address found by the
QRZ/HamQTH lookup. On success the QSO is stamped `EQSL_SENT=Y` (ADIF). The
e-mail subject/body templates live in the designer
(`{CALL} {DATE} {BAND} {MODE} {MYCALL}` variables).

Fonts: Archivo Black, Lilita One, Baloo 2, Oswald, Great Vibes, Allura (all
OFL, embedded — licenses in `internal/qslcard/assets/fonts/`); Cooper Black is
offered when MS Office installed it. Flags: flag-icons (MIT), embedded for the
commonly-worked DXCC entities.

---

## Data & storage

- **Config** (settings, profiles, rigs/antennas, cluster nodes, lookup cache,
  award lists, QSL templates) always lives in the local SQLite file under
  `data/` — instant even when the logbook is on a far-away MySQL.
- **Logbook** (QSOs) lives where the active profile points it: the local SQLite
  file or a per-profile shared **MySQL** database.

---

--------------------------------------------------------------------------------------------------

# OpsLog

Un logiciel de log radioamateur moderne et rapide pour Windows — saisie façon
Log4OM, CAT en temps réel pour **OmniRig**, **FlexRadio/SmartSDR** natif,
**Icom CI-V** natif (USB **et** à distance par internet, en remplacement de
RS-BA1) et **TCI** (SunSDR / Expert Electronics), cluster DX avec alertes de
spots, suivi des diplômes, cartes, log de concours, gestion des QSL et un
concepteur de cartes QSL. Construit avec **Wails v2** (backend Go + frontend
React/TypeScript), **100 % Go** (sans CGO) : SQLite pour la configuration,
**MySQL partagé** optionnel pour le journal afin que plusieurs opérateurs
partagent un même log. Entièrement thémable et bilingue (anglais / français).

Développé par **F4BPO**.

---

## Compilation / développement

- **Dev :** `wails dev` (rechargement à chaud Vite ; méthodes Go accessibles sur http://localhost:34115).
- **Build :** `wails build` (utiliser la version wails v2.11 du projet — `~/go/bin/wails.exe`).
- **Régénérer les bindings Go↔TS** après modification des méthodes `App`
  exportées : `wails generate module`.
- **Release :** `.vscode/release.ps1` (Ctrl+Maj+P → *Tasks: Run Task* →
  *Release OpsLog*) — incrémente la version, pousse le source sur Gitea, compile
  l'exe et le publie sur les releases Gitea + GitHub.

---

## Journalisation

- **Bandeau de saisie façon Log4OM :** indicatif, RST émis/reçu, nom/QTH/locator,
  bande/mode, fréquence TX/RX (split), heure de début/fin, commentaire/note. Le
  **drapeau** de l'entité contactée est affiché en grand à côté des champs RST.
- **Recherche d'indicatif** (QRZ.com / HamQTH) avec photo, pré-remplissage du
  nom/QTH/locator et onglet QRZ.com.
- **Résolution DXCC hors ligne** depuis `cty.dat` (pays, zones CQ/ITU,
  continent), avec gestion des `/MM` `/AM` `/B` (balise) et des changements de
  district (`/8`, `/W6`), plus les dérogations DXpédition de ClubLog par dates.
- **QSO récents**, matrice **déjà contactés** (par créneau bande/mode),
  re-résolution en masse depuis cty/QRZ/ClubLog, envoi en masse vers les services
  QSL.
- **Constructeur de filtres QSO avancé** (champ / opérateur / valeur, ET / OU,
  préréglages enregistrés) avec **export ADIF** des lignes filtrées ou
  sélectionnées.
- **Recherche de doublons** (Outils) — regroupe les QSO par même indicatif +
  bande + mode (optionnellement même jour / minute) et permet de choisir lesquels
  supprimer.
- **Conforme ADIF 3.1.7** en import/export : dictionnaire complet des champs,
  30 colonnes promues, éditeur générique de « champs supplémentaires » et modes
  d'export standard/complet.
- **Profils :** chaque réglage est par profil ; chaque profil peut pointer son
  journal vers le fichier SQLite local ou une base **MySQL partagée**
  (multi-opérateur).

## Cartes & antenne

- **Vue principale = deux volets configurables** (par profil, Réglages → Général
  → *Vue principale*) : carte grand-cercle, carte locator (rue), la grille du
  cluster, la grille des déjà-contactés, les QSO récents, les **commandes
  FlexRadio**, la **console Icom** ou le panneau **Net control**.
- **Carte grand-cercle** avec distance & azimut trajet court/long, fonds de carte
  sélectionnables (Light / Voyager / Street / Satellite, tous sans clé et
  légendés) et le(s) **lobe(s) du faisceau d'antenne** tracés depuis l'azimut du
  rotor.
- **Compas de rotor** (azimutal équidistant, clic pour tourner) piloté par
  PstRotator.
- **Support Ultrabeam** (Normal / inversé 180° / bidirectionnel) : la direction
  rayonnée est en vert et le **boom mécanique** en gris, à la fois sur le compas
  et sur la carte, pour toujours savoir où pointe l'antenne.

## Cluster DX

- Plusieurs serveurs de cluster avec reconnexion auto, un maître pour les
  commandes.
- **Barre latérale de filtres** (recherche d'indicatif, masquer-déjà-contactés,
  grouper les doublons, bande / mode / statut / source) partagée entre l'onglet
  Cluster et le volet cluster de la vue principale, avec bascule affichage/masquage.
- **Statut** par spot (nouveau / nouvelle bande / nouveau créneau / contacté),
  clic pour accorder la radio, et une **bandmap** multi-bandes (bandes façon
  panadapter).
- Les spots **POTA** sont étiquetés avec leur référence de parc (via
  `api.pota.app`).
- **Alertes de spots** (façon Log4OM) : règles sur indicatif / pays / bande /
  mode / spotter, avec notification sonore, visuelle et e-mail (Outils →
  *Gestion des alertes*).

## Contrôle CAT

Quatre backends natifs (Réglages → CAT), chacun avec reconnexion auto et une
connexion rapide non bloquante — une radio éteinte ne fige jamais l'application :

- **OmniRig** (Rig 1/2, changement à chaud) — fonctionne avec toute radio
  supportée par OmniRig.
- **FlexRadio (SmartSDR)** via l'API TCP de la radio — fréquence / mode / split
  de la slice en temps réel, découverte UDP, et **spots panadapter** (les spots
  du cluster poussés sur l'écran Flex, clic → remplir l'indicatif).
- **Icom CI-V** — natif, via le port **USB** de la radio *ou* par internet via le
  **serveur LAN intégré** de la radio (voir *Icom à distance* ci-dessous). Ni
  RS-BA1 ni Remote Utility nécessaires.
- **TCI** (WebSocket) — SunSDR / ExpertSDR2 et tout serveur compatible TCI :
  fréquence / mode / PTT / split, plus spots panorama optionnels.

Le mode est lu depuis la radio ; le sous-mode numérique (FT4 vs FT8) est déduit
de la fréquence. Les **antennes RX/TX Flex par bande** sont configurables et
appliquées automatiquement au changement de bande.

### Onglet de commande FlexRadio (façon SmartSDR)

Affiché uniquement quand le backend CAT est une FlexRadio :

- **Émission :** puissance RF, puissance d'accord, TUNE, MOX, processeur de
  parole (NOR/DX/DX+), VOX (+ niveau + délai), moniteur (+ niveau), gain micro.
- **Réception (slice active) :** mode/seuil AGC, niveau audio, NB / NR / ANF.
- **Coupleur d'antenne (ATU) :** accord / bypass / mémoires.
- **Amplificateur :** la carte de commande suit l'ampli configuré —
  **PowerGenius XL** (operate/standby, mode ventilateur, défaut) ou
  **SPE Expert** (operate/standby, Marche/Arrêt, niveau Low/Mid/High, barre de
  puissance de sortie, bande & température).
- **Mesures en direct** via le flux UDP VITA-49 : S-mètre (unités S), puissance
  directe (W), ROS, ALC, température PA, tension, plus les mesures de l'ampli.

### Onglet de commande Icom

Affiché quand le backend CAT est Icom (USB ou réseau). Une console complète façon
RS-BA1 :

- **Double afficheur VFO** (MAIN / SUB) avec la grande fréquence tabulaire, le
  badge de mode, la bande et l'offset RIT/ΔTX, et une **rangée de boutons de
  mode** (SSB / CW / RTTY / PSK / AM / FM).
- **Scope de spectre + waterfall** (panadapter) : ON/OFF, CTR/FIX, double-clic
  pour accorder, et boutons **◀ ⊙ ▶** pour centrer le scope sur la fréquence
  actuelle (±50 kHz) et le décaler à gauche/droite.
- **Mesures en direct** toujours visibles : S-mètre (clic → remplir le RST),
  puissance en watts, ROS.
- **DSP réception :** gain AF / RF, squelch, AGC, préampli, atténuateur, filtre
  (FIL1/2/3), NB, NR, ANF et — **en CW seulement** — l'**APF** (filtre de pic
  audio).
- **Passe-bande / notch :** Twin PBT (intérieur / extérieur), notch manuel +
  position.
- **Émission :** puissance RF, MOX, TUNE, **split avec offset automatique**
  (+5 kHz en SSB, +1 kHz en CW), et moniteur. **En phonie seulement** : gain
  micro, processeur de parole, VOX (+ gain + anti-VOX). Les commandes qui ne
  s'appliquent pas au mode courant sont masquées automatiquement.
- **Bandes & antenne :** boutons de bande en un clic et sélection ANT1/ANT2.
- **Clarificateurs :** RIT et ΔTX avec accord molette / ± (Ctrl+←/→ décale le
  RIT).
- Boutons **Marche / Arrêt** (manuels par choix — l'application ne réveille
  jamais la radio à la connexion).
- La **manipulation CW** peut passer par le keyer intégré de la radio (voir
  *Keyers* ci-dessous).

### Icom à distance (par internet, sans RS-BA1)

OpsLog parle directement le protocole réseau intégré de l'IC-7610 — il
**remplace à la fois l'Icom Remote Utility et RS-BA1**. Saisissez l'IP de la
radio, le nom/mot de passe Network User1 et l'adresse CI-V, et toute la console
Icom fonctionne sur le LAN/internet : login + token (renouvelé automatiquement),
tunnel CI-V, retransmission côté réception pour une liaison très stable même
avec le panadapter en flux, et Marche/Arrêt manuel. (L'audio est hors périmètre
— utilisez la radio en USB + une liaison voix comme Mumble.)

## Keyers & audio

- **Keyer CW** avec macros et macros sur touches F. Le moteur du keyer est
  sélectionnable : **WinKeyer** (K1EL WK1/2/3 sur port COM), **FlexRadio CWX**
  (le keyer intégré de la radio via l'API SmartSDR — type-ahead et retour arrière,
  sans WinKeyer ni SmartCAT), **Icom** (le keyer intégré de la radio via CI-V —
  sans matériel supplémentaire, fonctionne aussi à distance) ou **TCI**.
- **Keyer vocal numérique** (DVK) : enregistrer les messages vocaux F1–F6 et les
  émettre.
- **Enregistrement audio des QSO :** capture continue en tampon glissant ; au
  *Log QSO* le contact est sauvegardé dans un WAV par QSO
  (`INDICATIF_AAAAMMJJ_HHMMSS.wav`) ; mixe RX + micro.

## Amplis & commutateurs

- **Amplificateurs** (Réglages → Amplificateur — la carte de commande apparaît
  sur l'onglet FlexRadio, ou seule quand ni Flex ni Icom n'est actif) :
  - **PowerGenius XL** (4O3A) en TCP direct — operate/standby, sélecteur de mode
    ventilateur et affichage des défauts.
  - **SPE Expert** (1.3K-FA / 1.5K-FA / 2K-FA) via **USB** (COM virtuel) ou le
    **réseau** (pont RS232-Ethernet) — operate/standby, Marche/Arrêt, niveau de
    sortie Low / Mid / High, une barre de puissance et le statut en direct (bande,
    ROS, courant PA, température, avertissements/alarmes).
- Commutateur d'antenne **Antenna Genius** (4O3A) via TCP/GSCP — un widget de
  commutation A/B ancré.
- Panneau **Station Control** (ancrable, widgets réordonnables par glisser) : le
  **rotor**, la commande d'éléments **Ultrabeam** et des **cartes relais** —
  WebSwitch 1216H, KMTronic, **Denkovi** USB (bit-bang FT245 D2XX, 4 ou 8 relais)
  et USB-série générique (CH340 / LCUS, protocole A0) — pour l'alimentation, les
  antennes et accessoires de la station.
- **Relais automatiques** (Réglages) : bascule les relais de Station Control
  automatiquement selon la fréquence / bande de la radio (comme PstRotator) — par
  relais, une fenêtre de fréquence ou un ensemble de bandes.

## QSL & diplômes

- **Moteur de diplômes :** définitions intégrées + personnalisées (partagées
  **globalement** entre profils) — DXCC, WAS / WAZ / WAC, WPX,
  IOTA / POTA / SOTA / WWFF, **DDFM**, contacté/confirmé/validé par bande & mode,
  règles OU et affectation manuelle de références, détection de référence en
  direct à la saisie de l'indicatif, **import de listes de références** pour les
  totaux/noms, et un **Rescan** qui relit le journal (récupère les nouvelles
  confirmations LoTW/QRZ).
- **Services QSL :** ClubLog (upload ADIF par lots), LoTW, QRZ.com, eQSL —
  upload et **téléchargement des confirmations** (qui rafraîchit automatiquement
  les stats de diplômes).
- **Concepteur de cartes QSL** (voir ci-dessous).
- **eQSL par e-mail :** clic droit sur un QSO → *Envoyer l'eQSL par e-mail* via
  le compte SMTP configuré. (Outlook/Hotmail désactivent le SMTP basic-auth —
  utilisez Gmail avec un mot de passe d'application, ou un mot de passe
  d'application Microsoft.)

## Log de concours

- **Onglet Contest :** choisissez un concours (liste ADIF `CONTEST_ID` intégrée)
  et un échange (numéro de série courant ou échange fixe). OpsLog remplit
  automatiquement `CONTEST_ID` et les numéros émis/reçus (`STX` / `SRX`), impose
  un début/fin de fenêtre, signale les doublons et tient un tableau de score en
  direct.

## Statistiques

- **Tableau de bord des statistiques :** tuiles de synthèse (QSO, indicatifs
  uniques, entités DXCC, continents, % confirmés) plus des graphiques — QSO **par
  mode**, un split **CW / phonie / data** par bande, l'**activité dans le temps**
  (vues glissantes jour / 7 jours / 30 jours / 12 mois), par opérateur et par
  continent. Filtres par plage de dates, par opérateur et par concours, et une vue
  **Table** qui reprend chaque graphique.

## Statut opérateur en direct (événements spéciaux)

Pour un indicatif d'événement spécial multi-op sur un journal MySQL partagé (ex.
**TM74TFR**) : Réglages → Général → *Publier le statut opérateur en direct*.
Chaque instance OpsLog envoie un battement de cœur de son activité (indicatif de
l'opérateur, bande, fréquence, mode) dans une table `live_status` toutes les
~15 s. Un petit rendu PHP
([`docs/livestatus/tm74-status.php`](docs/livestatus/tm74-status.php)) sur votre
propre serveur web lit cette table et produit une page/image en direct que vous
pouvez intégrer sur la bio **QRZ.com** de la station
(`<img src="…/tm74-status.php?img=1">`). OpsLog écrit seulement dans la base —
ce n'est pas un serveur web.

## Net control

- **Log de net dirigé** (Outils → Net) : un roster global (`nets.json`) plus une
  session active en mémoire — pointez les stations présentes, puis loguez tout le
  net d'un coup en utilisant la fréquence CAT.

## Apparence & langue

- **Thèmes :** quatre thèmes complets (Clair chaud, Sombre chaud, Graphite
  sombre, Contraste élevé) plus **Auto** (suit la préférence clair/sombre du
  système), sélectionnables dans Réglages → Général. Chaque panneau et chaque
  table AG-Grid suit le thème.
- **Bilingue :** interface complète **anglais / français**, avec un choix de
  drapeau au premier lancement et un sélecteur dans Réglages → Général.

## Sécurité

- **Coffre à secrets :** chiffrement optionnel par phrase de passe des mots de
  passe stockés (AES-GCM + PBKDF2). Les valeurs chiffrées sont portables ; une
  seule invite de déverrouillage au lancement les déchiffre pour la session.

## Intégrations (sortantes)

- **Émetteurs UDP :** pousser la fréquence actuelle vers **PstRotator**, les
  infos radio au format **N1MM `RadioInfo`**, ou un **enregistrement ADIF à
  chaque QSO logué** — pour que les outils externes (contrôle de rotor,
  applications numériques, autres logiciels de log) restent synchronisés.

## Divers

- **Démarrage automatique :** lancer des programmes externes (WSJT-X, JTAlert,
  contrôle de rotor…) au démarrage d'OpsLog, en sautant ceux déjà lancés.
- **Sauvegarde :** sauvegarde optionnelle base + ADIF à la fermeture.
- **Vérification de mise à jour** au démarrage et toutes les 5 minutes (et à
  l'ouverture d'Aide → À propos), avec un toast (désactivable), plus une fenêtre
  **Nouveautés** qui affiche le changelog (anglais / français) au premier
  lancement après une mise à jour — réouvrable à tout moment depuis le menu Aide.
- **Télémétrie d'usage anonyme** (un battement de cœur quotidien : ID
  d'installation aléatoire + version + OS — aucune donnée d'indicatif ou de QSO ;
  désactivable dans les Préférences).

---

## Concepteur de cartes QSL

Outils → *Concepteur de cartes QSL…* transforme quelques photos en une carte
eQSL soignée :

1. Choisissez 1 à 6 photos (jpeg/png). OpsLog les analyse hors ligne (grille de
   détail/luminance) et propose **3 designs** — indicatif dans la zone la plus
   calme de la meilleure photo, nom de l'opérateur, zones CQ/ITU + ligne locator,
   drapeau du pays, les autres photos en inserts encadrés, et un encart de
   confirmation par QSO.
2. Choisissez une proposition et affinez-la : cliquez un élément pour le
   sélectionner, glissez pour déplacer, changez la police / le préréglage de
   style (gel or, gel argent, contour blanc classique, script, plat) et les
   réglages par préréglage dans le panneau de droite.
3. Enregistrez le modèle (les photos sont copiées dans
   `data/qsl/templates/<id>/`, les originaux peuvent donc être déplacés). Un
   modèle peut être le défaut par profil.

Envoi : clic droit sur un QSO → *Envoyer l'eQSL par e-mail*. La carte est rendue
avec les données de ce QSO, matricée en JPEG ≤ 800 Ko, archivée dans
`data/qsl/outbox/` et envoyée via le compte SMTP configuré à l'adresse trouvée
par la recherche QRZ/HamQTH. En cas de succès le QSO est marqué `EQSL_SENT=Y`
(ADIF). Les modèles de sujet/corps de l'e-mail sont dans le concepteur (variables
`{CALL} {DATE} {BAND} {MODE} {MYCALL}`).

Polices : Archivo Black, Lilita One, Baloo 2, Oswald, Great Vibes, Allura
(toutes OFL, embarquées — licences dans `internal/qslcard/assets/fonts/`) ;
Cooper Black est proposée si MS Office l'a installée. Drapeaux : flag-icons
(MIT), embarqués pour les entités DXCC couramment contactées.

---

## Données & stockage

- La **config** (réglages, profils, radios/antennes, nœuds de cluster, cache de
  recherche, listes de diplômes, modèles QSL) réside toujours dans le fichier
  SQLite local sous `data/` — instantané même quand le journal est sur un MySQL
  lointain.
- Le **journal** (QSO) réside là où pointe le profil actif : le fichier SQLite
  local ou une base **MySQL** partagée par profil.

---




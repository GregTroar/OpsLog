# OpsLog

<a href="https://www.paypal.com/donate?hosted_button_id=R9TC4KXWR96UE" target="_blank">
  <img src="https://www.paypalobjects.com/en_US/i/btn/btn_donate_SM.gif" alt="Donate with PayPal" />
</a>
<a href="https://discord.gg/FYM8yw5pT" target="_blank">
  <img src="https://img.shields.io/badge/Discord-Join%20the%20server-5865F2?logo=discord&logoColor=white" alt="Join our Discord" />
</a>

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

*A French version of this document is available in [README.fr.md](README.fr.md).*

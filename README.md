# OpsLog

A modern, fast ham-radio logger for Windows — Log4OM-style entry, real-time CAT
(OmniRig **and** native FlexRadio/SmartSDR), DX cluster, awards tracking, maps,
QSL management and a QSL-card designer. Built with **Wails v2** (Go backend +
React/TypeScript frontend), **pure Go** (no CGO): SQLite for configuration,
optional **shared MySQL** for the logbook so several operators can run one log.

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
  with `/MM` `/AM` and call-area (`/8`, `/W6`) handling, plus ClubLog DXpedition
  date overrides.
- **Recent QSOs**, **Worked-before** matrix (per band/mode slot), bulk re-resolve
  from cty/QRZ/ClubLog, bulk send to QSL services.
- **Profiles:** every setting is per-profile; each profile can point its logbook
  at the local SQLite file or a **shared MySQL** database (multi-operator).

## Maps & antenna

- **Main view = two configurable panes** (per profile, Settings → General →
  *Main view*): great-circle map, locator (street) map, the cluster grid, the
  worked-before grid, or the **FlexRadio controls**.
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

## CAT control

- **OmniRig** backend (Rig 1/2, hot-swap), and a native **FlexRadio (SmartSDR)**
  backend over the radio's TCP API — real-time slice freq/mode/split, auto
  reconnect, UDP discovery, and **panadapter spots** (cluster spots pushed to the
  Flex display, click → fill the call).
- Mode is taken from the radio; the digital sub-mode (FT4 vs FT8) is inferred
  from the frequency.

### FlexRadio control tab (SmartSDR-style)

Shown only when the CAT backend is a FlexRadio:

- **Transmit:** RF power, tune power, TUNE, MOX, speech processor (NOR/DX/DX+),
  VOX (+ level + delay), monitor (+ level), mic gain.
- **Receive (active slice):** AGC mode/threshold, audio level, NB / NR / ANF.
- **Antenna tuner (ATU):** tune / bypass / memories.
- **Amplifier:** PowerGenius XL operate/standby + fault.
- **Live meters** over the UDP VITA-49 stream: S-meter (S-units), forward power
  (W), SWR, ALC, PA temperature, voltage, plus the amplifier's meters.

## Keyers & audio

- **WinKeyer** CW keyer (macros, F-key macros, auto-call repeat).
- **Digital Voice Keyer** (DVK) message playback.
- **QSO audio recording** (SSB/DAX) archived per QSO; disabled for CW (no DAX
  audio in CW).

## QSL & awards

- **Awards engine:** built-in + custom award definitions (shared **globally**
  across profiles), worked/confirmed/validated by band & mode, OR rules and
  manual reference assignment, live reference detection on call entry, and a
  **Rescan** that re-pulls the logbook (picks up fresh LoTW/QRZ confirmations).
- **QSL services:** ClubLog (batched ADIF upload), LoTW, QRZ.com, eQSL — upload
  and **confirmation download** (which auto-refreshes the award stats).
- **QSL Card Designer** (see below).
- **E-mail eQSL:** right-click a QSO → *Send eQSL by e-mail* via the configured
  SMTP account. (Outlook/Hotmail disable basic-auth SMTP — use Gmail with an app
  password, or a Microsoft app password.)

## Multi-operator live status (special events)

For a multi-op special-event call on a shared MySQL logbook (e.g. **TM74TFR**):
Settings → General → *Publish live operator status*. Each OpsLog instance
heartbeats its current activity (operator call, band, frequency, mode) into a
`live_status` table every ~15 s. A small PHP renderer
([`docs/livestatus/tm74-status.php`](docs/livestatus/tm74-status.php)) on your
own web server reads that table and produces a live page/image you can embed on
the station's **QRZ.com** bio (`<img src="…/tm74-status.php?img=1">`). OpsLog
only writes to the DB — it is not a web server.

## Other

- **Autostart:** launch external programs (WSJT-X, JTAlert, rotator control…) at
  OpsLog startup, skipping any already running.
- **Update check** at startup with a toast (toggleable).
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

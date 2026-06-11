# OpsLog

**A modern, fast ham radio logging suite for Windows — built in Go.**

OpsLog is a full station logbook designed for the operator who works DX, chases awards, runs digital modes and contests — and wants one application that ties the whole shack together. Native desktop app (Go + [Wails](https://wails.io) + React), single portable executable, local SQLite database. No cloud account, no subscription, your log stays on your machine.

> Developed by **F4BPO** (contest: TM2Q) as a modern alternative to RUMlogNG, Ham Radio Deluxe and Log4OM.

---

## Features

### Logging
- Fast QSO entry with live callsign lookup (**QRZ.com** / **HamQTH**, with local cache)
- Built-in **DXCC resolver** (cty.dat-based) with portable-prefix and special-callsign handling, validated against established loggers
- Worked-before panel, band/mode slot matrix and per-QSO details at a glance
- Powerful filter builder (any field, AND/OR conditions) on the full log
- **ADIF import/export** with round-trip fidelity (covered by automated tests)
- Multi-profile support (home call, contest call, portable operations) with per-profile scoping

### QSL & awards
- **QSL Manager**: upload/download to **LoTW** (via TQSL), **Club Log**, **QRZ.com** and eQSL status tracking — with per-service sent/confirmed status on every QSO
- Awards engine with confirmed band/mode/entity slot tracking
- Award references module: **POTA**, SOTA and custom award programs, including POTA hunter-log synchronization from pota.app

### Station integration
- **CAT control via OmniRig** (Rig1/Rig2): frequency, mode, split, PTT
- **DX Cluster** client: multiple servers, spot grid, band map, self-spot alerts, distance/bearing from your grid
- **UDP integrations**: auto-log and DX-call follow from **WSJT-X / JTDX / MSHV**, N1MM-style broadcasts, with duplicate-safe auto-logging
- **WinKeyer** support for CW: macros, live echo, speed control
- Rotator control through **PstRotator** (UDP) with compass display
- **Ultrabeam** antenna controller integration (TCP), including follow-frequency mode

### Audio
- Continuous **QSO recorder** with rolling pre-roll — never miss the start of a contact
- **DVK voice keyer** (record/play slots, PTT keying via CAT, RTS or DTR)
- E-mail a QSO recording to your correspondent directly from the log (SMTP, SSL/STARTTLS)

### Data safety
- Local **SQLite** database (WAL mode), transactional schema migrations
- Automatic rotating **backups** (daily snapshots, optional zip, configurable retention)
- Portable mode: keep the app and its data on a USB stick

---

## Screenshots

*(coming soon)*

---

## Requirements

- **Windows 10/11** (64-bit) — CAT control relies on [OmniRig](https://dxatlas.com/OmniRig/) (COM)
- [OmniRig](https://dxatlas.com/OmniRig/) installed and configured for your rig (only if you use CAT)
- [TQSL](https://lotw.arrl.org/lotw-help/installation/) installed (only if you upload to LoTW)

## Installation

Download the latest release from the [Releases](../../releases) page and run `OpsLog.exe`. The application creates a `data/` folder next to the executable (configuration, log database, cty.dat) — delete the folder, and the app is gone. No installer, no registry.

## Architecture

```
.
├── main.go / app.go        # Wails bootstrap and frontend bindings
├── internal/
│   ├── qso/                # QSO repository (SQLite)
│   ├── adif/               # ADIF parser / exporter
│   ├── dxcc/               # DXCC / zone resolver (cty.dat)
│   ├── award/, awardref/   # Awards engine and reference catalogs (POTA, …)
│   ├── cluster/            # DX cluster telnet client
│   ├── cat/                # CAT manager + OmniRig COM backend
│   ├── integrations/udp/   # WSJT-X / N1MM UDP listeners
│   ├── extsvc/             # LoTW, Club Log, QRZ upload/download
│   ├── lookup/             # QRZ / HamQTH callsign lookup + cache
│   ├── audio/              # QSO recorder, DVK, playback
│   ├── winkeyer/           # WinKeyer serial protocol
│   ├── rotator/, ultrabeam/# Rotator & antenna controllers
│   ├── backup/, db/        # Backups, SQLite connection & migrations
│   └── ...
└── frontend/               # React + TypeScript + Tailwind (Wails asset server)
```

The backend is pure Go (no CGO — `modernc.org/sqlite`), which keeps builds simple and the binary self-contained.

## Roadmap

- [ ] Awards engine: additional programs and submission exports
- [ ] Contest mode
- [ ] macOS/Linux support (pending a cross-platform CAT backend)
- [ ] Localization (French first)

## Contributing

Issues and pull requests are welcome. If you're reporting a logging or DXCC-resolution bug, please attach a minimal ADIF sample that reproduces it.

## License

*(to be defined — see [LICENSE](LICENSE))*

## Author

**Greg — F4BPO** · contest callsign **TM2Q** · grid JN36
[qrz.com/db/F4BPO](https://www.qrz.com/db/F4BPO)

73 and good DX!

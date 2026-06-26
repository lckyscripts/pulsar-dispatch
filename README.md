# 🚔 Pulsar Dispatch

**Advanced FiveM Dispatch & Alert System** — a complete in-game dispatch panel with live map, automated alert detection, and real-time unit tracking.

[![Framework](https://img.shields.io/badge/Framework-QBCore%20|%20QBX%20|%20ESX-blue)]()
[![Language](https://img.shields.io/badge/Language-Lua%20%7C%20TypeScript-orange)]()
[![UI](https://img.shields.io/badge/UI-React%2019%20%7C%20Leaflet-green)]()
[![License](https://img.shields.io/badge/License-Proprietary-red)]()

---

## 📋 Table of Contents

- [Features](#-features)
- [Framework Support](#-framework-support)
- [Requirements](#-requirements)
- [Installation](#-installation)
- [Configuration](#-configuration)
- [Customization (No Rebuild)](#-customization-no-rebuild)
- [Client Commands](#-client-commands)
- [External API / Exports](#-external-api--exports)
- [Alert System](#-alert-system)
- [Architecture](#-architecture)
- [Performance](#-performance)
- [Troubleshooting](#-troubleshooting)
- [License & Support](#-license--support)

---

## ✨ Features

### Dispatch Panel
- **Live Dispatch Map** — GTA V map rendered via Leaflet with real-time unit positions (WebP tiles, 60% smaller than PNG)
- **Incident Management** — Active calls panel with priority-based sorting, assigned units display
- **Unit Status Board** — Collapsible table with search, filter, sort, inline callsign editing, and job-based color badges
- **Status Dropdown** — 5 customizable status codes with color-coded badges and pulse animation for emergencies
- **Alert Pop-ups** — Animated pop-up alerts with countdown timers, police lights, and one-click waypoint
- **Profile Badge** — Persisted badge number stored in framework metadata (survives restarts)

### Alert Detection System
- **Shots Fired** — Automatic gunshot detection with weapon identification (skips stunguns, flashlights)
- **Explosions** — Explosion event detection with witness requirement
- **Traffic Accidents** — Vehicle rollover detection via native polling
- **Melee Fights** — Street fight detection with active combat verification
- **Car Jacking** — Vehicle theft in progress detection
- **Vehicle Theft** — Car alarm-based theft detection
- **Officer Down** — Police officer fatality detection (priority 1, all units)
- **Civilian Down** — Civilian fatality detection
- **Reckless Driving** — Multi-event speeding detection (80+ km/h threshold)
- **PANIC Button** — Manual panic trigger with GPS coordinates

### Map Features
- **Unit Markers** — Circle dots with job-based customizable colors (police/sheriff/ambulance/mechanic)
- **Alert Markers** — Pulse-animated alert icons with tooltip + popup detail views
- **One-click Waypoint** — GPS button sets in-game waypoint and auto-assigns the responding unit
- **Keyboard Shortcut** — `E` key sets waypoint + auto-assigns to the latest priority alert
- **Batch-Optimized** — Unit markers refresh every 2 seconds without disrupting alert popups

### Technical Highlights
- **RAM-Only Cache** — All state kept in memory, no database required
- **Panel-State-Aware Batch** — Only clients with open panels receive data broadcasts
- **Framework Bridge Pattern** — Single API contract, adapters for QBX/QBCore/ESX
- **Live Config** — Status codes and job dot colors read from `types.lua` at runtime, no UI rebuild needed
- **Export API** — External resources can trigger dispatches via `exports['pulsar-dispatch']:CreateDispatch()`

---

## 🏗 Framework Support

| Framework | Status | Bridge Adapter |
|-----------|:------:|----------------|
| **QBCore** | ✅ Full | `server/bridges/qb.lua` |
| **QBX Core** | ✅ Full | `server/bridges/qbx.lua` |
| **ESX Legacy** | ✅ Full | `server/bridges/esx.lua` |

The bridge pattern auto-detects the active framework at startup and loads the correct adapter. No configuration needed.

---

## 📦 Requirements

- **FiveM** server (latest artifact recommended)
- **One of:** QBCore, QBX Core, or ESX Legacy
- **ox_lib**

---

## 🚀 Installation

### 1. Download & Extract
```
resources/
└── [lcky-scripts]/
    └── pulsar-dispatch/
        ├── fxmanifest.lua
        ├── client/
        ├── server/
        ├── shared/
        └── ui/
```

### 2. Configure
Open `shared/config.lua` and set your allowed jobs:
```lua
Config.Jobs.allowedJobs = { 'police', 'sheriff' }  -- Add 'ambulance', 'mechanic', etc.
```

### 3. Start the Resource
Add to your `server.cfg`:
```
ensure pulsar-dispatch
```

That's it. The panel opens with `/dispatch` command by default or F12.

---

## ⚙️ Configuration

All configuration lives in three files:

| File | Purpose |
|------|---------|
| `shared/config.lua` | Jobs, dispatch settings, alert thresholds, map settings |
| `shared/types.lua` | Status codes, job dot colors (live config, no rebuild) |
| `shared/events.lua` | Event name registry |

### Key Config Options

```lua
-- Jobs that can access the dispatch panel
Config.Jobs.allowedJobs = { 'police', 'sheriff' }

-- Require clocked-in duty status
Config.Jobs.requireDuty = false

-- Dispatch auto-expiry (5 minutes default)
Config.Dispatch.dispatchLifetime = 300000  -- ms

-- Maximum solved/expired dispatches kept in RAM
Config.Dispatch.maxHistoricalDispatches = 100

-- Unit location update interval
Config.Map.updateInterval = 5000  -- ms

-- Debug mode (F8 console logging)
Config.UI.debug = true
```

### Alert Configuration

Each alert type has its own section with these parameters:
- `enabled` — toggle on/off
- `code` / `codeLabel` — 10-code displayed in UI
- `priority` — 1 (critical) to 5 (info)
- `throttle` — minimum interval between alerts from the same player (ms)
- `detector` — game event listener or native polling configuration
- `descriptionTemplate` — story-driven alert text with `{placeholders}`

---

## 🎨 Customization

### Status Codes (`shared/types.lua`)

Add, remove, or rename status codes freely. The UI dropdown reads this table when the panel opens.

```lua
StatusCodes = {
    ['10-8']    = { label = 'Available',    category = 'available', color = 'green'  },
    ['10-6']    = { label = 'Busy',         category = 'busy',      color = 'yellow' },
    ['CODE 99'] = { label = 'Emergency',    category = 'emergency', color = 'red', animation = 'pulse' },
    -- Add your own:
    ['10-99']   = { label = 'Coffee Break', category = 'busy',      color = 'purple' },
}
```

### Job Dot Colors (`shared/types.lua`)

Customize the CircleMarker dot for each job on the dispatch map.

```lua
JobDotColors = {
    police    = { fill = "#1e3a5f", stroke = "#162d4a", radius = 6 },
    sheriff   = { fill = "#3f1e0a", stroke = "#2a1508", radius = 6 },
    ambulance = { fill = "#1e3a1e", stroke = "#162a16", radius = 6 },
    mechanic  = { fill = "#5a3a1e", stroke = "#3a2814", radius = 6 },
}
```

---

## ⌨️ Client Commands

| Command | Description |
|---------|-------------|
| `/dispatch` | Open/close the dispatch panel |
| `/panic` | Trigger PANIC button manually |
| `/setcallsign [name]` | Set your unit callsign (e.g., `1-ADAM-1`) |
| `/joinunit [serverId]` | Join another unit as secondary officer |
| `/leaveunit` | Leave secondary unit, return to primary |
| `/dispatch_test` | Create a test dispatch at your location |

---

## 🔌 External API / Exports

Other resources can interact with the dispatch system via server-side exports:

```lua
-- Create a dispatch from an external resource (e.g., bank heist script)
exports['pulsar-dispatch']:CreateDispatch(0, {
    code         = '211',
    codeLabel    = 'Bank Robbery',
    priority     = 1,
    location     = 'Pacific Standard Bank',
    district     = 'Alta',
    coords       = vector3(235.0, -895.0, 30.0),
    description  = 'Silent alarm triggered.',
    callerName   = 'Bank Security',
    channel      = 1,
    meta         = { alarmType = 'silent' }
})

-- Get all active dispatches (returns table)
local dispatches = exports['pulsar-dispatch']:GetActiveDispatches()

-- Get all active units
local units = exports['pulsar-dispatch']:GetActiveUnits()

-- Assign a unit to a dispatch
exports['pulsar-dispatch']:SendUnitToDispatch('DSP-250619-1234', '3')

-- Trigger the Panic Button
exports['pulsar-dispatch']:TriggerPanicButton()
```

**Note:** `source = 0` means "system-triggered" — no source-specific event is sent.

---

## 🚨 Alert System

### Architecture

```
Game Event / Native Polling
    ↓
Client Detector (client/detectors/)
    ↓
TriggerAlert(alertKey, sourcePed)
    ↓
CollectMeta() — weapon, vehicle, speed, nearby civilians
    ↓
TriggerServerEvent('pulsar_dispatch:createAlert', dispatchData)
    ↓
Server: CreateDispatch() → NotifyAllPolice()
    ↓
Client: HandleNewDispatch() → NUI Pop-up + Dispatch Map Icon
```

---

## 🏛 Architecture

```
┌─────────────────────────────────────────────┐
│                  SERVER                      │
│  ServerState (RAM Cache)                     │
│  ├── dispatches[]   (active/solved/expired)  │
│  ├── units{}        (key: tostring(source))  │
│  └── panelOpenSources{}  (batch optimization)│
│                                              │
│  Bridge Pattern: qbx.lua / qb.lua / esx.lua │
│  Periodic Tasks: cleanup + batch broadcast   │
└─────────────────────────────────────────────┘
           │                    ▲
           │ TriggerClientEvent │ TriggerServerEvent
           ▼                    │
┌─────────────────────────────────────────────┐
│                  CLIENT                      │
│  DispatchState (NUI message relay)           │
│  ├── Alert Detectors (game events/polling)  │
│  ├── NUI Callbacks (setWaypoint, etc.)      │
│  └── Periodic Location Updates (5s)         │
└─────────────────────────────────────────────┘
           │                    ▲
           │ SendNUIMessage      │ fetch POST
           ▼                    │
┌─────────────────────────────────────────────┐
│               NUI (React 19)                 │
│  Index.tsx — Main Dispatch Panel            │
│  ├── IncidentPanel — Active calls list      │
│  ├── DispatchMap — Leaflet map              │
│  │   ├── Unit Markers (batch remount)       │
│  │   └── Alert Markers (stable popups)      │
│  ├── UnitTable — Status board               │
│  └── AlertsContainer — Pop-up alerts        │
└─────────────────────────────────────────────┘
```

---

## ⚡ Performance

| Metric | Value |
|--------|-------|
| **Server State** | RAM-only, no database queries |
| **Unit Lookup** | O(1) via table key |
| **Batch Interval** | 2 seconds |
| **Batch Target** | Only clients with open panels (panel-state-aware) |
| **Location Update Interval** | 5 seconds (client → server) |
| **Stale Unit Cleanup** | 2 minutes of inactivity |
| **Dispatch Expiry** | 5 minutes (auto-expired) |
| **Max Historical Dispatches** | 100 (older purged) |
| **UI Bundle Size** | ~415 KB JS + ~50 KB CSS (gzipped: ~123 KB + ~14 KB) |
| **Build Time** | ~2.7 seconds |

### Event Frequency (50 online, 10 panels open)

| Event | Per 2 Seconds |
|-------|:------------:|
| `setUnits` (batch) | 10 (only open panels) |
| `unitLocationUpdate` | 10 (every 5s, staggered) |
| Total events/sec | ~7.5 |

---

## 🔧 Troubleshooting

### Panel doesn't open with `/dispatch`
- Check that your job is in `Config.Jobs.allowedJobs`
- If `Config.Jobs.requireDuty` is `true`, make sure you are clocked in
- Check F8 console for `[PULSAR-DISPATCH]` errors

### Map tiles not loading
- Verify `map-tiles` folder exists in `ui/assets/map-tiles/`

### Status codes not showing in dropdown
- Panel must be opened fresh (the `panelOpened` event sends reference data)
- Check `shared/types.lua` for syntax errors

### Badge number resets on restart
- Ensure `Bridge.setBadge()` is called in the `setBadgeNumber` handler
- Framework metadata must be writable (check 'badgeNumber' value on player data)

---

## 📄 License & Support

This resource is **proprietary software**. Redistribution without authorization is prohibited.

For support, feature requests, or customization inquiries, contact the developer through the purchase channel.

---

**Map tiles & Leaflat idea** [from](https://github.com/RiceaRaul/gta-v-map-leaflet) 

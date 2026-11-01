# CivicShield

> **Dashcams find road damage. CivicShield proves whether each repair actually lasted, explains the evidence, and holds repairs accountable.**

CivicShield answers the municipal hackathon challenge **"Public Transit Pothole & Road Anomaly Mapping Engine"**. When a city marks a road repair ticket "closed", nobody checks if the fix survived the monsoon. The pothole returns, is counted as a brand-new issue, and contractors are paid for repeated low-quality fixes.

CivicShield transforms transit fleet dashcam footage (buses, garbage trucks, municipal vehicles) into continuous road-audit infrastructure. It tracks every pothole spot across multiple passes, verifies repair durability at 30, 60, and 90 days inside a 25 m geofence, and issues tamper-evident 2-page cryptographic PDF evidence reports to enforce warranty clauses.

---

## ⚡ Quickstart (Run in 1 Minute)

CivicShield runs locally on any laptop CPU with no paid APIs, no GPU, and no external cloud services.

### 1. Install Dependencies
```bash
pip install -r requirements.txt
```

### 2. Seed Database
Seeds 250 spots, 120 repairs across 4 Bangalore wards, planted contractor hold rates, sample road condition segments, and demo emails:
```bash
python -m backend.seed
```

### 3. Start Application
```bash
python -m uvicorn backend.main:app --host 0.0.0.0 --port 8000
```

Open your browser to:
👉 **`http://localhost:8000`** — Unified full-stack application (Command Center, Road Scan, Durability Map, Priority Spots, Repairs & Warranty, Evidence Reports).
👉 **`http://localhost:8000/docs`** — Interactive OpenAPI / Swagger documentation.

*(Optional Docker deployment: `docker-compose up --build`)*

---

## 🛠️ Architecture & Tech Stack

```
                          ┌─────────────────────────────────────┐
                          │    Transit Dashcams & GPS Track     │
                          └──────────────────┬──────────────────┘
                                             │ Video / CSV
                                             ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ BACKEND (FastAPI, Python 3.11, SQLAlchemy, APScheduler)                         │
│                                                                                 │
│  • CV Detection Engine: OpenCV + PIL + NumPy adaptive frame analysis            │
│  • Spot Clustering: Haversine distance clustering (≤ 5m threshold)             │
│  • Durability Verification: 25m geofence evaluation across 30, 60, 90 days      │
│  • Scoring & Honesty Engine: Closure Honesty Score (Logged vs Held repairs)      │
│  • Evidence Compiler: ReportLab 2-page cryptographic PDF with SHA-256 hash      │
│  • Simulated Mailbox: Corporation ticket sync & automated warranty breach alerts│
│  • Road Condition Engine: 100m segment scoring (Bus + Citizen observations)     │
└────────────────────────────────────┬────────────────────────────────────────────┘
                                     │ REST JSON & PDF Streams
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ FRONTEND (Tailwind CSS, Next.js / Vanilla SPA, Leaflet, CARTO Tiles)            │
│                                                                                 │
│  • Fixed Navy Sidebar (240px) with Official/Viewer Role Switcher                │
│  • Command Center with Hero Banner ("Closed is not fixed. Prove it lasted.")    │
│  • Repair Durability Map (Held / Watch / Failed / Not enough evidence)          │
│  • Road Condition Layer (Good / Caution / Avoid / No recent scan)               │
│  • Fix First: Priority Spots (Severity ranking + repeat failure detection)      │
│  • Demo Time Control Strip (+30d, +60d, +90d, Reset)                            │
│  • Corporation Mail Panel (Outbox / Inbox with interactive simulated replies)   │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 🔬 Core Durability Verification Rules

Every repair logged by an official initiates a durability clock and a **25-meter spatial geofence**:

| Verdict | Verification Condition |
| :--- | :--- |
| **Held** | $\ge 2$ subsequent camera passes covered the spot with **zero** detections inside the 25 m geofence. |
| **Failed** | A detection above confidence threshold ($\ge 0.5$) occurs within the 25 m geofence in any subsequent pass. |
| **Not Enough Evidence** | $< 2$ subsequent passes have traversed the spot since the repair date. |
| **Watch** | Repair is within the 90-day warranty window and awaiting final verification passes. |

### Closure Honesty Score
$$\text{Closure Honesty Score} = \frac{\text{Held Repairs}}{\text{Total Logged Repairs}} \times 100\%$$
- Evaluated per ward and per contractor.
- Warns of **"Low Sample Size"** when evaluated against fewer than 10 logged repairs to prevent statistical skew.

---

## 🎨 UI Palette & Design Principles

Built strictly adhering to the civic engineering design language:
- **Navy Primary**: `#0B1F3A`
- **Dark Navy Sidebar & Headers**: `#081629`
- **Emerald Brand Accent**: `#10B981` (Hover: `#047857`)
- **Page Canvas**: `#F4F7F6`
- **Verdicts**:
  - Held: `#059669`
  - Watch: `#D97706`
  - Failed: `#DC2626`
  - Insufficient Data: `#64748B`
- **Road Condition Segments**:
  - Red (Score $\ge 5$): `#E24B4A`
  - Yellow (Score $2 - <5$): `#EF9F27`
  - Blue (Score $<2$, scanned $\le 14$ days): `#378ADD`
  - Gray dashed (Not scanned within 14 days): `#888780`

---

## 🧪 Running Automated Tests

Run the complete regression suite (Haversine distance, severity calculations, geofence verdicts, mail state transitions, and segment scoring):

```bash
python -m unittest backend/test_civicshield.py
```
*(All 6 unit test suites pass in $<0.2$ seconds).*

---

## 📜 Dataset Credits & Licensing

- **Road Damage Dataset 2022 (RDD2022)**:
  - Authors: Deeksha Arya, Hiroya Maeda, Sanjay Kumar Ghosh, Durga Toshniwal, Alexander Mraz, Takehiro Kashiyama, Yoshihide Sekimoto.
  - License: Creative Commons Attribution-ShareAlike 4.0 International ([CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)).
  - Reference: *Global Road Damage Detection: State-of-the-art Solutions*, IEEE Big Data / RDD2022.

---

## ⚠️ Disclaimer & Synthetic Data Notice

All road damage spots, contractor profiles, municipal engineer correspondence, ward boundaries, and durability statistics presented in default demonstration states are **synthetically generated sample data** for evaluation and demonstration purposes. No real contractor names, civic worker emails, or actual municipal performance ratings are represented.


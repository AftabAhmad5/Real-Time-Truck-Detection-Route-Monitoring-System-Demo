<div align="center">

# 🚛 Real-Time Truck Detection & Route Monitoring System Demo

<p align="center">
  <img src="https://img.shields.io/badge/Status-Delivered-brightgreen?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Client-US%20Government%20Agency-003087?style=for-the-badge&logo=flag" />
  <img src="https://img.shields.io/badge/Type-Computer%20Vision%20Pipeline-purple?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Cameras-Live%20Multi--Feed-orange?style=for-the-badge" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/YOLOv11-00FFFF?style=flat-square&logo=yolo&logoColor=black" />
  <img src="https://img.shields.io/badge/OpenCV-5C3EE8?style=flat-square&logo=opencv&logoColor=white" />
  <img src="https://img.shields.io/badge/FFmpeg-007808?style=flat-square&logo=ffmpeg&logoColor=white" />
  <img src="https://img.shields.io/badge/SQL-4479A1?style=flat-square&logo=postgresql&logoColor=white" />
</p>

> A production-grade computer vision pipeline that **detects, tracks, and counts trucks** across live government road camera feeds in real time — delivered for a **US Government Agency** monitoring truck traffic across Wisconsin interstate routes.

</div>

---

## 📌 Project Overview

This system was built for a confidential US government client requiring accurate, real-time truck traffic monitoring across specific trucking corridors in Wisconsin. 

The pipeline fetches live HLS streams from the **511 WI public traffic camera API**, filters cameras to truck-designated interstates, geo-maps them to nearby truck parking facilities using the **Haversine algorithm**, then runs **YOLOv11 inference with ByteTrack** multi-object tracking — logging all detection events with timestamps for client reporting.

---

## 🚀 Key Results

| Metric | Result |
|--------|--------|
| 🎯 Detection Model | YOLOv11 — real-time, production-grade |
| 🔁 Tracking | ByteTrack — persistent IDs, zero duplicate counts |
| 📡 Camera Feeds | Multiple simultaneous live HLS streams |
| 🛣️ Routes Monitored | I-39 · I-41 · I-43 · I-90 · I-94 · I-535 |
| 📍 Geo-Filter Radius | 5 km Haversine radius per parking facility |
| 🗄️ Delivery Format | Timestamped SQL database for client reporting |
| 🔌 Stream Resilience | Auto-reconnect on drops, bad packets, HLS interruptions |

---

## 🏗️ System Architecture

```
511 WI Traffic Camera API
           │
           ▼
  ┌─────────────────────┐
  │   Route Filter      │  ← I-39 · I-41 · I-43 · I-90 · I-94 · I-535
  │   (api_client.py)   │
  └─────────────────────┘
           │
           ▼
  ┌─────────────────────┐
  │  Haversine          │  ← cameras within 5 km of truck parking spots
  │  Geo-Filter         │    ranked by proximity · deduplicated
  │  (geo_filter.py)    │
  └─────────────────────┘
           │
           ▼
  ┌─────────────────────┐
  │  PyAV HLS Decoder   │  ← per-process · frame-drop resilient
  │  (stream.py)        │    auto-reconnect · FPS throttled
  └─────────────────────┘
           │
           ▼
  ┌─────────────────────┐
  │  ROI / Lane Mask    │  ← click-mapped polygons → lane_coords.json
  │  (lane_mapper.py)   │
  └─────────────────────┘
           │
           ▼
  ┌─────────────────────┐
  │  YOLOv11 +          │  ← detection · persistent ID · motion traces
  │  ByteTrack          │    unique truck count · direction of travel
  │  (detector.py)      │
  └─────────────────────┘
           │
           ▼
  ┌─────────────────────┐
  │  SQL Database       │  ← timestamped events + client reporting
  └─────────────────────┘
```

---

## 🔧 Key Features

### 🛣️ Truck Route Filtering
Queries the 511 WI camera API and filters down to cameras on known truck interstates only — **I-39, I-41, I-43, I-90, I-94, and I-535** — so no irrelevant feeds ever enter the pipeline.

### 📍 Haversine Geo-Mapping
Pulls truck parking facility data from the government API and applies the **Haversine formula** to find cameras within a **5 km radius** of each parking spot. Cameras are ranked by proximity and deduplicated, giving the pipeline the most relevant feeds first.

### 🎯 YOLOv11 + ByteTrack Detection
**YOLOv11** handles vehicle detection with class filtering focused on trucks. **ByteTrack** assigns persistent IDs across frames for accurate unique truck counting with zero duplicates. Motion traces are drawn manually using OpenCV — fading from dim to bright to show direction of travel.

### 📐 Interactive ROI / Lane Mapper
A click-to-define lane mapping tool lets you draw **Region of Interest polygons** directly on a live camera frame. Coordinates are saved to `lane_coords.json` and loaded at runtime to keep inference focused on the relevant road lanes only.

### 📡 Resilient Multi-Camera Streaming
PyAV-based HLS decoder runs each camera feed in its **own isolated process**. Handles frame drops, bad packets, and stream interruptions with automatic reconnection. FPS is throttled to avoid overloading the system.

### 🗄️ SQL Database Logging
All detection events are **timestamped and stored** in a SQL database for downstream reporting, trend analysis, and client delivery.

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|-----------|
| Object Detection | YOLOv11 (Ultralytics) |
| Multi-Object Tracking | ByteTrack |
| Video Decoding | PyAV, FFmpeg |
| Visualisation | OpenCV |
| Route & Geo Filtering | Haversine Algorithm |
| Camera & Parking Data | 511 WI Government REST API |
| Database | SQL |
| Language | Python 3 |

---

## 📁 Project Structure

```
truck-detection-system/
│
├── main.py           # Entry point — run this to start the full pipeline
├── config.py         # All settings in one place (model, FPS, routes, colors...)
├── api_client.py     # 511 WI API calls (cameras + parking facilities)
├── geo_filter.py     # Haversine distance calculation + camera–parking mapping
├── stream.py         # Resilient PyAV HLS decoder (per camera process)
├── detector.py       # YOLOv11 + ByteTrack + motion traces (TruckDetector class)
├── lane_mapper.py    # One-time interactive ROI tool → saves lane_coords.json
└── requirements.txt  # Python dependencies
```

---

## ⚙️ How to Run

**1. Install dependencies**
```bash
pip install -r requirements.txt
```

**2. Add your API key**
```python
# config.py
API_KEY = "YOUR_511_WI_API_KEY_HERE"
```

**3. Map your lane ROIs** *(one time per camera)*
```bash
python lane_mapper.py --url <YOUR_CAMERA_STREAM_URL>
# Left-click to add points · S to save a lane · U to undo · Q to finish
```

**4. Run the full pipeline**
```bash
python main.py

# Optional flags:
python main.py --limit 5       # open only 5 camera streams
python main.py --url <URL>     # test on a single camera, skip API
```

---

## 🔒 Note on Source Code

> Source code is private under a **US Government client confidentiality agreement**.
> This repository documents the system architecture, approach, and technical decisions made during development.
> The full codebase is available upon request for verified employers and clients.

---

## 👤 Author

<div align="center">

**Aftab Ahmad Lodhi**
*Electrical Engineer | Computer Vision & IoT*

[![GitHub](https://img.shields.io/badge/GitHub-AftabAhmad5-181717?style=flat-square&logo=github)](https://github.com/AftabAhmad5)
[![Email](https://img.shields.io/badge/Email-aftabahmadlodhi%40gmail.com-D14836?style=flat-square&logo=gmail&logoColor=white)](mailto:aftabahmadlodhi@gmail.com)

</div>

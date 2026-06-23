# EMERALD — AI-Powered Traffic Violation Detection & Analytics Platform

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python)](https://python.org)
[![YOLOv8](https://img.shields.io/badge/Detection-YOLOv8n-00BFFF?logo=yolo)](https://ultralytics.com/yolov8)
[![ByteTrack](https://img.shields.io/badge/Tracking-ByteTrack-orange)](https://github.com/ifzhang/ByteTrack)
[![EasyOCR](https://img.shields.io/badge/OCR-EasyOCR-green)](https://github.com/JaidedAI/EasyOCR)
[![CUDA](https://img.shields.io/badge/Hardware-NVIDIA%20GPU-76B900?logo=nvidia)](https://developer.nvidia.com/cuda-toolkit)
[![Version](https://img.shields.io/badge/Pipeline%20Version-v8.1-purple)](.)
[![License](https://img.shields.io/badge/License-MIT-lightgrey)](LICENSE)

> **Intelligent traffic monitoring that combines Computer Vision, Multi-Object Tracking, OCR, and Rule-Based Analytics to automate violation detection and evidence generation at scale.**

---

## Table of Contents

- [Problem Statement](#problem-statement)
- [Solution Overview](#solution-overview)
- [Key Features](#key-features)
- [System Architecture](#system-architecture)
- [Technology Stack](#technology-stack)
- [Supported Violations](#supported-violations)
- [Quick Start / Demo Usage](#quick-start--demo-usage)
- [Running the Demo — Step-by-Step](#running-the-demo--step-by-step)
- [Output Structure](#output-structure)
- [Analytics & Reporting](#analytics--reporting)
- [Performance Evaluation](#performance-evaluation)
- [Scalability Vision](#scalability-vision)
- [Impact & Strategic Value](#impact--strategic-value)
- [Future Enhancements](#future-enhancements)
- [Team](#team)

---

## Problem Statement

Rapid urbanisation is driving unprecedented growth in vehicle density across urban corridors. Traditional traffic enforcement — reliant on manual monitoring, static cameras reviewed by human operators, and physical patrols — simply cannot scale to meet this challenge:

- **Coverage gaps:** hundreds of junctions across a city cannot be simultaneously monitored by human operators.
- **Human fatigue & inconsistency:** manual review is error-prone, non-reproducible, and unscalable.
- **Evidence quality:** without automated evidence generation, violation records are difficult to contest, audit, or act upon.
- **Data blind spots:** absence of structured violation data prevents data-driven policy and infrastructure decisions.

Governments and smart-city authorities need an automated, evidence-grade, data-driven traffic intelligence layer — not a standalone detection model.

---

## Solution Overview

EMERALD is a production-oriented computer vision pipeline that processes traffic video footage end-to-end. Starting from raw CCTV or drone footage, the system:

1. **Detects** vehicles, motorcycles, bicycles, and pedestrians using YOLOv8 with per-class confidence floors and shape-based false-positive rejection.
2. **Tracks** every road user across frames using a purpose-built dual-ByteTrack architecture that separates two-wheeler and large-vehicle tracking to minimise ID loss and ghost detections.
3. **Understands road geometry** through scene calibration — lane polygons, directional vectors, stop-line regions, and no-parking zones defined in pixel coordinates for any camera viewpoint.
4. **Applies a rule-based violation engine** that evaluates five violation categories per confirmed track using geometric, temporal, and multi-frame voting logic.
5. **Generates evidence** — annotated JPEG frames, JSONL violation logs, and OCR-read number plate strings — for every detected infraction.
6. **Produces analytics and reports** via a pandas-backed analytics module with searchable logs, trend visualisations, and exportable HTML reports.

EMERALD is designed as a **scalable traffic intelligence layer** that can be adapted to any camera viewpoint by updating scene calibration coordinates, without retraining any model.

---

## Key Features

| Feature | Description |
|---|---|
| **Vehicle Detection & Classification** | YOLOv8n inference across 6 COCO vehicle classes with per-class adaptive confidence floors |
| **Dual-Tracker Architecture** | Independent ByteTrack instances for two-wheelers and large vehicles; ID offset prevents collision |
| **Stale Track Pruning** | Automatic removal of ghost tracks after configurable frame age threshold |
| **Helmet Violation Detection** | Dedicated secondary YOLO model on head-crop sub-images; multi-frame vote window before flagging |
| **Triple Riding Detection** | Rider-count monitoring on two-wheelers; flags when occupancy exceeds 2 across recent frames |
| **Wrong-Side Driving Detection** | Velocity-vector voting against lane direction polygons with configurable agreement threshold |
| **Stop-Line & Red-Light Violation** | Traffic light colour extraction combined with geometric stop-line crossing detection |
| **Illegal Parking Detection** | Stationary duration tracking in no-parking zones; configurable dwell-time threshold |
| **OCR-Based Number Plate Recognition** | EasyOCR on lower-centre vehicle crop; lazy-loaded GPU reader |
| **Evidence Generation** | Per-violation annotated JPEG frames saved with bounding boxes, IDs, and overlay text |
| **Searchable Violation Records** | Filterable JSONL log by violation type, vehicle class, confidence, plate string, or timestamp |
| **Analytics & Reporting Dashboard** | pandas-backed trend visualisation, summary statistics, and one-click HTML report export |
| **Smart-City Scalability** | Configurable for any camera viewpoint via polygon coordinate update; no retraining required |

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         EMERALD PIPELINE v8.1                          │
└─────────────────────────────────────────────────────────────────────────┘

  Traffic Video Feed  (MP4 · AVI · MOV · MKV)
          │
          ▼
  ┌───────────────────┐
  │  Frame Ingestion  │  ← Configurable sample rate · CLAHE / unsharp-mask
  │  & Preprocessing  │    preprocessing · live-feed lite mode
  └────────┬──────────┘
           │
           ▼
  ┌───────────────────┐
  │   YOLOv8 Object   │  ← Per-class confidence floors · bounding-box area
  │     Detection     │    & aspect-ratio filters · traffic light state
  └────────┬──────────┘
           │
           ▼
  ┌───────────────────┐
  │  Dual ByteTrack   │  ← moto_tracker (persons · bicycles · motorcycles)
  │  Multi-Object     │    lv_tracker (cars · buses · trucks)
  │    Tracking       │    ID offset 50,000 · stale-track pruning
  └────────┬──────────┘
           │
           ▼
  ┌───────────────────┐
  │  Scene Calibration│  ← Lane polygons · direction vectors
  │  (per-junction)   │    stop-line regions · no-parking zones
  └────────┬──────────┘
           │
           ▼
  ┌───────────────────┐
  │  Violation Engine │  ← 5 rule evaluators · per-track flag deduplication
  │  (Rule Evaluator) │    multi-frame helmet & wrong-side voting
  └────────┬──────────┘
           │
           ▼
  ┌───────────────────┐
  │  OCR & Evidence   │  ← EasyOCR number plate reading · annotated JPEG
  │    Generation     │    export · JSONL violation log
  └────────┬──────────┘
           │
           ▼
  ┌───────────────────┐
  │  Analytics &      │  ← pandas DataFrame · trend charts · search API
  │    Reporting      │    · HTML report export · benchmark metrics
  └───────────────────┘
```

### Stage Descriptions

| Stage | Role |
|---|---|
| **Frame Ingestion & Preprocessing** | Decodes video at configurable sample rate; applies CLAHE contrast enhancement and unsharp-mask sharpening for low-contrast footage; can be bypassed in `LIVE_FEED_MODE` to reduce latency |
| **YOLOv8 Detection** | Runs YOLOv8n at full resolution (`YOLO_IMGSZ=1920`); applies per-class confidence floors post-inference; rejects road-paint false positives via area fraction and aspect-ratio shape filters |
| **Dual-Tracker** | Maintains two independent `sv.ByteTrack` instances — permissive settings for motorcycles/persons (single-frame confirmation), strict settings for large vehicles (2-frame confirmation); IDs offset by 50,000 to prevent collision |
| **Scene Calibration** | Junction-specific polygon definitions loaded once per run; any camera viewpoint is handled by replacing four coordinate arrays with CVAT-exported values |
| **Violation Engine** | Five geometry- and time-based rules evaluated per confirmed track; each rule fires at most once per track ID; returns a `ViolationEvent` dataclass |
| **OCR & Evidence** | Crops the lower-centre of the vehicle bounding box for EasyOCR; saves annotated frames to `violation_output/frames/` and appends a JSON record to `violations.jsonl` |
| **Analytics & Reporting** | `ViolationAnalytics` loads the JSONL log into a pandas DataFrame and exposes `stats()`, `search()`, `plot_trends()`, and `export_report()` methods |

---

## Technology Stack

| Component | Technology | Notes |
|---|---|---|
| Object Detection | YOLOv8n (Ultralytics) | COCO-pretrained; inference at 1920 px |
| Helmet Detection | YOLO (custom binary classifier) | Head-crop sub-image inference |
| Multi-Object Tracking | ByteTrack (via Supervision) | Dual-instance design; class-group-specific params |
| OCR | EasyOCR | GPU-accelerated; lazy initialisation |
| Detection Post-Processing | Supervision (`sv`) | Detections, ByteTrack, annotators |
| Image Processing | OpenCV | Frame decode, CLAHE, polygon tests, drawing |
| Data Analysis | pandas, Matplotlib | Log loading, analytics, chart export |
| Programming Language | Python 3.9+ | Dataclasses, type hints throughout |
| Hardware Acceleration | NVIDIA GPU (CUDA) | Verified at runtime; VRAM usage benchmarked |
| Notebook Environment | Google Colab / Jupyter | Colab file upload + ipywidgets fallback |

---

## Supported Violations

| Violation | Detection Method | Rule Logic |
|---|---|---|
| **No-Helmet Riding** | Secondary YOLO classifier on rider head crop | Multi-frame vote over rolling window; flags when helmet-absent fraction exceeds threshold |
| **Triple Riding** | Rider-count accumulation per track | Flags when max occupancy in recent 5 frames exceeds 2 |
| **Wrong-Side Driving** | Velocity vector vs. lane direction polygon | Multi-frame direction vote; flags when wrong-direction fraction exceeds 60% |
| **Red-Light / Stop-Line Violation** | Traffic light colour + stop-line geometry | Flags when vehicle leading edge crosses stop-line polygon while light is red |
| **Illegal Parking** | Stationary duration in no-parking zone | Flags when vehicle has been still for ≥ 5 seconds inside a defined no-parking polygon |

> Each violation fires **at most once per track ID**, preventing duplicate records for the same vehicle.

---

## Quick Start / Demo Usage

```bash
# 1. Install dependencies
pip install ultralytics supervision easyocr ipywidgets

# 2. Clone the helmet model weights
git clone --depth 1 https://github.com/Juliowiwiwiwi/Bike-Helmet-Detction-Model.git helmet_src

# 3. Open the notebook
jupyter notebook v_8submission.ipynb

# 4. Upload your traffic video when prompted, configure your scene
#    calibration polygons, then execute all cells in order.
```

Outputs are written to `violation_output/`:
- `annotated_output_fixed.mp4` — full annotated video
- `frames/` — per-violation JPEG evidence images
- `violations.jsonl` / `violations.json` — structured violation log
- `trends.png` — violation trend chart
- `summary_report.html` — exportable HTML summary

---

## Running the Demo — Step-by-Step

### Step 1 — Obtain Traffic Footage

Download or record traffic footage from CCTV cameras, roadside units, drones, or public datasets. Supported formats: `.mp4`, `.avi`, `.mov`, `.mkv`. Place the video in your working directory.

> The notebook's upload cell auto-detects the runtime: in Google Colab it opens a browser file picker via `google.colab.files.upload()`; in local Jupyter it renders an `ipywidgets.FileUpload` widget.

### Step 2 — Extract a Reference Frame

Select a clear frame showing the full road layout (lane markings, stop lines, parking boundaries). Save it as a `.jpg` or `.png` for use in the calibration step.

```python
# Quick reference frame extraction
import cv2
cap = cv2.VideoCapture("your_video.mp4")
cap.set(cv2.CAP_PROP_POS_FRAMES, 100)   # choose a representative frame number
ret, frame = cap.read()
cv2.imwrite("reference_frame.jpg", frame)
cap.release()
```

### Step 3 — Configure the Scene using CVAT

Use **[CVAT](https://app.cvat.ai)** (free, browser-based) to annotate your reference frame:

1. Create a new task and upload `reference_frame.jpg`.
2. Draw **Polygon** shapes for each zone:
   - `Lane A` — primary traffic direction lane
   - `Lane B` — secondary / opposing lane
   - `Stop Line (Lane A)` — stop-bar region for Lane A
   - `Stop Line (Lane B)` — stop-bar region for Lane B
   - `No-Parking Zone` — restricted parking area(s)
3. Label each polygon with its semantic name.
4. Save and export annotations.

### Step 4 — Obtain Polygon Coordinates

Export annotations from CVAT (JSON or XML), or manually read off pixel coordinates from the annotation tool. Coordinates are expressed as `[(x1,y1), (x2,y2), ...]` arrays.

Example coordinate sets from the demo calibration (2560×1440 reference):

```python
LANE_A = [(193,1439), (1004,1), (1230,1), (2448,1440), (795,1440)]

LANE_B = [...]   # opposing-direction polygon

STOP_LINE_A = [(550,420), (820,420), (820,450), (550,450)]

NO_PARKING_ZONE = [(120,300), (300,300), (300,550), (120,550)]
```

### Step 5 — Insert Coordinates into the Configuration

In **Section 4 (Scene Calibration)** of the notebook, replace the `make_demo_calibration()` polygon arrays with your CVAT-derived coordinates:

```python
def make_demo_calibration(frame_w: int, frame_h: int) -> SceneCalibration:
    lane_A = np.array([
        # ← Replace with your CVAT Lane A polygon points
        [193,1439],[1004,1],[1230,1],[2448,1440],[795,1440]
    ], dtype=np.int32)

    lane_B = np.array([
        # ← Replace with your CVAT Lane B polygon points
    ], dtype=np.int32)

    lane_A_stop_line = np.array([
        # ← Replace with your stop-line region points
    ], dtype=np.int32)

    parking_zone = np.array([
        # ← Replace with your no-parking zone points
    ], dtype=np.int32)

    return SceneCalibration(
        lane_A=lane_A,
        lane_B=lane_B,
        lane_A_direction=(0.0, -1.0),   # ← set your lane direction vectors
        lane_B_direction=(0.0, 1.0),
        no_parking_zones=[parking_zone],
        lane_A_stop_line=lane_A_stop_line,
        ...
    )
```

> The same detection engine handles any junction — only these coordinate arrays change.

### Step 6 — Run EMERALD

Execute **Section 12 (Pipeline Execution)**:

```python
records = run_pipeline(source=VIDEO_PATH, max_frames=2000)
```

The pipeline will:
- Decode and preprocess frames at the configured sample rate.
- Run YOLOv8 detection and extract traffic light state.
- Route detections through the dual ByteTrack instances.
- Evaluate each confirmed track against all 5 violation rules.
- Run EasyOCR on the plate region of flagged vehicles.
- Save annotated evidence frames and append records to `violations.jsonl`.
- Write the full annotated video to `annotated_output_fixed.mp4`.

### Step 7 — Review Outputs

After the pipeline completes, **Section 13 (Output Export)** packages everything into a downloadable `emerald_outputs.zip`. Inside you will find:

| Output | Description |
|---|---|
| `annotated_output_fixed.mp4` | Full video with bounding boxes, track IDs, violation labels, and dashboard overlay |
| `frames/viol_NNNN_idXX_fYYYY.jpg` | Per-violation annotated JPEG evidence image |
| `violations.jsonl` | Newline-delimited JSON log (one record per violation event) |
| `violations.json` | Same data as a standard JSON array for external tool compatibility |
| `trends.png` | Violation-type bar chart and time-series trend plot |
| `summary_report.html` | Exportable HTML report with full violation table and statistics |

---

## Output Structure

Each violation event produces one JSON record:

```json
{
  "id": 12,
  "track_id": 347,
  "class": "motorcycle",
  "violations": ["no_helmet", "triple_riding"],
  "timestamp": 34.72,
  "frame_idx": 868,
  "plate": "KA05MN1234",
  "confidence": 0.812,
  "evidence": "viol_0012_id347_f868.jpg"
}
```

| Field | Type | Description |
|---|---|---|
| `id` | int | Sequential violation record number within the session |
| `track_id` | int | ByteTrack persistent ID (two-wheelers: 1–49,999; large vehicles: 50,001+) |
| `class` | string | COCO vehicle class name (e.g. `motorcycle`, `car`, `truck`) |
| `violations` | array | One or more violation tags fired in this event |
| `timestamp` | float | Video timestamp in seconds when the violation was recorded |
| `frame_idx` | int | Absolute decoded frame index |
| `plate` | string | OCR-read number plate string; empty string if unreadable |
| `confidence` | float | Detection confidence score from YOLOv8 (0–1) |
| `evidence` | string | Filename of the annotated JPEG in `violation_output/frames/` |

---

## Analytics & Reporting

The `ViolationAnalytics` class loads `violations.jsonl` into a pandas DataFrame and exposes:

### Summary Statistics (`stats()`)

Returns total violations, breakdown by type and vehicle class, mean detection confidence, number of plates successfully read, and the busiest 10-second window in the video.

### Searchable & Filterable Log (`search()`)

```python
# Find high-confidence no-helmet violations on motorcycles after 30s
analytics.search(
    violation_type="no_helmet",
    vehicle_class="motorcycle",
    min_confidence=0.70,
    start_time=30.0
)
```

Supported filters: `violation_type`, `vehicle_class`, `min_confidence`, `plate_contains`, `start_time`, `end_time`.

### Trend Visualisation (`plot_trends()`)

Generates two charts saved to `trends.png`:
- **Violations by Type** — bar chart of total counts per violation category.
- **Violations Over Video Time** — 10-second-binned time-series showing violation density.

### HTML Report Export (`export_report()`)

Writes a self-contained `summary_report.html` containing aggregate statistics, per-type counts, per-class breakdown, and the full violation table — ready for sharing with stakeholders or attaching to enforcement case files.

---

## Performance Evaluation

### Detection Quality

| Metric | Target | Evaluation Method |
|---|---|---|
| mAP@50 | ≥ 0.70 | `sv.metrics.MeanAveragePrecision` on hand-labelled frames |
| mAP@50–95 | ≥ 0.50 | Same framework, tighter IoU ladder |
| Precision | — | `sklearn.metrics.classification_report` per violation type |
| Recall | — | Same |
| F1 Score | — | Harmonic mean of precision and recall |

> The notebook's `run_map_eval()` function computes mAP against manually annotated `sv.Detections` objects. Ultralytics publishes YOLOv8n COCO mAP benchmarks which may be cited if on-site annotation is not feasible.

### OCR Accuracy

```python
# Exact-match rate against human-verified plate strings
plate_checks = [("KA05MN1234", "KA05MN1234"), ...]
exact_match_rate = sum(p == a for p, a in plate_checks) / len(plate_checks)
```

### Throughput & Latency

The `benchmark_pipeline()` function measures per-stage latency (preprocessing, detection, tracking) over a configurable number of frames and reports:

| Metric | Output |
|---|---|
| End-to-end FPS | Frames processed per second (preprocessing + detection + tracking) |
| Per-stage ms/frame | Breakdown for `preprocess`, `detect`, `track` |
| Peak GPU memory | `torch.cuda.max_memory_allocated()` in GB |
| Concurrent streams | Estimated camera feeds supportable at 15 fps/stream |

These benchmarks directly inform infrastructure sizing for multi-junction deployments.

---

## Scalability Vision

EMERALD is architected as a modular intelligence layer, not a monolithic model. Each component (detection, tracking, calibration, violation rules, OCR, analytics) can be upgraded or replaced independently.

```
Prototype (single camera, offline video)
        │
        ▼
Pilot Deployment (3–5 live CCTV feeds, edge GPU node)
        │
        ▼
District-Wide Rollout (multi-camera, centralised inference cluster)
        │
        ▼
City-Wide Monitoring (100+ junctions, streaming pipeline, real-time dashboard)
        │
        ▼
Intelligent Traffic Management Platform (adaptive signal control, predictive analytics)
```

Key scaling levers:

- **Camera agnosticism** — scene calibration is pure coordinate configuration; no model retraining required per junction.
- **Edge AI** — the YOLOv8n backbone is optimised for deployment on NVIDIA Jetson and similar edge devices.
- **Streaming input** — `FrameIngestion` accepts any OpenCV-compatible source, including RTSP streams, enabling live CCTV integration.
- **Modular violation rules** — each rule is independently toggleable (`ENABLE_HELMET`, `ENABLE_STOP_LINE`, etc.) and can be extended without touching the detection or tracking layers.
- **Analytics pipeline** — JSONL output is directly ingestible by downstream dashboards, databases, or enforcement workflow systems.

---

## Impact & Strategic Value

### 🚦 Road Safety

Automated, consistent detection of helmet violations, wrong-side driving, and red-light running removes human monitoring gaps and creates a credible deterrent. Evidence-grade records support enforcement actions that manual monitoring cannot reliably produce.

### ⚙️ Operational Efficiency

A single EMERALD deployment can monitor footage from dozens of junctions that would require hundreds of human operator-hours per day to review manually. Evidence generation is automatic, reducing administrative burden from violation recording to case preparation.

### 📊 Data-Driven Governance

Structured JSONL logs and analytics dashboards convert raw enforcement activity into actionable intelligence: violation hotspot identification, peak-hour analysis, repeat-offender detection via plate OCR, and long-term trend monitoring to measure the impact of traffic policy changes.

### 🏙️ Smart-City Readiness

EMERALD's modular architecture is designed to integrate with broader urban mobility platforms — traffic signal controllers, city dashboards, and automated enforcement backend systems — making it a practical building block for Intelligent Transportation System (ITS) deployments.

---

## Future Enhancements

- **Additional violation categories** — lane changing without indicators, overspeeding (via reference-object calibration), overloaded vehicles, mobile phone use while driving.
- **Enhanced OCR** — fine-tuned plate detection model to replace generic vehicle crop; multi-language plate support.
- **Real-time dashboards** — WebSocket-based live dashboard with per-junction feeds, live violation counts, and alert notifications.
- **Edge AI deployment** — TensorRT export of YOLOv8n and Triton Inference Server integration for sub-10ms latency on edge hardware.
- **Advanced traffic analytics** — vehicle flow estimation, queue detection, junction-level congestion scoring, and predictive maintenance alerts.
- **Automated enforcement integration** — REST API endpoint for downstream challan generation, RTO database lookups, and court-ready evidence packaging.
- **Official dataset integration** — fine-tuning on India-specific traffic datasets (IDD, CARLA synthetic) to improve detection accuracy under South Asian road conditions.

---

## Team

| Field | Details |
|---|---|
| **Team Name** | `[Team Name]` |
| **Members** | `[Member 1]` · `[Member 2]` · `[Member 3]` · `[Member 4]` |
| **Contact** | `[team@email.com]` |
| **Repository** | `[https://github.com/your-org/emerald]` |
| **Demo Video** | `[Link to demo footage]` |
| **Submission** | Hackathon Submission — `[Event Name]`, `[Date]` |

---

<div align="center">

**EMERALD** — Built to make roads safer, enforcement smarter, and cities more intelligent.

*Powered by YOLOv8 · ByteTrack · EasyOCR · OpenCV · Python*

</div>

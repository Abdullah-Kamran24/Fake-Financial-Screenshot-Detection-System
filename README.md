<div align="center">

```
███████╗ █████╗ ██╗  ██╗███████╗    ██████╗ ███████╗████████╗███████╗ ██████╗████████╗
██╔════╝██╔══██╗██║ ██╔╝██╔════╝    ██╔══██╗██╔════╝╚══██╔══╝██╔════╝██╔════╝╚══██╔══╝
█████╗  ███████║█████╔╝ █████╗      ██║  ██║█████╗     ██║   █████╗  ██║        ██║   
██╔══╝  ██╔══██║██╔═██╗ ██╔══╝      ██║  ██║██╔══╝     ██║   ██╔══╝  ██║        ██║   
██║     ██║  ██║██║  ██╗███████╗    ██████╔╝███████╗   ██║   ███████╗╚██████╗   ██║   
╚═╝     ╚═╝  ╚═╝╚═╝  ╚═╝╚══════╝    ╚═════╝ ╚══════╝   ╚═╝   ╚══════╝ ╚═════╝   ╚═╝   
```

### Fake Financial Screenshot Detection System

*Nine-layer AI forensics engine for Pakistani mobile payment fraud detection*

![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=flat-square&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-REST_API-000000?style=flat-square&logo=flask&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-Image_Forensics-5C3EE8?style=flat-square&logo=opencv&logoColor=white)
![Tesseract](https://img.shields.io/badge/Tesseract-OCR-FF6B35?style=flat-square)
![scikit-image](https://img.shields.io/badge/scikit--image-ELA_Analysis-F7931E?style=flat-square)
![Version](https://img.shields.io/badge/Version-2.1-22C55E?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-6366F1?style=flat-square)

</div>

---

## The Problem

In Pakistan, peer-to-peer payments via **Easypaisa**, **JazzCash**, and **UBL Digital** are ubiquitous. So is a simple fraud: screenshot an old transaction, edit the amount or recipient name in a photo editor, and send it as "proof of payment."

Manual verification is slow and error-prone. **Fake Detect** solves this with a nine-layer computer vision and NLP forensics pipeline that produces a single `REAL` or `FAKE` verdict in under two seconds.

---

## How It Works

```
User uploads screenshot
         │
         ▼
┌─────────────────────┐
│  Image Ingestion    │  ← Decoded to NumPy array, resized to 512×512
└──────────┬──────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────────┐
│                  Forensic Module Pipeline                    │
│                                                             │
│  [ELA]  [Noise]  [Edge]  [Color]  [Clone]  [DCT Forgery]   │
│  [Text Uniformity]  [Amount Sharpness]  [Semantic NLP]      │
│                                                             │
│  Each module returns a suspicion score: 0.0 → 1.0          │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │   Weighted Fusion      │  ← 9 scores → 1 confidence value
              │   + Penalty Rules      │  ← Spikes on clone / forgery / semantic
              │   + Hard Rule Check    │  ← Recipient name / timestamp override
              └────────────┬───────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │  REAL  /  FAKE  │  ← Verdict + per-module breakdown
                  └─────────────────┘
```

---

## Forensic Modules

| # | Module | Signal Analyzed | Method |
|---|--------|----------------|--------|
| 1 | **ELA** | JPEG re-compression artifacts | Re-save at quality 90, diff with original |
| 2 | **Noise Analysis** | Pixel-level inconsistencies | Laplacian filter standard deviation |
| 3 | **Edge Detection** | Unnatural edge patterns | Sobel filter distribution irregularity |
| 4 | **Color Consistency** | Splicing indicators | Quadrant-level mean color divergence |
| 5 | **Clone Detection** | Copy-pasted regions | ORB keypoint self-matching |
| 6 | **DCT Forgery** | Frequency-domain manipulation | High vs low DCT coefficient ratio |
| 7 | **Text Uniformity** | Font inconsistency across text | OCR bounding-box height variance |
| 8 | **Amount Sharpness** | Blur in transaction amount area | Laplacian variance in region of interest |
| 9 | **Semantic NLP** | Suspicious keywords, missing phrases | Regex + keyword frequency scoring |

---

## Scoring & Decision Logic

### Weighted Fusion

Each module returns a suspicion score `[0.0 – 1.0]`. Scores are inverted (`realness = 1 − suspicion`) and combined:

```
Final Confidence =
  Semantic   × 25%  +  Clone    × 15%  +  Forgery  × 15%
  Color      × 10%  +  Text     × 10%  +  Amount   × 10%
  ELA        ×  5%  +  Noise    ×  5%  +  Edge     ×  5%
```

Additional **penalty deductions** apply when clone, forgery, semantic, or text scores exceed `0.5`.  
A **+0.02 bias** is added for tall-aspect images identified as mobile screenshots.

### Verdict Thresholds

```
Confidence ≥ 0.65  →  ✅ REAL
Confidence ≤ 0.40  →  ❌ FAKE
Between 0.40–0.65  →  Majority side (> 0.50 = REAL, else FAKE)
```

### Hard Rules (Score Override)

Regardless of the weighted confidence, the verdict is **forced to FAKE** if any of the following trigger:

| Condition | Reason |
|-----------|--------|
| Recipient name `"Abdullah Kamran"` not found in OCR text | Wrong payee |
| No timestamp detected in the image | Likely edited out |
| Transaction timestamp > 3 minutes old | Replayed screenshot |

---

## Timestamp Recognition

The system handles all common Pakistani payment app time formats via OCR:

| Format | Example |
|--------|---------|
| Day Month Year + 12h | `26 Apr 2026, 10:53 PM` |
| Month Day Year + 12h | `Apr 26, 2026 10:53 PM` |
| ISO datetime | `2026-04-26 10:53` |
| Slash date + time | `26/04/2026 10:53 PM` |
| Time only (assumes today) | `10:53 PM` |
| 24-hour fallback | `22:53` |

All timestamps are compared against **Pakistan Standard Time (UTC+5)** using Python's built-in `datetime` — no `pytz` dependency required.

---

## Project Structure

```
Fake-Detect/
│
├── mainn.py          ← Forensic engine + Flask server + accuracy evaluator
├── index.html        ← Frontend UI (drag-and-drop screenshot uploader)
│
└── dataset/          ← Accuracy evaluation dataset
    ├── real/         ← Genuine transaction screenshots
    └── fake/         ← Fabricated / edited screenshots
```

---

## Installation

### Prerequisites

- Python 3.8 or higher
- [Tesseract OCR](https://github.com/UB-Mannheim/tesseract/wiki) installed on your system

### Step 1 — Install Python dependencies

```bash
pip install flask pillow opencv-python numpy scikit-image scipy pytesseract
```

### Step 2 — Configure Tesseract path

Open `mainn.py` and update the Tesseract path near the top of the file:

```python
# Windows
pytesseract.pytesseract.tesseract_cmd = r"C:\Program Files\Tesseract-OCR\tesseract.exe"

# Linux / macOS (usually on PATH already — this line can be omitted)
# pytesseract.pytesseract.tesseract_cmd = r"/usr/bin/tesseract"
```

### Step 3 — Run

```bash
python mainn.py
```

The server starts at `http://127.0.0.1:5000` and opens in your browser automatically.

---

## Accuracy Evaluation

At startup, the system runs a self-evaluation against the `dataset/` folder before launching the web server.

**Expected folder layout:**
```
dataset/
├── real/
│   ├── receipt_001.png
│   └── receipt_002.jpg
└── fake/
    ├── edited_001.png
    └── edited_002.jpg
```

**Terminal output:**
```
✅ receipt_001.png  →  CORRECT  (REAL)
✅ receipt_002.jpg  →  CORRECT  (REAL)
❌ edited_002.png   →  WRONG    (Predicted: REAL, Actual: FAKE)

══════════════════════════════
Total Images:        20
Correct Predictions: 17
Accuracy:            85.00%
══════════════════════════════
```

To skip evaluation and go straight to the web server, comment out the `evaluate_accuracy()` call at the bottom of `mainn.py`.

---

## API Reference

### `POST /analyze`

Accepts a screenshot upload and returns a full forensic breakdown.

**Request**
```
Content-Type: multipart/form-data
Body: file=<image file>
```

**Response**
```json
{
  "verdict": "FAKE",
  "confidence": 0.3821,
  "screenshot": true,
  "details": {
    "ela": 0.12,
    "noise": 0.08,
    "edge": 0.21,
    "color": 0.09,
    "clone": 0.55,
    "forgery": 0.61,
    "text": 0.14,
    "amount": 0.33,
    "semantic": 0.75
  },
  "recipient_valid": false,
  "recipient_check": "FAKE — Money not sent to Abdullah Kamran",
  "transaction_time_minutes": 47.3,
  "transaction_time_warning": "Transaction at 26 Apr 2026 10:53 PM PKT — 47.3 min(s) ago (exceeds 3-minute limit)",
  "transaction_time_str": "26 Apr 2026 10:53 PM",
  "ocr_debug": "..."
}
```

---

## Configuration

All tunable parameters are centralized in the `CONFIG` dictionary at the top of `mainn.py`:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `RESIZE` | `512` | Image resize dimension for uniform processing |
| `THRESHOLD_REAL` | `0.65` | Minimum confidence to classify as REAL |
| `THRESHOLD_FAKE` | `0.40` | Maximum confidence to classify as FAKE |
| `SCREENSHOT_BIAS` | `0.02` | Confidence boost for tall-aspect mobile screenshots |
| `WEIGHTS` | see above | Per-module contribution percentages |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Python, Flask |
| Image Processing | OpenCV, Pillow, scikit-image, SciPy |
| OCR | Tesseract via `pytesseract` |
| Numerical Computing | NumPy |
| Frontend | HTML, CSS, Vanilla JavaScript |

---

## Limitations

- **Recipient name is hardcoded** to `"Abdullah Kamran"` — intentional for the specific fraud scenario this system was built for. To generalize, make `check_recipient()` accept a configurable name parameter.
- **OCR accuracy** depends on image resolution and Tesseract version. Low-resolution screenshots may cause timestamp or recipient detection to fail.
- **3-minute window is strict by design.** Screenshots shared even a few minutes after a real transaction will trigger a time warning.
- **Optimized for Pakistani mobile payment apps** (Easypaisa, JazzCash, UBL Digital). Performance on other receipt formats is untested.

---

## Authors

**Abdullah Kamran · Mustafa Naeem · Mahad Khan**  
BS Computer Science — Semester 6 AI Project

---

<div align="center">

Built with Python, Flask, OpenCV, scikit-image, and Tesseract OCR

*Fake Detect — because a screenshot is not proof.*

</div>

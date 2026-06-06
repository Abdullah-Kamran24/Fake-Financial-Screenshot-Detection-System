<div align="center">

███████╗ █████╗ ██╗  ██╗███████╗    ██████╗ ███████╗████████╗███████╗ ██████╗████████╗██╗ ██████╗ ███╗   ██╗
██╔════╝██╔══██╗██║ ██╔╝██╔════╝    ██╔══██╗██╔════╝╚══██╔══╝██╔════╝██╔════╝╚══██╔══╝██║██╔═══██╗████╗  ██║
█████╗  ███████║█████╔╝ █████╗      ██║  ██║█████╗     ██║   █████╗  ██║        ██║   ██║██║   ██║██╔██╗ ██║
██╔══╝  ██╔══██║██╔═██╗ ██╔══╝      ██║  ██║██╔══╝     ██║   ██╔══╝  ██║        ██║   ██║██║   ██║██║╚██╗██║
██║     ██║  ██║██║  ██╗███████╗    ██████╔╝███████╗   ██║   ███████╗╚██████╗   ██║   ██║╚██████╔╝██║ ╚████║
╚═╝     ╚═╝  ╚═╝╚═╝  ╚═╝╚══════╝    ╚═════╝ ╚══════╝   ╚═╝   ╚══════╝ ╚═════╝   ╚═╝   ╚═╝ ╚═════╝ ╚═╝  ╚═══╝

### Fake Financial Screenshot Detection System

*Nine-layer AI forensics engine for Pakistani mobile payment fraud detection*

![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=flat-square&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-REST_API-000000?style=flat-square&logo=flask&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-Image_Forensics-5C3EE8?style=flat-square&logo=opencv&logoColor=white)
![Tesseract](https://img.shields.io/badge/Tesseract-OCR-FF6B35?style=flat-square)
![scikit-image](https://img.shields.io/badge/scikit--image-ELA_Analysis-F7931E?style=flat-square)
![NumPy](https://img.shields.io/badge/NumPy-Numerical_Computing-013243?style=flat-square&logo=numpy&logoColor=white)
![Version](https://img.shields.io/badge/Version-2.1-22C55E?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-6366F1?style=flat-square)

</div>

---

## The Problem

In Pakistan, peer-to-peer payments via **Easypaisa**, **JazzCash**, and **UBL Digital** are used by millions daily. So is a dangerously simple fraud: take a real transaction screenshot, edit the amount or recipient name in any photo editor, and present it as proof of payment.

Victims lose money. Sellers ship goods they were never paid for. Manual verification is slow, inconsistent, and trivially fooled by a careful edit.

**Fake Detect** eliminates this attack surface. It runs nine independent forensic signals across every uploaded screenshot — image compression artifacts, noise patterns, clone detection, frequency-domain analysis, OCR text validation, and real-time timestamp verification — and fuses them into a single weighted confidence score with a `REAL` or `FAKE` verdict in under two seconds.

---

## System Architecture

```
                    User uploads screenshot
                             │
                             ▼
                  ┌──────────────────────┐
                  │    Image Ingestion   │
                  │  Decode → NumPy      │
                  │  Resize to 512×512   │
                  └──────────┬───────────┘
                             │
          ┌──────────────────▼──────────────────────┐
          │           Forensic Pipeline              │
          │                                          │
          │  ┌──────────┐  ┌──────────┐  ┌────────┐ │
          │  │   ELA    │  │  Noise   │  │  Edge  │ │
          │  └──────────┘  └──────────┘  └────────┘ │
          │  ┌──────────┐  ┌──────────┐  ┌────────┐ │
          │  │  Color   │  │  Clone   │  │  DCT   │ │
          │  └──────────┘  └──────────┘  └────────┘ │
          │  ┌──────────┐  ┌──────────┐  ┌────────┐ │
          │  │   Text   │  │ Amount   │  │  NLP   │ │
          │  └──────────┘  └──────────┘  └────────┘ │
          │                                          │
          │   Each module → suspicion score 0.0–1.0  │
          └──────────────────┬──────────────────────┘
                             │
                    ┌────────▼────────┐
                    │ Weighted Fusion │  ← 9 scores → 1 confidence
                    │ Penalty Rules   │  ← Spikes on critical modules
                    │ Hard Rules      │  ← Recipient / timestamp override
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  REAL  /  FAKE  │  ← Verdict + full breakdown
                    └─────────────────┘
```

---

## Forensic Modules

| # | Module | What It Checks | Method Used |
|---|--------|---------------|-------------|
| 1 | **ELA — Error Level Analysis** | JPEG re-compression artifacts from editing | Re-save at quality 90, pixel-diff with original |
| 2 | **Noise Analysis** | Pixel-level inconsistencies across the image | Laplacian convolution standard deviation |
| 3 | **Edge Detection** | Unnatural edge sharpness patterns | Sobel filter distribution irregularity |
| 4 | **Color Consistency** | Spliced or replaced image regions | Quadrant-level mean color divergence |
| 5 | **Clone Detection** | Copy-pasted areas (e.g. duplicated amounts) | ORB keypoint self-matching |
| 6 | **DCT Forgery Analysis** | Frequency-domain manipulation signatures | High vs low DCT coefficient ratio |
| 7 | **Text Uniformity** | Font-size inconsistency across text regions | OCR bounding-box height variance |
| 8 | **Amount Region Sharpness** | Blur in the transaction amount area | Laplacian variance in region of interest |
| 9 | **Semantic NLP** | Suspicious keywords, missing required phrases, wrong recipient | Regex + keyword frequency scoring |

---

## Scoring & Decision Logic

### Weighted Confidence Fusion

Each module returns a suspicion score between `0.0` (genuine) and `1.0` (suspicious).  
Scores are inverted (`realness = 1 − suspicion`) and combined with these weights:

```
┌─────────────────────────────────────────────────────────┐
│                  Confidence Formula                     │
│                                                         │
│  Semantic NLP       ×  25%   (highest weight)           │
│  Clone Detection    ×  15%                              │
│  DCT Forgery        ×  15%                              │
│  Color Consistency  ×  10%                              │
│  Text Uniformity    ×  10%                              │
│  Amount Sharpness   ×  10%                              │
│  ELA                ×   5%                              │
│  Noise Analysis     ×   5%                              │
│  Edge Detection     ×   5%                              │
│                          ──                             │
│                        100%                             │
└─────────────────────────────────────────────────────────┘
```

**Additional penalty deductions** apply when clone, forgery, semantic, or text scores individually exceed `0.5` — a single strong signal can pull the verdict toward FAKE even if other modules score clean.

**+0.02 bias** added for tall-aspect-ratio images identified as mobile screenshots.

### Verdict Thresholds

```
Confidence ≥ 0.65          →   ✅  REAL
Confidence ≤ 0.40          →   ❌  FAKE
Between 0.40 and 0.65      →   Majority side (> 0.50 = REAL, else FAKE)
```

### Hard Rules — Score Override

These rules **force a FAKE verdict** regardless of the weighted score:

| Hard Rule | Trigger Condition |
|-----------|------------------|
| Wrong recipient | `"Abdullah Kamran"` not found in OCR-extracted text |
| No timestamp | No recognizable date/time detected in the image |
| Stale screenshot | Transaction timestamp is more than **3 minutes** in the past |

---

## Timestamp Recognition

The OCR engine handles all common Pakistani payment app timestamp formats:

| Format | Example |
|--------|---------|
| Day Month Year + 12h time | `26 Apr 2026, 10:53 PM` |
| Month Day Year + 12h time | `Apr 26, 2026 10:53 PM` |
| ISO datetime | `2026-04-26 10:53` |
| Slash date + time | `26/04/2026 10:53 PM` |
| Time only (assumes today) | `10:53 PM` |
| 24-hour fallback | `22:53` |

All timestamps are compared against **Pakistan Standard Time (UTC+5)** using Python's built-in `datetime` module — no external `pytz` dependency required.

---

## Project Structure

```
Fake-Detect/
│
├── mainn.py              ← Forensic engine, Flask server, accuracy evaluator
├── index.html            ← Frontend UI — drag-and-drop screenshot uploader
│
└── dataset/              ← Labeled dataset for accuracy evaluation
    ├── real/             ← Genuine transaction screenshots
    └── fake/             ← Fabricated or edited screenshots
```

---

## Installation

### Prerequisites

- Python **3.8** or higher
- [Tesseract OCR](https://github.com/UB-Mannheim/tesseract/wiki) installed on your system

### Step 1 — Install Python dependencies

```bash
pip install flask pillow opencv-python numpy scikit-image scipy pytesseract
```

### Step 2 — Configure Tesseract path

Open `mainn.py` and update the path near the top of the file:

```python
# Windows
pytesseract.pytesseract.tesseract_cmd = r"C:\Program Files\Tesseract-OCR\tesseract.exe"

# Linux / macOS — usually already on PATH, this line can be removed
# pytesseract.pytesseract.tesseract_cmd = r"/usr/bin/tesseract"
```

### Step 3 — Run the application

```bash
python mainn.py
```

The server starts at `http://127.0.0.1:5000` and opens in your default browser automatically.

---

## Accuracy Evaluation

At startup, the system **automatically evaluates itself** against the `dataset/` folder before launching the web server. No separate command needed.

**Expected folder structure:**

```
dataset/
├── real/
│   ├── receipt_001.png
│   ├── receipt_002.jpg
│   └── ...
└── fake/
    ├── edited_001.png
    ├── edited_002.jpg
    └── ...
```

**Terminal output:**

```
✅  receipt_001.png   →  CORRECT   (REAL)
✅  receipt_002.jpg   →  CORRECT   (REAL)
❌  edited_002.png    →  WRONG     (Predicted: REAL, Actual: FAKE)
✅  edited_003.png    →  CORRECT   (FAKE)

══════════════════════════════════════
  Total Images:         20
  Correct Predictions:  17
  Accuracy:             85.00%
══════════════════════════════════════
```

To skip evaluation and jump straight to the web server, comment out the `evaluate_accuracy()` call at the bottom of `mainn.py`.

---

## API Reference

### `POST /analyze`

Accepts a screenshot upload and returns a complete forensic breakdown.

**Request**

```
Content-Type: multipart/form-data
Body:         file=<image file>
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

**Response fields explained:**

| Field | Description |
|-------|-------------|
| `verdict` | `"REAL"` or `"FAKE"` |
| `confidence` | Final weighted score `[0.0 – 1.0]` |
| `screenshot` | `true` if image identified as a mobile screenshot |
| `details` | Per-module suspicion scores `[0.0 – 1.0]` |
| `recipient_valid` | `false` if recipient name check failed |
| `transaction_time_minutes` | Minutes elapsed since transaction timestamp |
| `transaction_time_warning` | Human-readable timestamp warning if stale |
| `ocr_debug` | Raw OCR text extracted from image for debugging |

---

## Configuration

All tunable parameters are centralized in the `CONFIG` dictionary at the top of `mainn.py`:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `RESIZE` | `512` | Image resize dimension for uniform processing |
| `THRESHOLD_REAL` | `0.65` | Minimum confidence to classify as REAL |
| `THRESHOLD_FAKE` | `0.40` | Maximum confidence to classify as FAKE |
| `SCREENSHOT_BIAS` | `0.02` | Confidence boost for tall-aspect mobile screenshots |
| `TIME_LIMIT_MINUTES` | `3` | Maximum allowed age of a transaction timestamp |
| `WEIGHTS` | see above | Per-module percentage contribution to final score |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Python, Flask |
| Image Processing | OpenCV, Pillow, scikit-image, SciPy |
| Frequency Analysis | NumPy (DCT via `numpy.fft`) |
| OCR | Tesseract via `pytesseract` |
| Frontend | HTML, CSS, Vanilla JavaScript |

---

## Limitations

**Recipient name is hardcoded** to `"Abdullah Kamran"` — intentional for the specific fraud scenario this system was built for. To adapt for general use, refactor `check_recipient()` to accept a configurable name at runtime.

**OCR accuracy** depends on image resolution and Tesseract version. Low-resolution or heavily compressed screenshots may cause timestamp or recipient extraction to fail silently.

**The 3-minute window is strict by design.** A genuine screenshot shared even four minutes after the transaction will trigger a time warning. Adjust `TIME_LIMIT_MINUTES` in `CONFIG` if your use case allows more flexibility.

**Optimized for Pakistani mobile payment apps** — Easypaisa, JazzCash, UBL Digital. Performance on receipts from other payment platforms is untested.

---

## Future Improvements

- [ ] Make recipient name configurable via environment variable or request parameter
- [ ] GPU-accelerated ELA and DCT processing for batch evaluation
- [ ] Support for video-recorded screen captures (frame extraction)
- [ ] Confidence calibration using a larger labeled dataset
- [ ] REST API authentication for production deployment
- [ ] Detailed PDF forensic report export per analysis

---

## Authors

**Abdullah Kamran · Mustafa Naeem · Mahad Khan**

BS Computer Science — Semester 6 AI Project

---

<div align="center">

Built with Python · Flask · OpenCV · scikit-image · Tesseract OCR

*Fake Detect — because a screenshot is not proof.*

</div>

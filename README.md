<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:0f172a,35:1e3a8a,70:2563eb,100:10b981&height=220&section=header&text=Facial%20Biometrics%20Attendance&fontSize=42&fontColor=ffffff&desc=Real-Time%20Face%20Detection%20%E2%80%A2%20LBPH%20Recognition%20%E2%80%A2%20Automated%20Attendance%20Tracking&descFontSize=16&descAlignY=68&animation=fadeIn" alt="Facial Biometrics Attendance Banner" width="100%" />
</p>

<p align="center">
  <a href="https://www.python.org/downloads/"><img src="https://img.shields.io/badge/Python-3.8%2B-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python" /></a>
  <a href="https://opencv.org/"><img src="https://img.shields.io/badge/OpenCV-Contrib_4.5%2B-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white" alt="OpenCV" /></a>
  <a href="https://numpy.org/"><img src="https://img.shields.io/badge/NumPy-Array_Math-013243?style=for-the-badge&logo=numpy&logoColor=white" alt="NumPy" /></a>
  <a href="https://pandas.pydata.org/"><img src="https://img.shields.io/badge/Pandas-Data_Logging-150458?style=for-the-badge&logo=pandas&logoColor=white" alt="Pandas" /></a>
  <a href="https://docs.python.org/3/library/tkinter.html"><img src="https://img.shields.io/badge/GUI-Tkinter-FF6F00?style=for-the-badge" alt="Tkinter GUI" /></a>
  <a href="#license"><img src="https://img.shields.io/badge/License-MIT-blue?style=for-the-badge" alt="License MIT" /></a>
</p>

---

## 📌 Executive Overview

**Facial Biometrics for Seamless Attendance Tracking** is an automated, non-intrusive computer vision system designed to eliminate the inefficiencies and proxy vulnerabilities of traditional attendance tracking in educational institutions and enterprise environments.

Manual paper roll calls and magnetic swipe cards suffer from high time latency, human recording errors, and buddy punching. This project automates the entire pipeline:
1. **Face Detection**: Rapid localization of human facial bounding boxes via Haar Feature-based Cascade Classifiers.
2. **Feature Extraction & Learning**: Texture analysis and histogram modeling using Local Binary Patterns Histograms (LBPH).
3. **Real-Time Recognition & Verification**: Low-latency video stream matching against registered student models with confidence-gated thresholds.
4. **Automated Auditing**: Deduplicated, timestamped CSV attendance reporting, alongside automatic capture of unrecognized individuals for campus security.

---

## ⚡ Key Engineering Features

- 📸 **Automated Sample Capture Pipeline**: Ingests 60 normalized facial crops per student via active webcam streaming, ensuring training variations across minor head tilts and expressions.
- 🎯 **Haar Cascade Feature Localization**: Multi-scale sliding window detection (`haarcascade_frontalface_default.xml`) optimized for real-time inference on standard CPU hardware.
- 🧠 **Illumination-Invariant LBPH Recognizer**: Employs $3 \times 3$ pixel neighborhood thresholding to create texture histograms resilient to ambient classroom lighting variations.
- 🛡️ **Confidence-Gated Validation**:
  - `Confidence < 50`: Verified positive match &rarr; logged to attendance roster.
  - `Confidence >= 50`: Unverified &rarr; labeled as `Unknown`.
  - `Confidence > 75`: Flagged intruder &rarr; cropped frame automatically exported to `ImagesUnknown/` for audit review.
- ⏱️ **Timestamped Audit Logging**: Generates dated CSV attendance reports (`Attendance_YYYY-MM-DD_HH-MM-SS.csv`) with automatic duplicate suppression (ensuring only first check-in is logged per session).
- 🖥️ **Integrated Desktop GUI**: Built using Python Tkinter, providing an accessible visual interface for sample registration, model training, and active tracking.

---

## 🏛️ System Architecture

```mermaid
%%{init: {"theme": "base", "themeVariables": {"primaryColor": "#e0f2fe", "primaryBorderColor": "#2563eb", "lineColor": "#0284c7", "fontFamily": "Inter, Arial", "tertiaryColor": "#f8fafc"}}}%%
flowchart TB
  classDef cInput fill:#f1f5f9,stroke:#64748b,stroke-width:2px,color:#0f172a;
  classDef cVision fill:#e0f2fe,stroke:#2563eb,stroke-width:2px,color:#0f172a;
  classDef cML fill:#ede9fe,stroke:#7c3aed,stroke-width:2px,color:#0f172a;
  classDef cGate fill:#fef3c7,stroke:#f59e0b,stroke-width:2px,color:#0f172a;
  classDef cStore fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#0f172a;
  classDef cAlert fill:#fee2e2,stroke:#ef4444,stroke-width:2px,color:#0f172a;

  subgraph Ingestion["1 · Video Acquisition"]
    Cam["Webcam / Video Stream<br/>cv2.VideoCapture(0)"]
    Gray["Grayscale Conversion<br/>cv2.COLOR_BGR2GRAY"]
    Cam --> Gray
  end

  subgraph Detection["2 · Face Detection"]
    Cascade["Haar Cascade Classifier<br/>haarcascade_frontalface_default.xml"]
    BBox["Bounding Box Extraction<br/>faces = detectMultiScale(gray, 1.2, 5)"]
    Gray --> Cascade --> BBox
  end

  subgraph RecognitionEngine["3 · Feature Modeling & Inference"]
    LBPH["LBPH Face Recognizer<br/>cv2.face.LBPHFaceRecognizer_create()"]
    ModelFile[("Trained Weights<br/>TrainingImageLabel/Trainner.yml")]
    Predict["Prediction & Distance Scoring<br/>Id, conf = recognizer.predict(crop)"]
    ModelFile --> LBPH
    BBox --> Predict
    LBPH --> Predict
  end

  subgraph ConfidenceGate["4 · Decision Boundary"]
    ConfCheck{"Confidence Threshold<br/>Check"}
    Predict --> ConfCheck
  end

  subgraph StorageAuditing["5 · Attendance & Security Logs"]
    Registry[("Student Registry<br/>StudentDetails.csv")]
    AttendLog[("Timestamped Attendance<br/>Attendance_DATE_TIME.csv")]
    IntruderLog[("Security Anomaly Store<br/>ImagesUnknown/")]
  end

  ConfCheck -->|conf under 50: Verified| Match["Match ID with Registry"]
  Match --> Registry
  Match --> Dedup["Deduplicate Entries"]
  Dedup --> AttendLog

  ConfCheck -->|conf >= 50: Unverified| Unknown["Mark Label as 'Unknown'"]
  ConfCheck -->|conf > 75: High Anomaly| IntruderLog

  class Cam,Gray cInput
  class Cascade,BBox cVision
  class LBPH,ModelFile,Predict cML
  class ConfCheck cGate
  class Registry,AttendLog,Dedup,Match cStore
  class Unknown,IntruderLog cAlert
```

---

## 🔄 Tri-Phase Operational Workflow

The system operates across three distinct lifecycles:

```mermaid
%%{init: {"theme": "base", "themeVariables": {"primaryColor": "#f8fafc", "primaryBorderColor": "#64748b", "lineColor": "#2563eb", "fontFamily": "Inter, Arial"}}}%%
flowchart LR
  classDef cStep fill:#f8fafc,stroke:#64748b,stroke-width:2px,color:#0f172a;
  classDef cProcess fill:#e0f2fe,stroke:#2563eb,stroke-width:2px,color:#0f172a;
  classDef cSuccess fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#0f172a;

  subgraph Phase1["Phase 1: Student Enrollment"]
    P1A["Enter Student ID & Name"] --> P1B["Burst Capture (60 frames)"]
    P1B --> P1C["Store to TrainingImage/ & StudentDetails.csv"]
  end

  subgraph Phase2["Phase 2: Model Training"]
    P2A["Load Crops & Parse IDs"] --> P2B["Compute 2D LBP Operators"]
    P2B --> P2C["Train & Save Trainner.yml"]
  end

  subgraph Phase3["Phase 3: Live Tracking"]
    P3A["Stream Live Webcam"] --> P3B["Predict ID + Confidence"]
    P3B --> P3C["Export Attendance CSV on 'q'"]
  end

  Phase1 --> Phase2 --> Phase3

  class P1A,P2A,P3A cStep
  class P1B,P2B,P3B cProcess
  class P1C,P2C,P3C cSuccess
```

---

## 🔬 Algorithmic Foundations

### 1. Haar Feature-Based Cascade Detection
Faces are detected using Paul Viola and Michael Jones' rapid object detection framework:
- **Integral Image Representation**: Allows rectangular Haar-like features (edge, line, and four-rectangle features) to be evaluated in constant time $O(1)$.
- **AdaBoost Classifier Selection**: Isolates a minimal set of critical features out of thousands of candidates.
- **Attentional Cascade**: Evaluates stages sequentially; non-face background regions are rejected in early stages, reserving heavy computation strictly for candidate facial sub-windows.

### 2. Local Binary Patterns Histograms (LBPH)
Rather than viewing facial images as holistic matrices, LBPH models micro-level surface textures:
1. **Neighborhood Thresholding**: For every pixel $p_c$, compare its intensity with its 8 adjacent circular neighbors $p_n$:
   $$\text{LBP}(x_c, y_c) = \sum_{n=0}^{7} s(i_n - i_c) \cdot 2^n \quad \text{where} \quad s(x) = \begin{cases} 1 & x \ge 0 \\ 0 & x < 0 \end{cases}$$
2. **Spatial Histogram Division**: The resulting LBP matrix is partitioned into uniform $m \times m$ grid cells.
3. **Concatenated Feature Vector**: Histograms from all sub-regions are computed and concatenated into a high-dimensional feature representation.
4. **Classification via Distance Metrics**: Recognition uses Chi-Square distance $\chi^2$ to measure divergence between the candidate face histogram and stored reference vectors:
   $$\chi^2(H_{\text{cand}}, H_{\text{ref}}) = \sum_{i} \frac{(H_{\text{cand}}(i) - H_{\text{ref}}(i))^2}{H_{\text{cand}}(i) + H_{\text{ref}}(i)}$$

---

## 📂 Repository Directory Structure

```text
Facial-Biometrics/
├── haarcascade_frontalface_default.xml   # Pre-trained Haar Cascade facial detector model
├── train.py                             # Core application (Tkinter GUI, capture, training, tracking)
├── setup.py                             # cx_Freeze standalone binary build script
├── requirements.txt                     # Pinned Python package dependencies
├── StudentDetails/                      # Master student records registry
│   └── StudentDetails.csv               # CSV database containing registered [Id, Name]
├── TrainingImage/                       # Raw facial dataset directory (60 crops per identity)
│   ├── Pavan.520.1.jpg                  # Standard format: [Name].[ID].[SampleNumber].jpg
│   └── ...
├── TrainingImageLabel/                  # Model weights output directory
│   └── Trainner.yml                     # Exported LBPH trained classifier weights (generated)
├── Attendance/                          # Timestamped session logs (generated on tracking exit)
│   └── Attendance_2026-10-05_18-30-00.csv
└── ImagesUnknown/                       # Security audit directory for unidentified intruders
```

---

## 🛠️ Installation & Setup

### 1. Prerequisites
- **Python**: Version `3.8` to `3.11` recommended.
- **Webcam**: Integrated or USB video camera supported by OpenCV index `0`.

### 2. Clone the Repository
```bash
git clone https://github.com/Pavan048/Facial-Biometrics-For-Seamless-Attendance-Tracking-In-Educational-Institutions-.git
cd Facial-Biometrics-For-Seamless-Attendance-Tracking-In-Educational-Institutions-
```

### 3. Create & Activate Virtual Environment
```bash
# On Linux/macOS:
python3 -m venv venv
source venv/bin/activate

# On Windows (PowerShell):
python -m venv venv
.\venv\Scripts\Activate.ps1
```

### 4. Install Dependencies

> [!IMPORTANT]
> The LBPH recognizer (`cv2.face`) is part of OpenCV's extra modules. You must install **`opencv-contrib-python`**, not base `opencv-python`.

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

---

## 🚀 Usage Guide

Launch the graphical interface:
```bash
python train.py
```

### Step 1: Student Enrollment ("Take Images")
1. Enter a numeric ID into the **Enter ID** field (e.g. `520`).
2. Enter the student's name into the **Enter Name** field (alphabetical characters only).
3. Click **Take Images**.
4. The system opens the camera feed, captures 60 cropped grayscale face samples, writes them to `TrainingImage/`, and records the student in `StudentDetails/StudentDetails.csv`.

### Step 2: Train Classifier ("Train Images")
1. Click **Train Images**.
2. The script processes all image samples in `TrainingImage/`, maps them to their respective student IDs, and computes the LBPH texture histograms.
3. The trained classifier is serialized and saved to `TrainingImageLabel/Trainner.yml`.

### Step 3: Real-Time Tracking & Attendance Logging ("Track Images")
1. Click **Track Images**.
2. The live detection loop initiates:
   - Faces detected with high confidence are labeled with their `ID-Name` in green/blue overlays.
   - Ambiguous or unrecognized faces are labeled as `Unknown`.
   - Any unknown faces with confidence $> 75$ are snapped and stored in `ImagesUnknown/`.
3. Press **`Q`** on your keyboard to terminate tracking.
4. An attendance sheet is automatically compiled with duplicates removed and exported to `Attendance/Attendance_<Date>_<Time>.csv`.

---

## 📊 Data Contracts & Schemas

### 1. Student Master Registry (`StudentDetails/StudentDetails.csv`)
| Field | Type | Description | Example |
|---|---|---|---|
| `Id` | Integer | Unique student enrollment identifier | `520` |
| `Name` | String | Alphabetical student name | `Pavan` |

### 2. Attendance Report Sheet (`Attendance/Attendance_YYYY-MM-DD_HH-MM-SS.csv`)
| Field | Type | Description | Example |
|---|---|---|---|
| `Id` | Integer | Verified student ID | `520` |
| `Name` | String | Verified student name | `Pavan` |
| `Date` | Date String | Date of attendance entry (`YYYY-MM-DD`) | `2026-10-05` |
| `Time` | Time String | Initial capture timestamp (`HH:MM:SS`) | `18:30:15` |

---

## 🗺️ Roadmap & Future Enhancements

- [x] Haar Cascade multi-scale face localization
- [x] LBPH histogram training and YAML weight serialization
- [x] Confidence-gated intruder detection and automated image capture
- [x] CSV duplicate suppression for daily attendance rosters
- [ ] **Anti-Spoofing & Liveness Detection**: Implement eye-blink frequency and texture reflection checks to prevent photo-presentation attacks.
- [ ] **Deep Learning Feature Extraction**: Upgrade from LBPH to Deep Learning embeddings (FaceNet / InsightFace ArcFace) for 99%+ accuracy at distance.
- [ ] **Web & Mobile Dashboard**: Migration from Tkinter desktop GUI to FastAPI backend with React dashboard.

---

## 📄 License

This repository is distributed under the **MIT License**. See the [LICENSE](LICENSE) file for more information.

# 🌊 AquaMorph — Microplastic Morphology Classifier

> Theme: Marine Ecosystem Protection & Water Quality Monitoring

---

## 📽️ Demo Video
▶️ [Watch Demo Video]https://drive.google.com/file/d/1NN_VU7zDTB2pvGUqbP6io8ZcJAt-oihn/view?usp=sharing

---

## 🖥️ App Screenshots

### Batch Analysis with ETI Scoring
[Batch](https://github.com/user-attachments/assets/a1cbf890-e859-4911-bc04-9044c78a026c)

<img width="1920" height="1080" alt="Classification" src="https://github.com/user-attachments/assets/2e5c243b-37dc-4fa5-9fbe-fcd9568b1c78" />

![GAN](https://github.com/user-attachments/assets/1056585a-27d7-4790-a22c-dbfe765d17c6)



---

## 📁 Datasets Used & Preprocessing

### Datasets

| Dataset | Source | Train Images | Val Images | Classes |
|---|---|---|---|---|
| microplastic_100 | [Roboflow Universe](https://universe.roboflow.com/kueranan/microplastic_100) | 210 | 30 | fiber, film, fragment, pallet |
| Microplastic-v2 | [Roboflow Universe](https://universe.roboflow.com/research-new-things-m0fiq/microplastic-v2-wowak) | 1,461 | 140 | bead, fiber, fragment |
| **Merged** | Combined | **1,671** | **170** | **fiber, fragment, film, bead** |

> 📂 Full datasets + trained model: [Google Drive](https://drive.google.com/drive/folders/1eHOGMavDzvi_qJu60wmKcRS0lirIFx5u?usp=drive_link)

### Class Standardization

| Dataset 1 original | Dataset 2 original | AquaMorph unified class |
|---|---|---|
| fiber | fiber | `fiber` |
| fragment | fragment | `fragment` |
| film | _(not present)_ | `film` |
| pallet | bead | `bead` |

### Preprocessing Pipeline

1. **Merging** — Images from both datasets merged into unified folder with remapped class labels
2. **Resize** — All images standardized to 640×640 for YOLOv8 input
3. **Augmentation** (applied by YOLOv8 during training):
   - Random horizontal flip (p=0.5)
   - HSV colour jitter (hue ±1.5%, saturation ±70%, value ±40%)
   - Random erasing (p=0.4)
   - Mosaic augmentation (4 images per tile)
4. **Normalization** — Pixel values normalized to [0, 1]
5. **Class imbalance** — Handled via augmentation; film class supplemented from Dataset 1

### Size Estimation Preprocessing (OpenCV)

1. Grayscale conversion
2. Gaussian blur (5×5 kernel) for noise reduction
3. Otsu's adaptive thresholding to isolate particle from background
4. `cv2.findContours()` to detect particle boundary
5. Maximum pairwise distance across contour points = Feret diameter
6. Pixel → micrometre conversion at **0.25 µm/pixel** (40× magnification)

---

## 🤖 Model Used & Performance Metrics

### Architecture
![Screenshot 2]!<img width="987" height="612" alt="System" src="https://github.com/user-attachments/assets/7fa186bb-4da6-413b-8e06-e14a86d013ef" />


| Property | Value |
|---|---|
| Model | YOLOv8n (nano) |
| Pretrained on | COCO dataset |
| Task | Object Detection + Classification |
| Parameters | 3,006,428 |
| Model size | 6.2 MB |
| Input size | 640×640 |
| Training epochs | 30 |
| Optimizer | Auto (SGD → Adam) |
| Batch size | 16 |
| Hardware | Tesla T4 GPU (Google Colab) |
| Training time | ~16 minutes |

### Performance Metrics (Final Validation)

| Class | Precision | Recall | mAP50 | mAP50-95 |
|---|---|---|---|---|
| **fiber** | 0.923 | 0.961 | **0.956** | 0.775 |
| **fragment** | 0.937 | 0.970 | **0.959** | 0.595 |
| **film** | 0.972 | 0.977 | **0.958** | 0.430 |
| **bead** | 0.801 | 0.825 | **0.816** | 0.515 |
| **Overall** | **0.908** | **0.933** | **0.922** | **0.578** |

- **Overall mAP50: 92.2%** — production-quality performance
- **Training loss at epoch 30:** box=0.912, cls=0.514, dfl=1.001
- **Inference speed:** 4.1ms per image on GPU

### Training Output
![Training Results](https://github.com/user-attachments/assets/0c9d373b-f48b-43b2-b1e9-ffa16f8e2b37)

### Ecological Threat Index (
ETI) Formula
```
ETI = morphology_score + size_score

morphology_score:
  fiber     = 40  (highest — ingested by filter feeders)
  fragment  = 30  (leaches chemical additives)
  film      = 25  (smothers benthic organisms)
  bead      = 20  (point source, predictable)

size_score = 60 × (1 − min(feret_µm, 5000) / 5000)
  → particles < 100µm can penetrate biological tissues
  → particles < 10µm can cross cell membranes

ETI range: 0–100  (higher score = greater ecological threat)
```

---

## ✨ Key Features

### Core Classification Features

| Feature | Description |
|---|---|
| 🔍 **Morphology classification** | Classifies into Fiber, Fragment, Film, Bead with confidence % |
| 📏 **Feret diameter estimation** | Estimates longest particle dimension in micrometres (µm) via OpenCV contour detection |
| 🌡️ **Ecological Threat Index** | Combined 0–100 risk score — ETI Score shown per particle |
| 🚦 **Threat level badge** | HIGH RISK / MEDIUM / LOW colour-coded alert per prediction |
| 🧪 **Polymer source predictor** | Maps morphology → likely polymer (e.g. Polyester/Nylon for fiber) |
| 🌿 **Degradation stage estimator** | Classifies particle as Fresh / Lightly Weathered / Heavily Weathered |

### Analysis Modules (from sidebar)

| Module | Description |
|---|---|
| 🔍 **Classify** | Single image upload → full morphology + ETI analysis |
| 📦 **Batch Analysis** | Upload multiple images → summary table with Class, Confidence, Feret Diameter, ETI Score, Threat Level, Degradation, Polymer → Download Full Results CSV |
| 🗺️ **Spatial Heatmap** | Multi-particle field image → colour-coded particle distribution map |
| 🎨 **GAN Generator** | Generate synthetic microplastic training images per class |
| 🧪 **Polymer Source** | Detailed polymer origin analysis per morphology type |
| 📍 **Field Logger** | Log GPS coordinates + sample source + date for each analysis |
| ⚠️ **Review Queue** | Low-confidence predictions flagged for manual human review |
| 📊 **Dashboard** | Summary stats — Images Processed, Avg ETI Score, High Threat count |
| ℹ️ **About** | Indian river context, model info, ETI formula explanation |

### System Status Panel

| Status | Component |
|---|---|
| 🟢 Ready | YOLOv8 model loaded |
| 🔵 Ready | Contour Analyser (OpenCV) |
| 🔴 Standby | DCGAN Generator |

### 🇮🇳 Indian Environmental Context

AquaMorph is calibrated against published Indian river research:

- **Pune — Mula River** → 1,808 microplastic particles/L detected (2025 study); film-shaped MPs most abundant; particles as small as 25µm *(our backyard)*
- **Ganga & Yamuna Rivers** → Fibre-shaped MPs dominant at 300µm–5mm; polyamide and PAM primary polymers
- **South Indian rivers** → Fibers 64.1%, films 21.7%, fragments 12%, beads 2.2%

---

## 🏗️ Model Architecture
```
Input Image (640×640)
        ↓
YOLOv8n Backbone (CSPDarknet)
        ↓
Feature Pyramid Network (FPN)
        ↓
Detection Head
        ↓
┌─────────────────────────────────────────┐
│  Class: fiber / fragment / film / bead  │
│  Confidence %                           │
│  Bounding box coordinates               │
└─────────────────────────────────────────┘
        ↓
OpenCV Contour Detection → Feret Diameter (µm)
        ↓
Degradation Stage Estimator
        ↓
ETI Score (0–100) + Threat Level + Polymer Source
        ↓
Streamlit UI — Classify / Batch / Heatmap / GAN / Dashboard
```

---

## 🚀 How to Run AquaMorph

### Step 1 — Clone the repo
```bash
git clone https://github.com/Saanidhi123/cummins_hackathon.git
cd cummins_hackathon
pip install -r requirements.txt
```

### Step 2 — Download the model
Download `best.pt` from Google Drive and place it inside the `model/` folder:

📁 [AquaMorph Model + Datasets (Google Drive)](https://drive.google.com/drive/folders/1eHOGMavDzvi_qJu60wmKcRS0lirIFx5u?usp=drive_link)

### Step 3 — Run
```bash
streamlit run app.py
```
Open `http://localhost:8501` in your browser.

---

## 📂 Repository Structure
```
cummins_hackathon/
├── app.py                  ← AquaMorph Streamlit web app
├── train.py                ← YOLOv8 training script
├── requirements.txt        ← Python dependencies
├── README.md
├── assets/                 ← Screenshots and training output images
│   ├── training_results.png
│   ├── screenshot_batch.png
│   └── screenshot_classify.png
├── utils/
│   ├── contour.py          ← OpenCV Feret diameter estimation
│   └── scoring.py          ← ETI score + polymer + degradation predictor
└── model/
    └── .gitkeep            ← Download best.pt from Drive link above
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| ML Model | YOLOv8n (Ultralytics) |
| Image Processing | OpenCV |
| GAN Generator | DCGAN (PyTorch) |
| Frontend / UI | Streamlit |
| Data Management | Pandas |
| Training Platform | Google Colab (Tesla T4 GPU) |
| Dataset Source | Roboflow Universe |
| Model Storage | Google Drive |
| Version Control | GitHub |

---

## 👩‍💻 Team — NeuralNomads_PS3

**AquaMorph**

Microplastic Morphology Classifier

---

## 📄 License

Datasets used under CC BY 4.0 license (Roboflow Universe).

# 🚧 Temporary Traffic Sign Detection

This project focuses on detecting **temporary traffic signs** (e.g., "Under Construction", "Detour") that appear in specific situations like road work or accidents — signs that are essential for autonomous driving but often missing from standard datasets.

We combine **YOLOv8**, **PaddleOCR**, and **DeepSORT** to detect, recognize, and track temporary signs in real-world driving scenes.

---

## 📁 Dataset

- **Base Dataset**: [BDD100K](https://bdd-data.berkeley.edu/)
  - 70,000 images (train) / 10,000 images (val)
  - Filtered for `traffic sign` annotations → 11,000 (train) / 1,650 (val)
- **Custom Dataset**:
  - 489 temporary sign images crawled from Google Image Search
  - Manually annotated using **CVAT**
  - Objects: `sign`, `sign_word`, `direction`, `hurdle`, `working_truck`, `tripod`

---

## 🧠 Model Training

- **Model**: `YOLOv8n` (3.2M parameters)
- **Training**:
  - First stage: BDD100K only → mAP@50 = 0.594
  - Second stage: Fine-tuned with custom temporary sign data (150 epochs, early stopping)
  - Final performance:
    - `sign`: 0.881  
    - `sign_word`: 0.817  
    - **Overall mAP@50**: 0.68

---

## 🔍 OCR Integration

- OCR Models Tested:
  - ❌ Tesseract: slow, poor Korean support  
  - ❌ EasyOCR: fast but sensitive to lighting/skew  
  - ❌ TrOCR: accurate but large and slow  
  - ✅ **PaddleOCR**: balanced in speed and accuracy for Korean traffic signs

- **Post-processing**:
  - Predefined dictionary of common sign phrases (e.g., "공사중", "우회도로")  
  - Fuzzy matching (`RapidFuzz`) used to clean and standardize OCR results

- **OCR Triggering Strategy**:
  - OCR runs only **when YOLO detects a new object ID**  
  - For tracked objects, OCR is re-applied only if `ocr_interval` has passed (e.g., 0.2 or 0.4 sec)  
  - Helps reduce redundant OCR and filter out unrelated text (e.g., shop signs)

---

## 🎥 Tracking with DeepSORT

- Maintains consistent ID for each sign across frames
- Handles temporary occlusions (e.g., tree-covered signs)
- Reduces unnecessary OCR by avoiding repeated reads of the same object

---

## ⚠️ Limitations

- PaddleOCR's bounding boxes did not perfectly align with actual text, preventing implementation of IoU-based filtering  
- OCR applied to the entire frame may still pick up unrelated text if not spatially constrained  
- Only tested in **daylight conditions** — performance in low-light/nighttime scenarios is limited

---

## 🔮 Future Work

- Implement IoU-based filtering between OCR and YOLO-detected boxes once box alignment issues are resolved  
- Optimize the model pipeline for real-time use on edge devices  
- Expand to handle **text + directional** understanding (e.g., "Detour → Left")  
- Refine `dictionary.txt` to improve text recognition accuracy  
- Integrate with a robotics platform so that detected signs can trigger physical actions

---

## 🧪 Try It Yourself

📄 [Project Documentation on Notion](https://invincible-gargoyle-054.notion.site/Temporary-Traffic-Sign-Detection-22bc4ba53ecb800c92e1e5cd24aae32e)

---

## 🗂 Output Format

All recognized signs are saved to a text file with the following format:


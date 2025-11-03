# การสร้าง Model as a Service ด้วย Flask และ Docker

## 1. วัตถุประสงค์

1. ทำความเข้าใจแนวคิดของการให้บริการโมเดล Machine Learning ในรูปแบบ Model as a Service
2. ฝึกสร้างเว็บแอปพลิเคชันโดยใช้ Flask เพื่อรับข้อมูลและแสดงผลการทำนาย
3. ฝึกใช้ Docker เพื่อสร้างสภาพแวดล้อมที่รันได้เหมือนกันทุกเครื่อง
4. เผยแพร่ผลงานบน GitHub พร้อมเอกสารอธิบายขั้นตอนอย่างครบถ้วน

---

## 2. โครงสร้างโฟลเดอร์ของโปรเจ็ค

ก่อนเริ่มต้น ให้สร้างโครงสร้างไฟล์ดังนี้
```
ml_service/
├─ app.py
├─ train_model.py
├─ requirements.txt
├─ Dockerfile
├─ templates/
│ └─ main.html
└─ model/
```
โฟลเดอร์ `model` จะถูกสร้างอัตโนมัติเมื่อฝึกและบันทึกโมเดล

---

## 3. ขั้นตอนการดำเนินงาน

### 3.1 สร้างโฟลเดอร์โครงการ

สร้างโฟลเดอร์สำหรับงานนี้ เช่น `D:\CPE312\ml_service`

เปิดโฟลเดอร์นี้ด้วยโปรแกรม VS Code

---

### 3.2 สร้างและฝึกโมเดล

ไฟล์: `train_model.py`

```python
from sklearn.datasets import fetch_california_housing
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestRegressor
import joblib, os

data = fetch_california_housing(as_frame=False)
X, y = data.data, data.target

Xtr, Xte, ytr, yte = train_test_split(X, y, test_size=0.2, random_state=42)

model = RandomForestRegressor(n_estimators=100, random_state=42)
model.fit(Xtr, ytr)

os.makedirs("model", exist_ok=True)
joblib.dump(model, "model/model.pkl")
print("บันทึกโมเดลเรียบร้อยที่ model/model.pkl")
```
---
### 3.3 สร้างแอปพลิเคชัน Flask

ไฟล์: `app.py`

```python
from flask import Flask, request, jsonify, render_template
import joblib, numpy as np, os

BASE_DIR = os.path.dirname(os.path.abspath(**file**))
MODEL_PATH = os.path.join(BASE_DIR, "model", "model.pkl")
model = joblib.load(MODEL_PATH)

FEATURES = ["MedInc","HouseAge","AveRooms","AveBedrms","Population","AveOccup","Latitude","Longitude"]

app = Flask(**name**, template_folder="templates")

@app.get("/health")
def health():
return {"ok": True, "features": FEATURES}

@app.post("/predict")
def predict():
data = request.get_json(silent=True) or {}
x = data.get("features")
if not x or len(x) != len(FEATURES):
return jsonify(error=f"Expected 8 features: {FEATURES}"), 400
X = np.array([x], dtype=float)
yhat = model.predict(X).tolist()[0]
return jsonify(prediction=yhat)

@app.route("/", methods=["GET", "POST"])
def main():
if request.method == "GET":
return render_template("main.html")
vals = [float(request.form.get(f)) for f in FEATURES]
pred = model.predict(np.array([vals])).tolist()[0]
return render_template("main.html", prediction=pred)

if **name** == "**main**":
app.run(debug=True)
```
---
### 3.4 สร้างหน้าเว็บสำหรับทดสอบ

ไฟล์: `templates/main.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>California Housing — ML Predictor</title>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <style>
    :root { font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif; }
    body { max-width: 920px; margin: 24px auto; padding: 0 16px; }
    h1 { margin-bottom: 4px; }
    .sub { color: #555; margin-top: 0; }
    .grid { display: grid; grid-template-columns: repeat(2, minmax(260px, 1fr)); gap: 14px 20px; }
    label { font-weight: 600; display:block; margin-bottom: 6px; }
    .help { font-size: 12px; color: #666; margin-top: 4px; }
    input[type="number"] { width: 100%; padding: 10px 12px; border: 1px solid #ccc; border-radius: 10px; }
    .actions { margin-top: 16px; display:flex; gap: 10px; flex-wrap: wrap; }
    button { padding: 10px 16px; border: 0; border-radius: 10px; cursor: pointer; }
    .primary { background: #2563eb; color: white; }
    .ghost { background: #f1f5f9; }
    .result { margin-top: 18px; padding: 14px; border-radius: 10px; background: #f8fafc; border: 1px solid #e2e8f0; }
    .err { color: #b91c1c; }
    .muted { color:#64748b; font-size:12px; }
    footer { margin-top: 24px; color:#64748b; font-size:12px; }
  </style>
</head>
<body>
  <h1>California Housing — ML Predictor</h1>
  <p class="sub">Enter values and click <b>Predict</b>. The model estimates median house value (in $100,000s).</p>

  <form id="form" class="grid" autocomplete="off">
    <div>
      <label for="MedInc">MedInc (Median income, $10k units)</label>
      <input id="MedInc" name="MedInc" type="number" step="0.1" min="0" max="25" placeholder="e.g., 8.3" required />
      <div class="help">Typical 0–15 (e.g., 8.3 means $83,000).</div>
    </div>

    <div>
      <label for="HouseAge">HouseAge (years)</label>
      <input id="HouseAge" name="HouseAge" type="number" step="1" min="1" max="100" placeholder="e.g., 25" required />
      <div class="help">Average age of houses in the block.</div>
    </div>

    <div>
      <label for="AveRooms">AveRooms (avg rooms per dwelling)</label>
      <input id="AveRooms" name="AveRooms" type="number" step="0.1" min="1" max="15" placeholder="e.g., 6.0" required />
      <div class="help">Total rooms / total households.</div>
    </div>

    <div>
      <label for="AveBedrms">AveBedrms (avg bedrooms per dwelling)</label>
      <input id="AveBedrms" name="AveBedrms" type="number" step="0.1" min="0.5" max="6" placeholder="e.g., 1.0" required />
      <div class="help">Total bedrooms / total households.</div>
    </div>

    <div>
      <label for="Population">Population (people)</label>
      <input id="Population" name="Population" type="number" step="1" min="1" max="50000" placeholder="e.g., 1200" required />
      <div class="help">Block group population.</div>
    </div>

    <div>
      <label for="AveOccup">AveOccup (avg occupants per household)</label>
      <input id="AveOccup" name="AveOccup" type="number" step="0.1" min="0.5" max="10" placeholder="e.g., 3.0" required />
      <div class="help">Population / households.</div>
    </div>

    <div>
      <label for="Latitude">Latitude</label>
      <input id="Latitude" name="Latitude" type="number" step="0.01" min="32" max="42" placeholder="e.g., 34.20" required />
      <div class="help">Approx. California 32–42.</div>
    </div>

    <div>
      <label for="Longitude">Longitude</label>
      <input id="Longitude" name="Longitude" type="number" step="0.01" min="-125" max="-114" placeholder="e.g., -118.30" required />
      <div class="help">Approx. California −125 to −114.</div>
    </div>

    <div class="actions">
      <button type="button" class="ghost" id="sample">Use sample values</button>
      <button type="submit" class="primary">Predict</button>
      <button type="reset" class="ghost">Reset</button>
    </div>
  </form>

  <div id="output" class="result" style="display:none;"></div>

  <footer>
    <div class="muted">
      Feature order sent to the API: [MedInc, HouseAge, AveRooms, AveBedrms, Population, AveOccup, Latitude, Longitude]
    </div>
  </footer>

  <script>
    const FEATURES = ["MedInc","HouseAge","AveRooms","AveBedrms","Population","AveOccup","Latitude","Longitude"];
    const sample = { MedInc: 8.3, HouseAge: 25, AveRooms: 6.0, AveBedrms: 1.0, Population: 1200, AveOccup: 3.0, Latitude: 34.2, Longitude: -118.3 };

    document.getElementById('sample').addEventListener('click', () => {
      for (const [k,v] of Object.entries(sample)) document.getElementById(k).value = v;
    });

    document.getElementById('form').addEventListener('submit', async (e) => {
      e.preventDefault();
      const vals = [];
      try {
        for (const f of FEATURES) {
          const el = document.getElementById(f);
          if (!el || el.value === "") throw new Error(`Missing value: ${f}`);
          vals.push(parseFloat(el.value));
        }
        const res = await fetch("/predict", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ features: vals })
        });
        const data = await res.json();
        const out = document.getElementById('output');
        out.style.display = 'block';
        if (!res.ok) {
          out.innerHTML = `<div class="err"><b>Error:</b> ${data.error || 'Request failed'}</div>`;
          return;
        }
        const y = Number(data.prediction);
        out.innerHTML = `<b>Prediction:</b> ${isFinite(y) ? y.toFixed(3) : data.prediction}
          <div class="muted">Inputs: [${vals.join(', ')}]</div>`;
      } catch (err) {
        const out = document.getElementById('output');
        out.style.display = 'block';
        out.innerHTML = `<div class="err"><b>Validation error:</b> ${err.message}</div>`;
      }
    });
  </script>
</body>
</html>
```
---
### 3.5 สร้างไฟล์ requirements.txt

```
flask
gunicorn
numpy
scikit-learn
joblib
pandas
```
---
### 3.6 สร้างไฟล์ Dockerfile

```
FROM python:3.11-slim
WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .
RUN python train_model.py

EXPOSE 5000
CMD ["gunicorn", "-w", "2", "-b", "0.0.0.0:5000", "app:app"]
```
---
### 3.7 สร้างและรัน Docker Container

เปิด PowerShell แล้วรันคำสั่ง
```
cd D:\CPE312\ml_service
docker build -t ds-ml-service .
docker run --rm -p 5000:5000 ds-ml-service
```

เปิดเบราว์เซอร์และเข้าตรวจสอบที่ web browser

หน้าเว็บ: `http://127.0.0.1:5000/`

ตรวจสอบสถานะ: `http://127.0.0.1:5000/health`

---
## 4. การตรวจสอบความถูกต้อง

- สร้างและรัน Docker image ได้สำเร็จ
- หน้าเว็บแสดงแบบฟอร์มรับค่าฟีเจอร์และผลการทำนายได้
- เรียกใช้งาน API /predict ด้วย JSON ได้ เช่น
 `{"features":[8.3,25.0,6.0,1.0,1200,3.0,34.2,-118.3]}`

ผลลัพธ์ที่ได้จะอยู่ในรูปแบบ
`{"prediction": 2.38}`

---
## 5. วิธีส่งงาน

- Push โค้ดทั้งหมด (รวมถึง Dockerfile และ README.md) ไปยัง GitHub
- ในไฟล์ README.md ให้ระบุวิธีการ build และ run
- แนบภาพหน้าจอของหน้าเว็บที่ทำนายผลสำเร็จ
- ส่งลิงก์ GitHub repository ผ่านระบบตามที่อาจารย์กำหนด

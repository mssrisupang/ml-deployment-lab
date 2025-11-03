# การทดลองที่ 5: การสร้าง Model as a Service ด้วย Flask และ Docker

## 1. วัตถุประสงค์

1. ทำความเข้าใจแนวคิดของการให้บริการโมเดล Machine Learning ในรูปแบบ Model as a Service
2. ฝึกสร้างเว็บแอปพลิเคชันโดยใช้ Flask เพื่อรับข้อมูลและแสดงผลการทำนาย
3. ฝึกใช้ Docker เพื่อสร้างสภาพแวดล้อมที่รันได้เหมือนกันทุกเครื่อง
4. เผยแพร่ผลงานบน GitHub พร้อมเอกสารอธิบายขั้นตอนอย่างครบถ้วน

---

## 2. โครงสร้างโฟลเดอร์ของโครงการ

ก่อนเริ่มต้น ให้สร้างโครงสร้างไฟล์ดังนี้

ml_service/
├─ app.py
├─ train_model.py
├─ requirements.txt
├─ Dockerfile
├─ templates/
│ └─ main.html
└─ model/

โฟลเดอร์ `model` จะถูกสร้างอัตโนมัติเมื่อฝึกและบันทึกโมเดล

---

## 3. ขั้นตอนการดำเนินงาน

### 3.1 สร้างโฟลเดอร์โครงการ

สร้างโฟลเดอร์สำหรับงานนี้ เช่น D:\CPE312\ml_service

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

### 3.3 สร้างแอปพลิเคชัน Flask

ไฟล์: app.py
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

### 3.4 สร้างหน้าเว็บสำหรับทดสอบ

ไฟล์: templates/main.html

<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="utf-8" />
  <title>California Housing Predictor</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    body { font-family: sans-serif; max-width: 900px; margin: 24px auto; padding: 0 16px; }
    .grid { display: grid; grid-template-columns: repeat(2, minmax(260px,1fr)); gap: 12px 20px; }
    label { font-weight: 600; display:block; margin-bottom: 6px; }
    input[type="number"] { width: 100%; padding: 10px 12px; border: 1px solid #ccc; border-radius: 8px; }
    button { padding: 10px 16px; border-radius: 8px; border: 0; cursor: pointer; }
    .primary { background: #2563eb; color: white; }
    .ghost { background: #f1f5f9; }
    .actions { margin-top: 12px; display:flex; gap: 10px; flex-wrap: wrap; }
    .result { margin-top: 16px; padding: 12px; border:1px solid #e2e8f0; border-radius:8px; }
  </style>
</head>
<body>
  <h1>California Housing Prediction</h1>
  <form method="post" class="grid">
    <div><label>MedInc<input name="MedInc" type="number" step="0.1" required></label></div>
    <div><label>HouseAge<input name="HouseAge" type="number" step="1" required></label></div>
    <div><label>AveRooms<input name="AveRooms" type="number" step="0.1" required></label></div>
    <div><label>AveBedrms<input name="AveBedrms" type="number" step="0.1" required></label></div>
    <div><label>Population<input name="Population" type="number" step="1" required></label></div>
    <div><label>AveOccup<input name="AveOccup" type="number" step="0.1" required></label></div>
    <div><label>Latitude<input name="Latitude" type="number" step="0.01" required></label></div>
    <div><label>Longitude<input name="Longitude" type="number" step="0.01" required></label></div>
    <div class="actions">
      <button type="submit" class="primary">Predict</button>
      <button type="reset" class="ghost">Reset</button>
    </div>
  </form>

{% if prediction is defined %}

<div class="result"><strong>Prediction:</strong> {{ prediction }}</div>
{% endif %}

</body>
</html>

### 3.5 สร้างไฟล์ requirements.txt

flask
gunicorn
numpy
scikit-learn
joblib
pandas

### 3.6 สร้างไฟล์ Dockerfile

FROM python:3.11-slim
WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .
RUN python train_model.py

EXPOSE 5000
CMD ["gunicorn", "-w", "2", "-b", "0.0.0.0:5000", "app:app"]

### 3.7 สร้างและรัน Docker Container

เปิด PowerShell แล้วรันคำสั่ง
cd D:\CPE312\ml_service
docker build -t ds-ml-service .
docker run --rm -p 5000:5000 ds-ml-service

เปิดเบราว์เซอร์และเข้าชมที่

หน้าเว็บ: http://127.0.0.1:5000/

ตรวจสอบสถานะ: http://127.0.0.1:5000/health

## 4. การตรวจสอบความถูกต้อง

- สร้างและรัน Docker image ได้สำเร็จ
- หน้าเว็บแสดงแบบฟอร์มรับค่าฟีเจอร์และผลการทำนายได้
- เรียกใช้งาน API /predict ด้วย JSON ได้ เช่น
  {"features":[8.3,25.0,6.0,1.0,1200,3.0,34.2,-118.3]}

ผลลัพธ์ที่ได้จะอยู่ในรูปแบบ
{"prediction": 2.38}

## 5. วิธีส่งงาน

- Push โค้ดทั้งหมด (รวมถึง Dockerfile และ README.md) ไปยัง GitHub
- ในไฟล์ README.md ให้ระบุวิธีการ build และ run
- แนบภาพหน้าจอของหน้าเว็บที่ทำนายผลสำเร็จ
- ส่งลิงก์ GitHub repository ผ่านระบบตามที่อาจารย์กำหนด

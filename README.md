# Telco Customer Churn Prediction

ทำนายว่าลูกค้าบริษัทโทรคมนาคมจะยกเลิกบริการ (churn) หรือไม่ โดยเปรียบเทียบ Logistic Regression กับ Decision Tree แล้วดูว่า feature ไหนมีผลมากที่สุด

## Dataset

- **ที่มา:** Telco Customer Churn (IBM Sample Dataset)
- **ขนาด:** 7,032 ลูกค้า, 24 features
- **Churn rate:** 26.6% (ข้อมูล imbalanced — ลูกค้าที่ไม่ churn เยอะกว่ามาก)

## เครื่องมือที่ใช้

- Python (scikit-learn, pandas, matplotlib)
- Google Colab

## Data Preprocessing

ข้อมูลดิบเป็น text ทั้งหมด (Yes/No, Male/Female, DSL/Fiber optic) ต้องแปลงเป็นตัวเลขก่อน train model

**สิ่งที่ทำ:**
- ลบ `customerID` ออก — ไม่มีผลต่อการทำนาย
- ลบ 11 rows ที่ `TotalCharges` เป็นค่าว่าง (ลูกค้าที่เพิ่ง signup ยังไม่มียอดเรียกเก็บ)
- columns ที่มี 2 ค่า (Yes/No, Male/Female) → แปลงเป็น 0/1
- `Contract` → Label Encoding (Month-to-month=0, One year=1, Two year=2) เรียงตามความยาวสัญญา
- `InternetService` → One-Hot Encoding แตกเป็น Int_DSL, Int_Fiber, Int_No เพราะมี 3 ค่าที่ไม่มีลำดับ
- `PaymentMethod` → One-Hot Encoding แตกเป็น Pay_ElecCheck, Pay_MailCheck, Pay_BankTransfer, Pay_CreditCard

7,043 rows → 7,032 rows (24 features + 1 target)

## Model Comparison

เปรียบเทียบ 2 models:
- **Logistic Regression** — scale ด้วย StandardScaler ก่อน train
- **Decision Tree** — max_depth=5 เพื่อป้องกัน overfitting

![Model Comparison](chart/chart_model_comparison.png)

| Metric | Logistic Regression | Decision Tree |
|--------|-------------------|---------------|
| Accuracy | 79.5% | 79.8% |
| Precision | 62.5% | 60.8% |
| Recall | 53.3% | 63.1% |
| F1-Score | 57.5% | 61.9% |

## Confusion Matrix

![Confusion Matrix](chart/chart_confusion_matrix.png)

- **Logistic Regression** ทายพลาด 171 คนที่ churn จริงแต่ model บอกว่าไม่ churn (46.7% ของ churner ทั้งหมด)
- **Decision Tree** ทายพลาดน้อยกว่า 135 คน (36.9%) แปลว่าจับลูกค้าที่กำลังจะหายได้ดีกว่า

## Feature Importance

![Feature Importance](chart/chart_feature_importance.png)

3 features ที่มีผลมากที่สุด:
1. **Contract** (0.516) — ลูกค้าที่ไม่มีสัญญาระยะยาว churn เยอะกว่ามาก
2. **tenure** (0.168) — ลูกค้าใหม่ (tenure น้อย) มีแนวโน้ม churn สูง
3. **Int_Fiber** (0.156) — ลูกค้าที่ใช้ Fiber optic internet churn มากกว่า DSL

## Business Insights

**1. Decision Tree เหมาะกว่าสำหรับ churn prediction**

Accuracy ใกล้กัน (~80%) แต่ Recall ของ DT สูงกว่า (63.1% vs 53.3%) ในบริบทของ churn สิ่งที่สำคัญคือ "จับลูกค้าที่กำลังจะหายไปให้ได้" ไม่ใช่แค่ทายถูกเยอะ เพราะต้นทุนการสูญเสียลูกค้าสูงกว่าต้นทุนการส่งโปรโมชั่นผิดคน

**2. ควรเน้นลูกค้าที่ไม่มี contract**

Contract เป็น feature สำคัญที่สุด ลูกค้า month-to-month ออกง่ายกว่าลูกค้าที่ผูกสัญญา 1-2 ปี → ควรมี incentive ให้ลูกค้าเปลี่ยนจาก month-to-month เป็นสัญญาระยะยาว

**3. ลูกค้าใหม่ + Fiber optic เป็นกลุ่มเสี่ยง**

ลูกค้าที่ tenure น้อย + ใช้ Fiber optic มีแนวโน้ม churn สูง อาจเป็นเพราะราคาสูงกว่า DSL แต่ไม่ได้รู้สึกว่าคุ้ม → ควรดูว่า service quality ของ Fiber ตรงกับความคาดหวังหรือไม่


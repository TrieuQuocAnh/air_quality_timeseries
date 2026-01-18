# Cấu Hình Tham Số - Air Quality Timeseries Project

## 📋 Tóm Tắt Cấu Hình Hiện Tại

Tài liệu này ghi rõ các tham số chính đã dùng trong pipeline chạy cuối cùng.

---

## 🎯 Tham Số Chính

### 1. **CUTOFF** (Ngưỡng Chia Train/Test)
- **Giá trị**: `2017-01-01`
- **Ý nghĩa**: Dữ liệu trước ngày này dùng cho huấn luyện, sau dùng cho kiểm tra
- **Áp dụng cho**: Classification, Regression, ARIMA forecasting

### 2. **LAG_HOURS** (Độ Trễ Thời Gian)
- **Giá trị**: `[1, 3, 24]` (giờ)
- **Ý nghĩa**: 
  - 1 giờ trước
  - 3 giờ trước
  - 24 giờ trước (1 ngày)
- **Áp dụng cho**: Preprocessing, Feature Preparation, Regression
- **Ghi chú**: Dùng để tạo features từ giá trị lịch sử

### 3. **HORIZON** (Khoảng Dự Báo)
- **Giá trị**: `1` (giờ)
- **Ý nghĩa**: Dự báo 1 giờ tiếp theo
- **Áp dụng cho**: Regression modeling
- **Ghi chú**: Dự báo ngắn hạn

### 4. **STATION** (Trạm Không Khí)
- **Giá trị**: `Aotizhongxin` (奥体中心)
- **Ý nghĩa**: Trạm đo không khí tại Trung tâm Thể thao Aoti, Bắc Kinh
- **Áp dụng cho**: ARIMA forecasting
- **Ghi chú**: Chỉ một trạm được dùng cho ARIMA

### 5. **Tham Số ARIMA**
- **P_MAX**: `3` (Maximum AR order)
- **D_MAX**: `2` (Maximum differencing order)
- **Q_MAX**: `3` (Maximum MA order)
- **IC (Information Criterion)**: `aic` (AIC thay vì BIC)
- **Grid Search Range**: (p, d, q) ∈ [0..3] × [0..2] × [0..3]

---

## 📊 Cấu Trúc Dữ Liệu

### Input
- **Raw Data**: `data/raw/PRSA2017_Data_20130301-20170228.zip`
- **Dữ liệu**: 2013-03-01 đến 2017-02-28 (khoảng 4 năm)

### Output
- **Cleaned Data**: `data/processed/cleaned.parquet`
- **Dataset (Classification)**: `data/processed/dataset_for_clf.parquet`
- **Dataset (Regression)**: `data/processed/dataset_for_regression.parquet`
- **ARIMA Summary**: `data/processed/arima_pm25_summary.json`
- **ARIMA Predictions**: `data/processed/arima_pm25_predictions.csv`
- **Regression Model**: `data/processed/regressor.joblib`
- **Regression Metrics**: `data/processed/regression_metrics.json`

---

## 🔧 Notebooks Chính

| Notebook | Tham Số Dùng | Mục Đích |
|----------|------------|---------|
| `preprocessing_and_eda.ipynb` | `LAG_HOURS` | Tiền xử lý, EDA |
| `feature_preparation.ipynb` | `CUTOFF` | Chuẩn bị features |
| `classification_modelling.ipynb` | `CUTOFF` | Phân loại chất lượng không khí |
| `regression_modelling.ipynb` | `LAG_HOURS`, `HORIZON`, `CUTOFF` | Dự báo theo lag |
| `arima_forecasting.ipynb` | `STATION`, `CUTOFF`, `P_MAX`, `D_MAX`, `Q_MAX` | ARIMA forecasting |

---

## 🎛️ File Cấu Hình Chính

- **`run_papermill.py`**: File chạy pipeline tự động, chứa tất cả tham số
- **Kernel**: `beijing_env` (Python environment)

---

## ⏱️ Timeline Train/Test

```
Dữ liệu: ──────────────── CUTOFF (2017-01-01) ────────────→
         Train Data      Test Data
         (2013-03 ~ 2016-12)   (2017-01 ~ 2017-02)
```

---

## 📌 Ghi Chú Quan Trọng

1. **LAG_HOURS** chỉ dùng cho **Regression** và **Preprocessing**
2. **HORIZON=1** có nghĩa dự báo 1 giờ tiếp theo (PM2.5 tại t+1)
3. **ARIMA chỉ dùng một trạm** (Aotizhongxin) cho mô hình đơn biến
4. **CUTOFF** thống nhất trên tất cả models để so sánh công bằng
5. **IC='aic'** dùng Akaike Information Criterion để chọn (p,d,q)

---

*Cập nhật lần cuối: 2026-01-18*

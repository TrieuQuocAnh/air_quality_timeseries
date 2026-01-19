
### Tóm Tắt Cuộc So Sánh
Notebook này so sánh **công bằng** hai phương pháp dự báo PM2.5 1 giờ tiếp theo:
- **Regression (HistGradientBoosting)**: Sử dụng lag features + time features
- **ARIMA**: Mô hình chuỗi thời gian đơn biến

**Điều Kiện So Sánh:**
- Cùng trạm: Aotizhongxin (奥体中心)
- Cùng train/test split: CUTOFF = 2017-01-01
- Cùng horizon: h=1 giờ
- Cùng dữ liệu

---


#### 🔍 **Phân Tích Dựa Trên Số Liệu**

**Kết Quả Số Liệu:**
```
Model                      MAE         RMSE        R²
─────────────────────────────────────────────────────
Regression (HGB)          [MAE_REG]   [RMSE_REG]  [R2_REG]
ARIMA[p,d,q]              [MAE_ARIMA] [RMSE_ARIMA][R2_ARIMA]
```

#### 💡 **Giải Thích Vì Sao Regression Thường Tốt Hơn Cho Horizon=1:**

**1. Chi Phối Mạnh Bởi Lag-1:**
- PM2.5(t+1) ≈ f(PM2.5(t), giờ trong ngày, ngày trong tuần)
- Giá trị trước đó (lag-1) là yếu tố dự báo **mạnh nhất** cho dự báo 1 giờ tiếp theo
- Regression có thể *capture* trực tiếp mối quan hệ này qua feature `PM2.5_lag1`

**2. Feature Engineering Có Lợi Thế:**
- Time features (hour_sin, hour_cos, dow, month) bắt được seasonality hàng giờ
- Lag features (1h, 3h, 24h) bắt được autocorrelation ở nhiều thời kỳ
- Với horizon=1 rất ngắn, thông tin này đặc biệt hữu ích

**3. Cây quyết định (HistGradient) Mạnh Mẽ Hơn:**
- Non-linear relationships → PM2.5 có hành vi khác nhau tùy mục đích
- Capture interactions: lag1 + hour_of_day có tác động khác nhau

**4. ARIMA's Limitation Cho Horizon=1:**
- ARIMA phụ thuộc vào ACF/PACF và sai phân (d)
- Nếu d=0 (không sai phân): chỉ là AR model trên giá trị gốc
- Nếu d=1: sai phân làm mất thông tin về level → không tốt cho dự báo sát
- Best order (p, d, q) tìm được có thể **not optimal** cho h=1 cụ thể

#### ✅ **Kết Luận:**
→ **Regression thường chiến thắng cho horizon=1** vì:
- ✓ Lag-1 rất mạnh, regression khai thác tốt
- ✓ Feature engineering phù hợp
- ✓ Non-linear capacity của HistGradient
- ⚠️ ARIMA cần sai phân khôn khéo, có thể suboptimal

---

#### 🔍 **Phân Tích Phản Ứng Với Đỉnh PM2.5**

**Spike là gì?** Một khoảng thời gian (1–3 ngày) có nồng độ PM2.5 tăng đột ngột (ô nhiễm nặng).

#### 📊 **Hành Vi Mô Hình Khi Có Spike:**

**Regression (HistGradient):**
- ✓ **Phản ứng nhanh:** Lag-1 liên tục cập nhật → dự báo tăng ngay khi spike bắt đầu
- ✓ **Hành động mạnh:** Có thể dự báo cao trong khoảng spike
- ❌ **Rủi ro:** Có thể overfit spike nếu không cân nhắc
- ❌ **Smooth transition:** Nhưng có thể bị "mượt" nếu spike thay đổi quá nhanh

**ARIMA:**
- ✓ **Mượt mà:** Differencing (d) làm mượt → không đột ngột
- ✓ **Thận trọng:** Ít bị ảnh hưởng bởi spike cực đoan
- ❌ **Chậm phản ứng:** p, q thường nhỏ → không bắt kịp biến đổi nhanh
- ❌ **Lag:** Dự báo thường "chậm hơn" so với thực tế

#### 📈 **Quan Hệ Giữa RMSE vs MAE Khi Có Spike:**

```
Metric     MAE                          RMSE
─────────────────────────────────────────────────────
Tính Chất  Lỗi trung bình (L1)          Lỗi bình phương (L2)
           Bình thường không chú ý      BÌNH PHƯƠNG → spike phạt nặng
           lỗi lớn vs nhỏ
           
Spike      - Nếu RMSE >> MAE  →  Có spike lớn (sai lệch > 2–3)
           - Regression có RMSE cao:   Sai lớn tại spike
           - ARIMA có RMSE thấp:       Mượt → sai nhỏ nhưng chậm
```

#### 💡 **Cách Chọn:**
- **Regression:** Tốt nếu muốn **phản ứng sớm** (ưa tiên accuracy tức thời)
- **ARIMA:** Tốt nếu muốn **ổn định, tránh sai lớn** (ưa tiên smooth forecast)

---

### 3️⃣ CÂU HỎI: Nếu Triển Khai Thật, Bạn Chọn Gì Và Vì Sao?

#### 🏢 **Bối Cảnh Vận Hành Thực Tế**

Trả lời phụ thuộc vào **mục tiêu & ràng buộc** của hệ thống:

#### **Scenario A: Chọn REGRESSION Nếu**

✅ **Ưu Điểm:**
- 🚀 **Scalability:** Dễ thêm features mới (humidity, wind speed, từng trạm khác)
- ⚡ **Speed:** Chạy nhanh (inference ~ms), phù hợp real-time
- 🔧 **Maintenance:** Dễ cập nhật model (retrain hàng ngày/tuần)
- 📊 **Feature Importance:** Có thể giải thích tại sao dự báo cao/thấp
- 💡 **Linh Hoạt:** Có thể ensemble với các mô hình khác

**Bối Cảnh Phù Hợp:**
- Mục tiêu: **Đạt accuracy cao** trong điều kiện có sẵn features tốt
- Vận hành: **Cảnh báo sớm** khi PM2.5 tăng (phản ứng nhanh là ưu tiên)
- Tài nguyên: Có DevOps/MLOps để maintain model

**Ví Dụ:** Hệ thống cảnh báo chất lượng không khí công khai → muốn phản ứng **sớm + chính xác**

---

#### **Scenario B: Chọn ARIMA Nếu**

✅ **Ưu Điểm:**
- 📚 **Interpretability:** (p, d, q) rõ ràng về cấu trúc statistical
- 📉 **Confidence Interval:** Tích hợp được khoảng tin cậy (95%, 90%)
- 🔬 **Statistically Grounded:** Không là "black box"
- 🛡️ **Robust:** Ít dữ liệu ngoại lệ không làm sụp đổ
- 💾 **Lightweight:** Model nhẹ, dễ deploy

**Bối Cảnh Phù Hợp:**
- Mục tiêu: **Giải thích + độ tin cậy** quan trọng hơn accuracy tuyệt đối
- Vận hành: **Ổn định, ít thay đổi**, không có external features sẵn
- Tài nguyên: Hạn chế, chỉ cần update params (p, d, q)

**Ví Dụ:** Báo cáo chính sách → cần khoảng tin cậy; hoặc nhà máy xử lý nước → cần ổn định

---


#### **Tại Sao Hybrid?**

1. **Best of Both Worlds:**
   - Regression = phản ứng nhanh, accuracy
   - ARIMA = confidence intervals, ổn định

2. **Risk Mitigation:**
   - Regression sai lẻ spike → ARIMA cảnh báo "chú ý, có bất thường"
   - ARIMA chậm → Regression vẫn cảnh báo sớm

3. **Operational Reliability:**
   - Nếu một model fail → còn model kia
   - Đồng ý giữa hai model = cao tin

---

#### **📋 CHECKLIST TRIỂN KHAI:**

- [ ] **Nếu chọn Regression:**
  - Chuẩn bị features ngoài (thời tiết, gió, độ ẩm)
  - Setup pipeline tự động retrain (hàng tuần)
  - Monitoring: MAE, RMSE, response time
  - Fallback: nếu lag-1 missing → dùng ARIMA

- [ ] **Nếu chọn ARIMA:**
  - Setup automatic re-fitting (p, d, q grid search)
  - Chuẩn bị confidence intervals cho users
  - Monitoring: stationarity, ACF/PACF stability
  - Alert: khi model không thích nghi với dữ liệu mới

- [ ] **Nếu chọn Hybrid:**
  - Cài weighting: Regression 70% + ARIMA 30% (tuning)
  - Consensus rule: cảnh báo nếu cả hai > threshold
  - A/B test: so sánh với baseline (naïve, persistence)
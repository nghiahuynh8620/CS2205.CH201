# AI-SOC / UEBA: Phát hiện hành vi bất thường người dùng trong Enterprise Network

> Đồ án/Đề tài: Ứng dụng khai phá dữ liệu & Machine Learning để phát hiện bất thường hành vi người dùng (UEBA) và hỗ trợ SOC (AI-SOC).  
> Môn/HP: CS2205 (SEP2025)  
> Nhóm: [Team Name] — GVHD: [Name]  
> Demo: [YouTube/Drive Link] • Repo: [GitHub Link]

---

## 1. Tổng quan
UEBA (User & Entity Behavior Analytics) giúp phát hiện các hành vi bất thường của người dùng/thực thể (user/host/service) dựa trên log hoạt động trong hệ thống doanh nghiệp.  
Dự án này xây dựng pipeline từ **thu thập log → chuẩn hoá/ETL → trích đặc trưng → mô hình ML/DL → chấm điểm bất thường → cảnh báo & drill-down** phục vụ SOC.

### Mục tiêu
- Chuẩn hoá dữ liệu log thành bộ dữ liệu học máy cho UEBA.
- Xây dựng mô hình phát hiện bất thường (baseline + mô hình chính).
- Đánh giá bằng các chỉ số phù hợp và cung cấp cơ chế giải thích/cung cấp bằng chứng (evidence).
- Demo dashboard/CLI để xem alerts và điều tra theo user/timeline.

---

## 2. Bài toán & phạm vi
### Bài toán
Từ log hệ thống (Windows/AD/VPN/Proxy/Firewall/EDR/… tùy nguồn), phát hiện:
- **User anomaly**: hành vi đăng nhập bất thường, truy cập tài nguyên lạ, hoạt động ngoài giờ, thay đổi đặc quyền,…
- **Entity/Host anomaly**: host phát sinh hành vi lạ, kết nối bất thường, spike sự kiện,…

### Phạm vi dữ liệu (tùy chọn theo khả năng)
- Batch logs (CSV/JSON) hoặc streaming giả lập theo time window.
- Dữ liệu có nhãn (semi-supervised) hoặc không nhãn (unsupervised).

---

## 3. Kiến trúc & Pipeline
### 3.1 Kiến trúc tổng thể (high-level)
Nguồn log → (SIEM/Log store) → ETL/Parsing → Feature Store → UEBA Model → Anomaly Score → Alert/Case → SOC Analyst

### 3.2 Pipeline xử lý (tóm tắt)
1) Ingest & Parsing  
2) Chuẩn hoá schema (user/host/action/resource/time/metadata)  
3) Window aggregation (5m/1h/1d)  
4) Feature engineering (statistical + sequence + peer/group + graph [optional])  
5) Train/Validate (offline)  
6) Inference & Alerting (online/batch)  
7) Dashboard/Report

> Sơ đồ pipeline chi tiết được mô tả trong slide/poster của nhóm.

---

## 4. Dataset
> Bạn cập nhật đúng dataset bạn dùng (ưu tiên công khai/hợp pháp).

### Nguồn dữ liệu
- [ ] Public dataset: [Tên + link]
- [ ] Synthetic logs: tạo dữ liệu giả lập theo kịch bản (normal vs anomaly)
- [ ] Logs nội bộ (nếu có): **không public**, chỉ mô tả thống kê/ẩn danh

### Định dạng chuẩn (gợi ý)
- `timestamp`, `user`, `host`, `src_ip`, `dst_ip`, `event_type`, `resource`, `status`, `count`, `extra_json`

---

## 5. Phương pháp
### 5.1 Tiền xử lý & ETL
- Làm sạch, loại trùng, xử lý missing
- Chuẩn hoá timestamp/timezone
- Mapping log → schema chuẩn
- Gom theo time window và tạo bảng “user-day” / “user-hour” / “user-session”

### 5.2 Trích đặc trưng (feature engineering)
Gợi ý nhóm feature:
- **Statistical**: số lần login, thất bại/thành công, tỉ lệ fail, số IP khác nhau, số host truy cập, entropy resource,…
- **Temporal**: hoạt động ngoài giờ, burst/spike, rolling mean/std,…
- **Peer-group**: so với nhóm (phòng ban/role), z-score theo cohort
- **Sequence** (optional): chuỗi sự kiện, n-gram, session patterns
- **Graph** (optional): user–resource graph, centrality, community deviation

### 5.3 Mô hình
- Baselines: Isolation Forest / One-Class SVM / LOF
- Main model (đề xuất): Autoencoder / LSTM-AE / XGBoost (nếu có nhãn) / Hybrid scoring
- Alerting: threshold theo quantile + rule guardrail (giảm false positive)

### 5.4 Giải thích (Explainability)
- Feature importance (SHAP) cho mô hình supervised
- Reconstruction error breakdown cho Autoencoder
- Evidence view: top events/hosts/resources đóng góp vào score

---

## 6. Đánh giá
### Chiến lược chia dữ liệu
- **Time-based split** (khuyến nghị): train trước → test sau (tránh leakage)

### Chỉ số
- Nếu có nhãn: PR-AUC, ROC-AUC, F1, Recall@k, FPR
- Nếu không nhãn: anomaly hit-rate theo kịch bản, stability theo thời gian, tỉ lệ alert/day, latency

### Báo cáo kết quả
- Confusion matrix (nếu có nhãn)
- Bảng so sánh baseline vs mô hình chính
- Biểu đồ score theo thời gian + case studies (top alerts)

---

## 7. Demo (UI/CLI)
Tính năng demo tối thiểu:
- Danh sách alerts theo thời gian + filter theo user/host/event_type
- Drill-down 1 alert: timeline sự kiện, top evidence, score
- Export report (CSV/PDF) [optional]

> Link demo: [YouTube/Drive Link]

---

## 8. Cấu trúc thư mục (gợi ý)
```text
.
├─ data/
│  ├─ raw/                 # log thô (không commit nếu nhạy cảm)
│  ├─ processed/           # dữ liệu sau ETL
│  └─ sample/              # sample nhỏ để chạy thử
├─ notebooks/              # EDA, thử nghiệm
├─ src/
│  ├─ ingest/              # parser, loader
│  ├─ etl/                 # normalize schema, window aggregation
│  ├─ features/            # feature engineering
│  ├─ models/              # train/infer
│  ├─ evaluation/          # metrics, reports
│  └─ app/                 # demo UI/CLI
├─ assets/                 # hình pipeline, poster
├─ configs/                # yaml/json configs
├─ outputs/                # model artifacts, figures, reports
├─ requirements.txt
└─ README.md

# ✨ Databricks-End-to-End-Flight-Data-Engineering-Project (Medallion Architecture)
Dự án này mô phỏng pipeline xử lý dữ liệu thực tế theo kiến trúc **Medallion** (Raw → Bronze → Silver → Gold) sử dụng **Databricks Free Edition**(trước 15/07/2025). Mục tiêu là xây dựng hệ thống **tự động, mở rộng và phù hợp với môi trường sản xuất**, bao gồm ingest dữ liệu theo thời gian thực, xử lý SCD, tạo Star Schema, và tích hợp với **DBT**.
---

## 🎯 Mục tiêu học được

- Hiểu và triển khai kiến trúc **Medallion Architecture**
- Làm việc với **PySpark Structured Streaming** và **Databricks Autoloader**
- Xử lý dữ liệu theo kiểu **Incremental ingestion** (dữ liệu mới mỗi ngày)
- Quản lý schema với **Schema Evolution & Rescue Data**
- Tự động hóa xây dựng **Star Schema** (Fact Table + Slowly Changing Dimensions)
- Quản lý mô hình dữ liệu với **DBT**
- Tạo pipeline động sử dụng **Widgets & Workflows**

---
## 🏗️ Kiến trúc tổng thể

Dự án được thiết kế theo kiến trúc **Medallion** gồm 4 lớp: Raw → Bronze → Silver → Gold. Dữ liệu được ingest từ các file CSV mới mỗi ngày và xử lý qua từng lớp như sau:

1. **Raw Layer**: Dữ liệu được tải lên từ các file `.csv` và lưu trữ trong Databricks Volume.
2. **Bronze Layer**: Dữ liệu thô được đọc bằng Autoloader (Spark Structured Streaming) và lưu dưới định dạng Delta.
3. **Silver Layer**: Làm sạch dữ liệu, chuẩn hóa schema, xử lý lỗi và chuẩn bị cho phân tích.
4. **Gold Layer**: Xây dựng mô hình dữ liệu dạng Star Schema (Fact và Dimension), hỗ trợ báo cáo và phân tích.

Dữ liệu sau cùng được sử dụng để tạo mô hình DBT và phục vụ công cụ BI như Power BI, Tableau, v.v.

---
## 🔁 Giải thích các tầng xử lý dữ liệu

### 🥉 Bronze Layer
- Đọc dữ liệu dạng **streaming** từ volume bằng **Databricks Autoloader**.
- Lưu dữ liệu dạng **Delta Table** (append-only).
- Thiết lập **checkpoint** và **schema location** để đảm bảo dữ liệu không bị xử lý trùng (idempotent).

### 🥈 Silver Layer
- Làm sạch dữ liệu: xoá dòng lỗi, chuẩn hóa kiểu dữ liệu, xử lý nulls, duplicates.
- Sử dụng **LakeFlow** (trước đây là Delta Live Tables).
- Đưa dữ liệu về trạng thái phân tích được.

### 🥇 Gold Layer
- Xây dựng **Star Schema** gồm:
  - **Fact Table**: chứa thông tin booking
  - **Dimension Tables**: passenger, flight, airport
- Tự động xử lý **Slowly Changing Dimensions (SCD Type 2)**.
- Kết hợp với **DBT** để tạo mô hình phân tích cuối.
---
## 🧾 Dataset

Dữ liệu liên quan đến hành khách và đặt vé máy bay:

| File                 | Mô tả                              |
| -------------------- | ---------------------------------- |
| `fact_bookings.csv`  | Bảng fact chính: giao dịch booking |
| `dim_passengers.csv` | Thông tin hành khách               |
| `dim_flights.csv`    | Thông tin chuyến bay               |
| `dim_airports.csv`   | Thông tin sân bay                  |

📌 Dữ liệu được chia nhỏ theo từng ngày (incremental files) → giúp luyện tập ingest & streaming pipeline.

---
## 🛠️ Công nghệ sử dụng
Databricks Free Edition (Unity Catalog, Serverless Cluster)

PySpark Structured Streaming

Databricks Autoloader

Schema Evolution + Checkpointing

DBUtils Widgets + Control Flow

Delta Lake (storage format)

DBT for Data Modeling

LakeFlow (tên mới của Delta Live Tables)

--- 
## 👨‍💻 Cách triển khai dự án

```bash
1. Tạo tài khoản tại https://databricks.com/try-databricks
2. Upload dữ liệu vào volume: `raw/raw_data/<table_name>`
3. Chạy notebook `setup.py` để tạo volume/schema
4. Chạy `bronze_layer.py` để ingest dữ liệu
5. Chạy `silver_layer.py` để xử lý & chuẩn hóa dữ liệu
6. Chạy `gold_layer.py` để tạo star schema
7. Dùng DBT để build các mô hình cuối
```

---
## 🎯 Kết quả của dự án

Dự án triển khai thành công pipeline xử lý dữ liệu đặt vé máy bay theo kiến trúc **Medallion (Bronze – Silver – Gold)** trên nền tảng Databricks. Kết quả cụ thể như sau:

---

### ✅ Dữ liệu được ingest & xử lý qua nhiều lớp

- Hệ thống ingest dữ liệu từ các file `.csv` chứa thông tin về:
  - Chuyến bay (`dim_flights.csv`)
  - Sân bay (`dim_airports.csv`)
  - Hành khách (`dim_passengers.csv`)
  - Giao dịch đặt vé (`fact_bookings.csv`)
- Dữ liệu được ingest tự động bằng **Autoloader (Streaming)** từ volume, hỗ trợ incremental updates theo ngày.

---

### 🥉 Bronze Layer: Lưu dữ liệu thô

- Dữ liệu gốc được lưu ở dạng **Delta Table** không chỉnh sửa.
- Hệ thống tự động xử lý **schema inference**, **checkpointing** và **rescue data** cho các dòng bị lỗi.

---

### 🥈 Silver Layer: Làm sạch & chuẩn hóa

- Dữ liệu được chuẩn hóa: định dạng lại cột ngày, xoá bản ghi trùng, xử lý `null`.
- Các bảng chuẩn hoá sẵn sàng cho việc xây dựng mô hình phân tích:
  - `silver_flights`
  - `silver_airports`
  - `silver_passengers`
  - `silver_bookings`

---

### 🥇 Gold Layer: Xây dựng Star Schema

- Xây dựng **Star Schema** với 1 bảng **Fact** và 3 bảng **Dimension**:
  - `fact_flight_bookings`
  - `dim_passengers`
  - `dim_flights`
  - `dim_airports`
- Bảng dimension xử lý theo cơ chế **Slowly Changing Dimensions (SCD Type 2)** để lưu lịch sử thay đổi (ví dụ: hành khách đổi tên, sân bay đổi mã code).

---

### 📊 Kết nối với DBT & BI Tools

- Tạo các **DBT models** từ Gold Layer để kiểm thử, mô hình hoá và triển khai dữ liệu phục vụ báo cáo.
- Dữ liệu đầu ra sẵn sàng để kết nối với **Power BI**, **Tableau**, hoặc phân tích SQL trực tiếp trên Databricks.

---

### 🔁 Tự động hoá toàn bộ quy trình

- Toàn bộ pipeline chạy được bằng 1 luồng tự động sử dụng **Databricks Workflows**.
- Hệ thống hỗ trợ **parameter hoá** với `dbutils.widgets`, cho phép dùng cùng một notebook xử lý nhiều bảng khác nhau.


---
## 📚 Tài liệu tham khảo
Databricks Documentation:https://docs.databricks.com/

Delta Lake:https://delta.io/

DBT Docs:https://docs.getdbt.com/

LakeFlow (Delta Live Tables):https://www.databricks.com/product/lakeflow








  

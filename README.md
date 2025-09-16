# Databricks ETL Pipeline

## Giới thiệu
Dự án này được thực hiện với mục tiêu chính là **học và thực hành các khái niệm Cloud Data Engineering** trên môi trường **Databricks**.  
Project mô phỏng một hệ thống ETL Pipeline end-to-end cho dữ liệu **hàng không – đặt vé máy bay**, bao gồm các bước xử lý dữ liệu theo kiến trúc **Medallion (Bronze – Silver – Gold)**.  

Các điểm nổi bật:
- Làm việc với **dữ liệu thô ở định dạng CSV**.
- Xây dựng **pipeline xử lý dữ liệu tự động** bằng **Delta Live Tables (DLT)**.
- Thiết kế mô hình dữ liệu theo **Star Schema** (Fact & Dimension tables).
- Thực hiện phân tích dữ liệu bằng SQL trong lớp Gold.
- Thực hành các khái niệm **incremental load, slowly changing dimension, surrogate key**.

---

## Kiến trúc tổng thể
Dữ liệu được chia thành 3 lớp theo Medallion Architecture:

- **Bronze**: Lưu trữ dữ liệu thô, giữ nguyên trạng từ nguồn.
- **Silver**: Làm sạch, chuẩn hoá, xử lý quan hệ giữa các bảng.
- **Gold**: Phục vụ phân tích nghiệp vụ, báo cáo.

Sơ đồ tổng quan pipeline:


---

## Bronze Layer

### Input data
Dữ liệu đầu vào gồm các bảng Fact và Dimension ở cả dạng full load và incremental load (xem hình dưới):

- `dim_airports.csv`, `dim_airports_increment.csv`
- `dim_flights.csv`, `dim_flights_increment.csv`
- `dim_passengers.csv`, `dim_passengers_increment.csv`
- `fact_bookings.csv`, `fact_bookings_increment.csv`

![Input Data](./images/input_data.png)

### Xử lý
- Tất cả các file CSV được load trực tiếp vào Databricks bằng **Delta Live Tables**.
- Lớp Bronze giữ nguyên dữ liệu (raw) để đảm bảo khả năng kiểm tra khi có lỗi xảy ra.

---

## Gold Layer

### Mục tiêu
- Thiết kế **Star Schema** từ dữ liệu đã được làm sạch (ở lớp Silver).
- Tạo các bảng **Fact và Dimension** phục vụ phân tích.

### ERD (Entity Relationship Diagram)
Dưới đây là ERD mô tả quan hệ giữa các bảng Fact & Dimension trong hệ thống:

![ERD](./images/bff16ddd-2328-4516-a82a-86b16c64254c.png)

### Ví dụ Query phân tích
Một số câu SQL được thực thi trên lớp Gold để phân tích dữ liệu:

```sql
-- Top 10 sân bay có số lượng chuyến bay nhiều nhất
SELECT a.airport_name, COUNT(f.flight_id) AS total_flights
FROM fact_bookings b
JOIN dim_flights f ON b.flight_id = f.flight_id
JOIN dim_airports a ON f.departure_airport_id = a.airport_id
GROUP BY a.airport_name
ORDER BY total_flights DESC
LIMIT 10;

-- Doanh thu theo từng hãng hàng không
SELECT f.airline, SUM(b.price) AS total_revenue
FROM fact_bookings b
JOIN dim_flights f ON b.flight_id = f.flight_id
GROUP BY f.airline
ORDER BY total_revenue DESC;

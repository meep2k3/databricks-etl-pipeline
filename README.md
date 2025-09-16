# Databricks ETL Pipeline Project

## Giới thiệu

Dự án này được xây dựng với mục tiêu **học tập và thực hành Cloud Data
Engineering trên Databricks**.\
Pipeline được thiết kế theo mô hình **Medallion Architecture (Bronze →
Silver → Gold)** để xử lý dữ liệu từ thô đến báo cáo cuối cùng.

Các bước chính trong pipeline:

-   **Bronze**: ingestion dữ liệu thô (CSV) vào Delta Lake.\
-   **Silver**: làm sạch, chuẩn hóa, và áp dụng Slowly Changing
    Dimension (SCD).\
-   **Gold**: tạo các bảng phân tích (fact + dimension) và chạy các truy
    vấn BI.

------------------------------------------------------------------------

## Cấu trúc dữ liệu đầu vào (Bronze)

Dữ liệu được nạp từ các file CSV ban đầu (raw zone):

    dim_airports.csv
    dim_airports_increment.csv
    dim_flights.csv
    dim_flights_increment.csv
    dim_passengers.csv
    dim_passengers_increment.csv
    fact_bookings.csv
    fact_bookings_increment.csv

Ảnh minh họa dữ liệu đầu vào:

![raw-data](./images/raw-data.png)

------------------------------------------------------------------------

## Bronze Layer -- Ingestion

-   Ingestion dữ liệu từ CSV vào **Delta Lake (Bronze tables)**.\
-   Giữ nguyên format, không làm sạch dữ liệu, để đảm bảo có bản gốc
    phục vụ auditing.

Code mẫu (PySpark on Databricks):

``` python
df = spark.read.format("csv")     .option("header", True)     .load("/Volumes/workspace/raw/dim_airports.csv")

df.write.format("delta").mode("overwrite").save("/Volumes/workspace/bronze/dim_airports")
```

Kết quả: các bảng Delta ở **Bronze Layer**.

------------------------------------------------------------------------

## Silver Layer -- Transformation (Sẽ cập nhật sau)

-   Làm sạch dữ liệu.\
-   Chuẩn hóa schema.\
-   Áp dụng **Slowly Changing Dimension (SCD Type 2)** để quản lý dữ
    liệu thay đổi theo thời gian.\
-   Tích hợp pipeline với **Databricks DLT (Delta Live Tables)** để tự
    động hóa.

👉 Phần này sẽ được bổ sung khi hoàn tất code & pipeline Silver.

------------------------------------------------------------------------

## Gold Layer -- Data Mart

-   Tạo **fact & dimension tables** đã chuẩn hóa.\
-   Dùng SQL để viết các truy vấn phục vụ phân tích.

Ảnh minh họa ERD kết quả:

![erd](./images/erd.png)

Ví dụ truy vấn:

``` sql
-- Top 10 sân bay có nhiều chuyến bay nhất
SELECT a.airport_name, COUNT(f.flight_id) AS total_flights
FROM fact_bookings f
JOIN dim_airports a ON f.airport_id = a.airport_id
GROUP BY a.airport_name
ORDER BY total_flights DESC
LIMIT 10;
```

------------------------------------------------------------------------

## Kết quả đạt được

-   Hiểu và áp dụng thành công kiến trúc **Medallion Architecture** trên
    Databricks.\
-   Xây dựng pipeline ingestion (Bronze) và analytic queries (Gold).\
-   Chuẩn bị sẵn dữ liệu để mở rộng cho **Silver Layer với SCD & DLT**.

------------------------------------------------------------------------

## How to Run

1.  **Clone repo** về máy hoặc Databricks Repo:

    ``` bash
    git clone <repo-url>
    ```

2.  **Import Job JSON** vào Databricks (trong UI → Workflows → Import).

3.  **Chạy Bronze ingestion job** để nạp dữ liệu thô vào Delta Lake.

4.  **Chạy Silver DLT pipeline** (sẽ bổ sung sau).

5.  **Kiểm tra Gold Layer** bằng cách mở notebook SQL và chạy các truy
    vấn.

------------------------------------------------------------------------

## Ghi chú

-   Toàn bộ dự án được xây dựng để **luyện tập kỹ năng Cloud Data
    Engineering trên Databricks**.\
-   Có thể mở rộng để tích hợp với Airflow, Power BI hoặc Superset.

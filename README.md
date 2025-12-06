# Structured Query Language (SQL)

## Table Of Content

- [1. Thao tác với database](#1-thao-tác-với-database)
- [2. Thao tác với Schema](#2-thao-tác-với-schema)
- [3. Thao tác với bảng](#3-thao-tác-với-bảng)
  - [Tạo bảng (Create)](#tạo-bảng-create)
  - [Xoá bảng (Delete)](#xoá-bảng-delete)
  - [Thay đổi bảng (Alter)](#thay-đổi-bảng-alter)
  - [Kiểu dữ liệu](#kiểu-dữ-liệu)
- [4. Truy vấn bảng](#4-truy-vấn-bảng)
  - [4.1. Các lệnh truy vấn](#41-các-lệnh-truy-vấn)
    - [4.1.1. Các phép JOIN trong SQL](#411-các-phép-join-trong-sql)
    - [4.1.2. Condition (Điều kiện)](#412-condition-điều-kiện)
    - [4.1.3. Aggregation (Hàm tổng hợp)](#413-aggregation-hàm-tổng-hợp)
    - [4.1.4. Subquery (Truy vấn con)](#414-subquery-truy-vấn-con)
    - [4.1.5. Toán tử tập hợp](#415-toán-tử-tập-hợp)
  - [4.2. Window Functions](#42-window-functions)
    - [4.2.1 Window](#421-window)
    - [4.2.2. Window function](#422-window-function)
  - [4.3. Common Table Expression (CTE)](#43-common-table-expression-cte)
    - [4.3.1. CTE](#431-cte)
    - [4.3.2. Recursive CTE (CTE đệ quy)](#432-recursive-cte-cte-đệ-quy)
  - [4.4. Một số hàm khác](#44-một-số-hàm-khác)
  - [4.5. View vs Materialized View](#45-view-vs-materialized-view)
    - [4.5.1. View](#451-view)
    - [4.5.2. Materialized View](#452-materialized-view)
  - [4.6. Tối ưu hoá truy vấn (SQL Query Optimization)](#46-tối-ưu-hoá-truy-vấn-sql-query-optimization)

## 1. Thao tác với database

Ta có tạo (Create)

```sql
CREATE DATABASE name;
```

và xoá (Delete).

```sql
DROP DATABASE name;
```

Khi xoá cần ngắt kết nối đến database cần xoá.

## 2. Thao tác với Schema

Schema là tập hợp các đối tượng (objects) trong một cơ sở dữ liệu — gồm:

- Bảng (tables)

- Khung nhìn (views)

- Hàm (functions)

- Thủ tục (stored procedures)

- Chỉ mục (indexes), v.v.

| DBMS                | Có hỗ trợ schema?           | Ghi chú                        |
| ------------------- | --------------------------- | ------------------------------ |
| **MySQL / MariaDB** | ✅ Nhưng gọi là **database** | Mỗi *database* = 1 *schema*    |
| **SQL Server**      | ✅ Có schema thật            | Mặc định là `dbo`              |
| **PostgreSQL**      | ✅ Có schema thật            | Mặc định là `public`           |
| **Oracle**          | ✅ Có schema thật            | Mỗi user = 1 schema            |
| **SQLite**          | ⚠️ Không hỗ trợ schema thật | Toàn bộ nằm trong 1 file `.db` |

Tạo Schema

```sql
CREATE SCHEMA schema_name;
```

Xoá Schema

```sql
DROP SCHEMA schema_name; -- thường chỉ được khi schema trống

-- PostgreSQL thêm CASCADE để xoá luôn các bảng
DROP SCHEMA schema_name CASCADE
```

## 3. Thao tác với bảng

### Tạo bảng (Create)

```sql
CREATE TABLE table_name (
    ...
 );
```

### Xoá bảng (Delete)

```sql
DROP TABLE table_name;
```

### Thay đổi bảng (Alter)

Nhập dữ liệu:

```sql
INSERT INTO table_name (column_names) VALUES (column_values);
```

Xoá dữ liệu:

```sql
-- Xoá theo điều kiện
DELETE FROM table_name WHERE condition;

-- Xoá toàn bộ dữ liệu của bảng
TRUNCATE TABLE table_name;
```

Cập nhật dữ liệu:

```sql
UPDATE table_name
SET column1 = value1,
    column2 = value2,
    ...
WHERE condition;
```

### Kiểu dữ liệu

Mỗi DBMS có các kiểu dữ liệu riêng, nhưng nhìn chung thì chúng được chia thành các nhóm chính: **Số học, chuỗi, ngày giờ, logic, và các kiểu đặc biệt.**

| Loại dữ liệu     | MySQL          | SQL Server    | PostgreSQL | Oracle     | SQLite         |
| ---------------- | -------------- | ------------- | ---------- | ---------- | -------------- |
| Số nguyên        | INT            | INT           | INTEGER    | NUMBER     | INTEGER        |
| Số thực | DECIMAL        | DECIMAL       | NUMERIC    | NUMBER     | REAL           |
| Chuỗi ngắn       | VARCHAR        | (N)VARCHAR      | VARCHAR    | VARCHAR2   | TEXT           |
| Chuỗi dài        | TEXT           | NVARCHAR(MAX) | TEXT       | CLOB       | TEXT           |
| Ngày giờ         | DATETIME       | DATETIME      | TIMESTAMP  | DATE       | TEXT / NUMERIC |
| Logic            | BOOLEAN        | BIT           | BOOLEAN    | (không có) | (không có)     |
| JSON             | JSON           | NVARCHAR      | JSONB      | JSON       | TEXT           |
| Tự tăng          | AUTO_INCREMENT | IDENTITY      | SERIAL     | IDENTITY   | AUTOINCREMENT  |

## 4. Truy vấn bảng

### 4.1. Các lệnh truy vấn

![query](./images/sql_query.png)
*Source: trên ảnh*

Ta thường viết lệnh theo thứ tự:

```FROM -> JOIN -> ON -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY -> LIMIT/TOP/OFFSET```

WHERE sẽ lọc trước khi nhóm, còn HAVING sẽ lọc sau khi nhóm

#### 4.1.1. Các phép JOIN trong SQL

Các phép JOIN thường dùng

![join](./images/join.png)
*Source: https://www.postgresqltutorial.com/*

Ngoài ra còn có NATURAL JOIN (join dựa trên các cột có cùng tên và kiểu dữ liệu), CROSS JOIN (Tích Descartes)

![cross_join](./images/cross_join.avif)

#### 4.1.2. Condition (Điều kiện)

- Toán tử so sánh cơ bản: ```= <> != > < >= <=```

- Trong danh sách: ```(NOT) IN (...)```

- Tồn tại: ```(NOT) EXISTS (...)```

- Null Check: ```IS (NOT) NULL```

- Trong khoảng: ```(NOT) BETWEEN ... AND ...```

- So khớp chuỗi: ```(NOT) LIKE '%Anh_'``` (%: nhiều ký tự bất kỳ, _: một ký tự bất kỳ)

- Toán tử logic: ```AND OR NOT```

#### 4.1.3. Aggregation (Hàm tổng hợp)

- Thường được sử dụng ở `SELECT` sau khi `GROUP BY`

![groupby](./images/groupby.png)

- Một số hàm tổng hợp: `COUNT()`, `MIN()`, `MAX()`, `SUM()`, `AVG()`

#### 4.1.4. Subquery (Truy vấn con)

- Được thực thi trước truy vấn cha, đặt trong dấu `(...)`

- Có thể dùng ở nhiều chỗ nhưng thường dùng ở `WHERE`

- Thường sử dụng trong toán tử ```(NOT) IN/EXISTS (...)```

- Correlated Subquery (Truy vấn con tương quan): nghĩa là mỗi lần xét 1 dòng ở truy vấn ngoài thì SQL sẽ chạy lại truy vấn con để kiểm tra điều kiện tương ứng. Hay nói cách khác là truy vấn con phụ thuộc vào truy vấn cha. Ví dụ:

```sql
-- Hiển thị tất cả các post có lượt likes lớn hơn lượt like trung bình
SELECT * FROM posts 
WHERE likes > ( SELECT AVG(likes) FROM posts);
```

#### 4.1.5. Toán tử tập hợp

Được thực hiện khi số cột, trật tự các cột trong mỗi câu lệnh SELECT phải giống nhau.

Kiểu dữ liệu giống nhau.

- Phép hợp `UNION`: kết hợp 2 hoặc nhiều lệnh SELECT và bỏ các giá trị trùng nhau.

- Phép hợp `UNION ALL`: tương tự `UNION` nhưng không bỏ các giá trị trùng nhau.

- Phép trừ `EXCEPT`: bỏ các dòng ở lệnh SELECT sau.

- Phép giao `INTERSET`: trả về kết quả chung.

Ví dụ:

```sql
SELECT title AS datalist FROM categories 
INTERSECT
SELECT tag AS datalist FROM tags;
```

### 4.2. Window Functions

Là các hàm thực hiện phép tính trên một cửa sổ của các dòng dữ liệu có liên quan đến dòng hiện tại

Khác với `GROUP BY`, hàm cửa sổ không gom nhóm dữ liệu — mỗi dòng vẫn được giữ lại, chỉ thêm một cột kết quả tính toán

![groupbyvswindowfuntions](./images/groupbyvswindowfuntions.png)

Cú pháp

```sql
<window_function>() OVER <window>
```

#### 4.2.1 Window

```sql
WINDOW w AS (
    PARTITION BY column
    ORDER BY column
    ROWS or RANGE ... -- nâng cao
)
```

- `PARTITION BY`: chia dữ liệu thành các nhóm (giống `GROUP BY`)

- `ORDER BY`: Xác định thứ tự xử lý trong từng nhóm

#### 4.2.2. Window function

Hàm tổng hợp: `MIN()`, `MAX()`, `AVG()`, `COUNT()`, `SUM()`

Hàm phân hạng:

- `ROW_NUMBER()`: đánh số thứ tự liên tiếp cho từng dòng (1, 2, 3, 4)

- `RANK()`: xếp hạng nhưng nhảy bậc khi trùng giá trị (1, 2, 2, 4)

- `DENSE_RANK()`: xếp hạng nhưng không nhảy bậc khi trùng giá trị (1, 2, 2, 3)

Hàm giá trị:

- `LAG()`: lấy giá trị dòng trước đó

- `LEAD()`: lấy giá trị dòng sau đó

- `FIRST_VALUE()`: lấy giá trị đầu tiên của cửa sổ

- `LAST_VALUE()`, `NTH_VALUE()`

Hàm thống kê / tỷ lệ:

- `NTITLE(n)`: chia dữ liệu thành n nhóm xấp xỉ bằng nhau

- `PERCENT_RANK()`: trả về thứ hạng phần trăm của dòng trong cửa sổ (%)

- `CUME_DIST()`: tính phân phối tích luỹ (%) có bao nhiêu % <=

- v.v.

```FROM -> JOIN -> ON -> WHERE -> GROUP BY -> HAVING -> WINDOW FUNCTIONS -> SELECT -> ORDER BY -> LIMIT/TOP/OFFSET```

### 4.3. Common Table Expression (CTE)

#### 4.3.1. CTE

Là một bảng tạm đặt tên, chỉ tồn tại trong phạm vi truy vấn đó.

Giúp truy vấn SQL dễ đọc, dễ bảo trì hơn.

```sql
WITH cte_name AS(
    CTE_query_definition
)
statement;
```

Dùng nhiều CTE:

```sql
WITH 
cte_1 AS(
    CTE_query_definition
),
cte_2 AS(
    CTE_query_definition
)
statement;
```

#### 4.3.2. Recursive CTE (CTE đệ quy)

Làm việc với cấu trúc phân cấp (tree)

Ví dụ:

Bảng nhân viên

| MaNV | TenNV | MaQL |
| ---- | ----- | ---- |
| 1    | An    | NULL |
| 2    | Binh  | 1    |
| 3    | Chi   | 2    |
| 4    | Dung  | 2    |

CTE đệ quy:

```sql
WITH RECURSIVE CapQuanLy AS (
    -- Bước 1: chọn gốc
    SELECT MaNV, TenNV, MaQL
    FROM NhanVien
    WHERE TenNV = 'Dung'

    UNION ALL

    -- Bước 2: nối dần lên cấp trên
    SELECT nv.MaNV, nv.TenNV, nv.MaQL
    FROM NhanVien nv
    JOIN CapQuanLy cq ON nv.MaNV = cq.MaQL
)
SELECT * FROM CapQuanLy;
```

Kết quả:

| MaNV | TenNV | MaQL |
| ---- | ----- | ---- |
| 4    | Dung  | 2    |
| 2    | Binh  | 1    |
| 1    | An    | NULL |

Ta đã truy ngược từ Dung → Binh → An

### 4.4. Một số hàm khác

CASE ... WHEN: cấu trúc rẽ nhánh trong SQL

```sql
CASE
    WHEN condition_1 THEN result_1
    WHEN condition_2 THEN result_2
    ...
    ELSE default_result
END
```

COALESCE(): trả về đối số non-null đầu tiên

```sql
SELECT COALESCE(NULL, 2, 1) -- 2
```

CAST(): chuyển đổi kiểu dữ liệu

```sql
CAST('100' AS INTEGER)
```

TO BE CONTINUE ...

### 4.5. View vs Materialized View

#### 4.5.1. View

Là bảng ảo (virtual table) được tạo từ một câu truy vấn để lưu câu truy vấn đó.

Khi bạn SELECT từ view, SQL sẽ chạy lại truy vấn gốc để trả dữ liệu mới nhất.

```sql
-- Tạo View
CREATE VIEW view_name AS
SELECT column1, column2, ...
FROM table_name
WHERE condition;

-- Xoá View
DROP VIEW view_name;

-- Sửa View
CREATE OR REPLACE VIEW view_name AS
SELECT ...
FROM ...
```

#### 4.5.2. Materialized View

Là bảng được lưu trữ vật lý (lên ổ cứng) dựa trên kết quả của một câu SELECT.

Dữ liệu không tự động cập nhật ngay khi bảng gốc thay đổi.

Phải **REFRESH** để cập nhật dữ liệu mới.

→ Nhanh hơn view thường vì không phải chạy lại câu query mỗi lần SELECT.

```sql
-- Tạo MV
CREATE MATERIALIZED VIEW mv_sales_summary AS
SELECT customer_id, SUM(amount) AS total_amount
FROM sales
GROUP BY customer_id;

-- Cập nhật dữ liệu (REFRESH)
REFRESH MATERIALIZED VIEW mv_sales_summary;
```

Pros:

- Tăng tốc truy vấn: hữu ích khi JOIN nhiều bảng lớn, tính toán (hàm tổng hợp, hàm cửa sổ, ...), reporting/dashboard/OLAP.

- Giảm tải cho DB: thay vì chạy query nhiều lần thì MV chỉ chạy 1 lần khi REFRESH.

Cons:

- Tốn dung lượng lưu trữ.

- Dữ liệu không real-time: cần REFRESH để cập nhật.

- Phải quản lý REFRESH: REFRESH thường xuyên gây tốn tài nguyên.

Tham khảo: https://www.datacamp.com/tutorial/sql-materialized-view

### 4.6. Tối ưu hoá truy vấn (SQL Query Optimization)

Một số kỹ thuật tối ưu hoá truy vấn:

**1. Sử dụng Index:**

Không có Index → DB phải quét toàn bảng (Table Scan)

Có Index → DB chỉ tìm đúng vị trí (Index Seek)

Có 3 loại Index chính:

- Clustered Indexes: tự động tạo khi đặt PRIMARY KEY (mỗi bảng chỉ có 1 clustered index). Dữ liệu trong bảng được sắp xếp vật lý theo index này.

- Non-Clustered Indexes: là index tự tạo (mỗi bảng có thể có nhiều non-clustered indexes). Không sắp xếp lại dữ liệu thật và chỉ chứa "con trỏ" đến dòng dữ liệu thật.

```sql
CREATE INDEX index_customer_id ON customers (customer_id);
```

- Full-text indexes: tìm kiếm theo từ khoá trong văn bản dài (theo từ, cụm từ, gần đúng, ...). Thường dùng cho mô tả sản phẩm, bài viết, cmt, bài báo, text dài.

```sql
ALTER TABLE SanPham ADD FULLTEXT(TenSP, MoTa);

SELECT * FROM SanPham
WHERE MATCH(MoTa) AGAINST('cà phê nguyên chất');
```

- Ngoài ra còn Composite Index, Hash/B-Tree Index, ...

Lưu ý:

- Sử dụng index lên các cột thường xuyên sử dụng để truy vấn (WHERE, JOIN, ORDER BY, GROUP BY hay các cột có tính phân biệt cao)

- Index sẽ làm chậm đi các thao tác INSERT, UPDATE, DELETE nên ta cần tránh việc này.

- Chọn đúng loại index phù hợp.

**2. Sử dụng SELECT hợp lý:** chỉ lấy các cột cần thiết tránh sử dụng `SELECT *`.

**3. Sử dụng LIMIT khi chỉ cần 1 phần dữ liệu:** dùng LIMIT sẽ giúp giảm đi số dòng trả về, giúp truy vấn nhanh hơn.

**4. Sử dụng phép JOIN hiệu quả:** dùng đúng phép JOIN và chỉ JOIN bảng cần thiết.

- Thứ tự JOINs hợp lý: nên bắt đầu với các bảng trả về ít dòng hơn. Điều này giúp giảm đi lượng data cần để xử lý trong những lần JOINs tiếp theo.

- Sử dụng Indexes lên cột cần JOIN.

- Cân nhắc sử dụng Subqueries hoặc CTEs để đơn gian hoá các phép JOIN phức tạp.

**5. Phân tích Query Execution Plans:** sử dụng hàm EXPLAIN để biết Execution Plans nhằm tránh câu truy vấn tồi.

- Tránh **Full table scan**.

- Chiến lược JOIN kém hiệu quả

- Các vấn đề tiềm ẩn khác.

**6. Tối ưu điều kiện WHERE:**

- **Thêm điều kiện lọc vào sớm**: lọc được nhiều dòng nhất có thể ở điều kiện đầu tiên giúp truy vấn chạy nhanh hơn.

- **Tránh sử dụng hàm lên cột**: khi áp dùng hàm, DB phải áp dụng hàm lên từng dòng trong bảng trước khi lọc ra kết quả. Nó sẽ ngăn việc sử dụng Indexes hiệu quả.

- **Sử dụng toán tử thích hợp**: cần sử dụng toán tử hiệu quả đáp ứng yêu cầu của câu truy vấn. Ví dụ, `=` sẽ nhanh hơn `LIKE`, sử dụng phạm vi ngày cụ thể sẽ tốt hơn dùng hàm như `MONTH(order_date)`

**7. Tránh sử dụng Subqueries:**

- Thay thế bằng phép JOIN nếu có thể: JOIN thường nhanh hơn và hiệu quả hơn so với Subqueries.

- Sử dụng CTEs thay vì Subqueries.

**8. Sử dụng EXISTS thay vì IN:** thường sử dụng khi làm việc với Subqueries.

**9. Tránh sử dụng DISTINCT:** gây tốn tài nguyên khi xử lý dữ liệu lớn.

- Thay bằng GROUP BY nếu có thể.

- Xử lý trùng lặp dữ liệu ngay từ giai đoạn data cleaning.

**10. Tận dụng các tính năng dành riêng cho CSDL:** mỗi DBMS đều có tính năng riêng của họ.

**11. Tối ưu ORDER BY / GROUP BY:**

- Tránh sử dụng khi không cần thiết.

- Dùng Indexes. Nếu áp dụng nhiều cột → dùng composite index (col1, col2)

- Tổng hợp data trước: dùng Materialized View.

**12. Sử dụng UNION ALL thay vì UNION:** do UNION sẽ thực hiện DISTINCT.

**13. Chia nhỏ truy vấn phức tạp:** dùng Materialized View.
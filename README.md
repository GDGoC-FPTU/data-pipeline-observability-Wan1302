[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=23572394&assignment_repo_type=AssignmentRepo)

# Day 10 Lab: Data Pipeline & Data Observability

**Student ID:** AI20K-0081  
**Student Email:** duyquangho1302@gmail.com  
**Name:** Hồ Trọng Duy Quang

---

## Mô tả

Bài lab này xây dựng một pipeline ETL đơn giản bằng Python để xử lý dữ liệu sản phẩm từ file JSON. Pipeline thực hiện 4 bước chính:

1. **Extract:** đọc dữ liệu từ `raw_data.json`.
2. **Validate:** loại bỏ record không hợp lệ, gồm `price <= 0` hoặc `category` bị thiếu/rỗng.
3. **Transform:** chuẩn hóa `category` sang Title Case, tính `discounted_price = price * 0.9`, và thêm cột `processed_at`.
4. **Load:** lưu dữ liệu sạch vào `processed_data.csv`.

Ngoài phần ETL, bài lab còn có thí nghiệm Data Observability bằng cách chạy `agent_simulation.py` với dữ liệu sạch và dữ liệu rác để quan sát ảnh hưởng của chất lượng dữ liệu đến câu trả lời của AI Agent.

---

## Cách chạy

### 1. Chuẩn bị môi trường

```bash
python -m venv venv
```

Trên Windows PowerShell:

```powershell
.\venv\Scripts\activate
```

Trên macOS/Linux:

```bash
source venv/bin/activate
```

Cài thư viện cần thiết:

```bash
pip install pandas pytest
```

### 2. Chạy ETL Pipeline

```bash
python solution.py
```

Kết quả chạy pipeline:

```text
ETL Pipeline Started...
Extracting data from raw_data.json...
Validation summary: 3 kept, 2 dropped.
Errors found: [{'id': 3, 'reason': 'Price <= 0'}, {'id': 4, 'reason': 'Missing Category'}]
Successfully loaded 3 records to processed_data.csv
Pipeline completed! 3 records saved.
```

### 3. Tạo dữ liệu rác cho stress test

```bash
python generate_garbage.py
```

File `garbage_data.csv` được tạo với một số lỗi dữ liệu như duplicate ID, sai kiểu dữ liệu, outlier lớn bất thường và giá trị null.

### 4. Chạy Agent Simulation

```bash
python agent_simulation.py
```

Kết quả:

```text
Testing with CLEAN data:
Agent: Based on my data, the best choice is Laptop at $1200.

Testing with GARBAGE data:
Agent: Based on my data, the best choice is Nuclear Reactor at $999999.
```

---

## Cấu trúc thư mục

```text
├── solution.py              # Script ETL chính
├── raw_data.json            # Dữ liệu nguồn ban đầu
├── processed_data.csv       # Dữ liệu sạch sau khi chạy pipeline
├── generate_garbage.py      # Script tạo dữ liệu rác
├── garbage_data.csv         # Dữ liệu rác dùng để stress test Agent
├── agent_simulation.py      # Script mô phỏng AI Agent đọc dữ liệu
├── experiment_report.md     # Báo cáo thí nghiệm Clean Data vs Garbage Data
├── tests/                   # Test tự động của bài lab
└── README.md                # Tài liệu hướng dẫn và tóm tắt bài làm
```

---

## Kết quả ETL

Dữ liệu đầu vào trong `raw_data.json` có 5 records:

| ID | Product | Price | Category | Trạng thái |
|----|---------|-------|----------|------------|
| 1 | Laptop | 1200 | electronics | Hợp lệ |
| 2 | Chair | 45 | furniture | Hợp lệ |
| 3 | Mystery Box | -10 | misc | Bị loại vì `price <= 0` |
| 4 | Phone | 800 | rỗng | Bị loại vì thiếu `category` |
| 5 | Monitor | 300 | electronics | Hợp lệ |

Sau khi validate và transform:

- **3 records** được giữ lại.
- **2 records** bị loại bỏ.
- `category` được chuẩn hóa thành `Electronics`, `Furniture`.
- `discounted_price` được tính giảm 10%.
- `processed_at` được thêm để ghi nhận thời điểm xử lý.

Kết quả trong `processed_data.csv`:

| ID | Product | Price | Category | Discounted Price |
|----|---------|-------|----------|------------------|
| 1 | Laptop | 1200 | Electronics | 1080.0 |
| 2 | Chair | 45 | Furniture | 40.5 |
| 5 | Monitor | 300 | Electronics | 270.0 |

---

## Nhận xét

Pipeline đã giúp loại bỏ dữ liệu không hợp lệ trước khi dữ liệu được dùng bởi Agent. Khi dùng `processed_data.csv`, Agent chọn Laptop là sản phẩm electronics tốt nhất theo logic giá cao nhất trong dữ liệu sạch. Khi dùng `garbage_data.csv`, Agent bị ảnh hưởng bởi outlier `Nuclear Reactor` có giá `$999999`, dẫn đến câu trả lời sai ngữ cảnh. Điều này cho thấy chất lượng dữ liệu là yếu tố rất quan trọng trong các hệ thống AI/RAG.

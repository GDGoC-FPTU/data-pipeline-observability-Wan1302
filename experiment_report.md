# Experiment Report: Data Quality Impact on AI Agent

**Student ID:** AI20K-2A202600081
**Name:** Hồ Trọng Duy Quang  
**Date:** 15/04/2026

---

## 1. Kết quả thí nghiệm

Chạy `agent_simulation.py` với 2 bộ dữ liệu:

- `processed_data.csv`: dữ liệu sạch được tạo từ pipeline ETL.
- `garbage_data.csv`: dữ liệu rác được tạo bởi `generate_garbage.py`.

| Scenario | Agent Response | Accuracy (1-10) | Notes |
|----------|----------------|-----------------|-------|
| Clean Data (`processed_data.csv`) | Based on my data, the best choice is Laptop at $1200. | 9 | Câu trả lời hợp lý vì dữ liệu đã được validate, category electronics có Laptop và Monitor, trong đó Laptop có giá cao hơn. |
| Garbage Data (`garbage_data.csv`) | Based on my data, the best choice is Nuclear Reactor at $999999. | 1 | Câu trả lời sai ngữ cảnh vì dữ liệu bị nhiễm outlier rất lớn, khiến Agent chọn một sản phẩm không phù hợp. |

Kết quả chạy thực tế:

```text
Testing with CLEAN data:
Agent: Based on my data, the best choice is Laptop at $1200.

Testing with GARBAGE data:
Agent: Based on my data, the best choice is Nuclear Reactor at $999999.
```

---

## 2. Phan tich & nhận xét

### Tại sao Agent trả lời đúng khi dùng Clean Data?

Trong `processed_data.csv`, dữ liệu đã đi qua pipeline ETL nên các record không hợp lệ đã bị loại bỏ. Pipeline giữ lại 3 record hợp lệ và loại 2 record lỗi:

- ID 3 bị loại vì `price <= 0`.
- ID 4 bị loại vì thiếu `category`.

Dữ liệu còn lại có cấu trúc rõ ràng, giá là số hợp lệ, category được chuẩn hóa, và có thêm thông tin `discounted_price`, `processed_at`. Khi Agent tìm sản phẩm thuộc category electronics, nó chỉ so sánh giữa Laptop giá 1200 và Monitor giá 300. Vì logic của Agent chọn sản phẩm có giá cao nhất trong category electronics, kết quả Laptop là hợp lý.

### Tại sao Agent trả lời sai khi dùng Garbage Data?

Agent trả lời sai khi dùng `garbage_data.csv` vì dữ liệu này chứa nhiều vấn đề chất lượng dữ liệu. File rác có duplicate ID, sai kiểu dữ liệu như giá `"ten dollars"`, giá trị null, record có giá bằng 0, và đặc biệt là outlier `Nuclear Reactor` với giá `999999` trong category electronics. Agent không hiểu ngữ cảnh thực tế của sản phẩm, nó chỉ đọc dữ liệu và chọn record có giá cao nhất theo logic được lập trình. Vì vậy, outlier cực lớn đã "đầu độc" kết quả, làm Agent chọn Nuclear Reactor thay vì một sản phẩm điện tử hợp lý như Laptop.

Điều này cho thấy một prompt tốt vẫn có thể tạo ra câu trả lời sai nếu dữ liệu đầu vào bị lỗi. Trong các hệ thống AI hoặc RAG, model thường phụ thuộc rất nhiều vào dữ liệu được cung cấp. Nếu dữ liệu có record bất thường, thiếu kiểm tra, hoặc không được làm sạch, Agent có thể đưa ra kết luận nghe có vẻ tự tin nhưng thực tế lại sai.

### Các vấn đề dữ liệu quan sát được trong Garbage Data

| Vấn đề | Ví dụ | Ảnh hưởng |
|--------|-------|-----------|
| Duplicate ID | ID 1 xuất hiện ở cả Laptop và Banana | Làm mất tính duy nhất của record, dễ gây nhầm lẫn khi truy xuất dữ liệu. |
| Wrong data type | `Broken Chair` có price là `"ten dollars"` | Có thể gây lỗi tính toán hoặc làm pipeline/Agent xử lý sai. |
| Extreme outlier | `Nuclear Reactor` giá `999999` | Làm Agent chọn kết quả sai vì giá quá lớn bất thường. |
| Null values | `Ghost Item` thiếu ID và category | Làm dữ liệu không đầy đủ, dễ gây lỗi lọc hoặc phân loại. |
| Invalid price | `Ghost Item` có price `0` | Không phù hợp với rule validate của pipeline. |

---

## 3. Kết luận

**Quality Data > Quality Prompt?** Đồng ý.

Một prompt tốt có thể hướng dẫn Agent cách trả lời, nhưng prompt không thể tự sửa dữ liệu sai nếu hệ thống vẫn đưa dữ liệu rác vào làm nguồn tri thức. Trong thí nghiệm này, cùng một câu hỏi và cùng một Agent logic, dữ liệu sạch cho kết quả hợp lý là Laptop, còn dữ liệu rác khiến Agent chọn Nuclear Reactor. Vì vậy, chất lượng dữ liệu là nền tảng quan trọng để hệ thống AI đưa ra câu trả lời đáng tin cậy.

ETL pipeline đóng vai trò như lớp kiểm soát chất lượng ban đầu. Việc validate, chuẩn hóa và ghi log số lượng record bị loại giúp phát hiện lỗi sớm hơn, giảm nguy cơ Agent tạo ra câu trả lời sai do dữ liệu bị nhiễm hoặc không nhất quán.

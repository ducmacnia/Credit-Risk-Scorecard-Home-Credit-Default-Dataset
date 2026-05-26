# Credit Risk Scorecard - Home Credit Default Risk

## 1. Tổng quan

Dự án này xây dựng mô hình đánh giá rủi ro tín dụng cho bộ dữ liệu **Home Credit Default Risk**. Mục tiêu là dự đoán khả năng khách hàng gặp khó khăn thanh toán, đồng thời tạo ra một quy trình có thể diễn giải được cho nghiệp vụ tín dụng.

Dự án triển khai song song hai hướng mô hình:

- **Logistic Regression Scorecard**: tập trung vào tính diễn giải, sử dụng WOE/IV, chọn biến và chuyển đổi xác suất thành điểm tín dụng.
- **LightGBM**: tập trung vào năng lực dự báo, dùng làm mô hình benchmark có hiệu suất cao hơn với dữ liệu dạng bảng.

## 2. Dataset

Nguồn dữ liệu: Kaggle - Home Credit Default Risk  
Link: https://www.kaggle.com/competitions/home-credit-default-risk/data

Dữ liệu không được đưa trực tiếp vào repository. Sau khi tải từ Kaggle, đặt các file CSV vào thư mục:

```text
data/raw/
```

Các file cần có:

```text
application_train.csv
application_test.csv
bureau.csv
bureau_balance.csv
previous_application.csv
POS_CASH_balance.csv
installments_payments.csv
credit_card_balance.csv
HomeCredit_columns_description.csv
```

### Vai trò các bảng chính

| File | Vai trò |
| --- | --- |
| `application_train.csv` | Bảng hồ sơ vay chính, có biến mục tiêu `TARGET` |
| `application_test.csv` | Bảng hồ sơ cần dự đoán, không có `TARGET` |
| `bureau.csv` | Lịch sử tín dụng của khách hàng tại tổ chức tín dụng khác |
| `bureau_balance.csv` | Trạng thái dư nợ hàng tháng của các khoản trong `bureau.csv` |
| `previous_application.csv` | Lịch sử các hồ sơ vay trước đây tại Home Credit |
| `POS_CASH_balance.csv` | Lịch sử dư nợ POS/cash loan theo tháng |
| `installments_payments.csv` | Lịch sử thanh toán từng kỳ |
| `credit_card_balance.csv` | Lịch sử dư nợ thẻ tín dụng theo tháng |

## 3. Cấu trúc repository

```text
credit-risk-scorecard-github/
├── README.md
├── requirements.txt
├── notebooks/
│   └── credit_risk_model_enhanced.ipynb
└── data/
    └── README.md
```

Repository chỉ lưu mã nguồn, notebook và tài liệu mô tả. Thư mục dữ liệu thực tế được loại khỏi Git để tránh upload file lớn và tuân thủ điều kiện sử dụng dữ liệu.

## 4. Chiến lược thực hiện

### Bước 1: Khảo sát dữ liệu

- Kiểm tra kích thước, khóa liên kết và tỷ lệ thiếu dữ liệu của từng bảng.
- Phân tích phân phối biến mục tiêu `TARGET`.
- Nhận diện các nhóm biến quan trọng như thu nhập, hạn mức tín dụng, thông tin nghề nghiệp, lịch sử tín dụng và điểm tín dụng bên ngoài.

### Bước 2: Làm sạch và chuẩn hóa

- Xử lý missing values theo loại biến.
- Chuẩn hóa các biến ngày dạng số âm thành đặc trưng dễ hiểu hơn, ví dụ tuổi, thâm niên làm việc, thời gian từ lần thay đổi thông tin gần nhất.
- Xử lý giá trị bất thường, đặc biệt là các mã hóa đặc biệt như `DAYS_EMPLOYED = 365243`.

### Bước 3: Feature engineering

- Tạo các tỷ lệ tài chính như credit-to-income, annuity-to-income, credit-to-goods.
- Tổng hợp thông tin từ các bảng phụ theo `SK_ID_CURR`.
- Tạo đặc trưng từ lịch sử trả chậm, số lần vay trước, trạng thái khoản vay, tổng dư nợ và hành vi thanh toán.

### Bước 4: WOE/IV và scorecard

- Chia bin các biến phù hợp với bài toán rủi ro tín dụng.
- Tính Weight of Evidence và Information Value để đánh giá sức mạnh dự báo.
- Chọn tập biến gọn, dễ giải thích và hạn chế đa cộng tuyến.
- Huấn luyện Logistic Regression trên biến đã WOE-transform.
- Chuyển đổi xác suất dự báo thành điểm tín dụng và nhóm rủi ro.

### Bước 5: Mô hình LightGBM

- Huấn luyện LightGBM trên tập đặc trưng mở rộng.
- Dùng validation set để kiểm soát overfitting.
- Đánh giá bằng các chỉ số phù hợp với credit scoring như GINI, AUC và KS.

### Bước 6: So sánh và diễn giải

- So sánh Logistic Regression Scorecard và LightGBM.
- Phân tích trade-off giữa hiệu suất dự báo và khả năng diễn giải.
- Đưa ra khuyến nghị mô hình theo mục tiêu sử dụng: scorecard minh bạch hoặc mô hình benchmark hiệu suất cao.

## 5. Cách chạy

Tạo môi trường Python và cài thư viện:

```bash
pip install -r requirements.txt
```

Mở notebook:

```bash
jupyter notebook notebooks/credit_risk_model_enhanced.ipynb
```

Sau đó chạy notebook từ trên xuống dưới. Lần chạy đầu có thể mất nhiều thời gian do các bước tổng hợp dữ liệu từ nhiều bảng lớn.

## 6. Kết quả chính

Notebook tạo ra hai nhóm kết quả:

- Mô hình scorecard có khả năng diễn giải theo biến, điểm tín dụng và nhóm rủi ro.
- Mô hình LightGBM có hiệu suất dự báo tốt hơn, phù hợp làm benchmark hoặc mô hình sản xuất khi ưu tiên độ chính xác.

Các chỉ số đánh giá chính:

- **AUC**: khả năng phân biệt khách hàng rủi ro cao và thấp.
- **GINI**: thước đo phổ biến trong credit scoring, tính từ AUC.
- **KS**: đo khoảng cách lớn nhất giữa phân phối nhóm tốt và nhóm xấu.

## 7. Ghi chú triển khai

- Không commit dữ liệu raw lên GitHub.
- Không commit file mô hình sinh ra trong quá trình chạy nếu không cần thiết.
- Nếu muốn tái lập kết quả đầy đủ, cần tải đúng các file dữ liệu từ Kaggle và đặt vào `data/raw/`.

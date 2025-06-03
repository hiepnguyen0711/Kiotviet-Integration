# 🚀 Hướng Dẫn Tích Hợp KiotViet API - Đồng Bộ Đơn Hàng

## ✅ Mục Tiêu

Tích hợp thành công **KiotViet API** để đồng bộ dữ liệu giữa website và hệ thống POS của KiotViet:

- ✅ Tự động tạo đơn hàng trên KiotViet từ website
- ✅ Tự động tạo sản phẩm trên KiotViet nếu chưa có
- ✅ Tự động tạo/tìm khách hàng
- ✅ Ghi lại KiotViet Order ID để theo dõi
- ✅ Ghi log chi tiết cho debugging

## 📋 Cấu Trúc Hệ Thống

```
Website Order → checkout.php → KiotVietAPI Class → KiotViet POS
```

## 🗃️ Các Bảng Cơ Sở Dữ Liệu

| Bảng                | Mục Đích                  | Trường Quan Trọng                |
|---------------------|---------------------------|----------------------------------|
| `db_kiotviet_config`| Cấu hình API và token     | `access_token`, `expires_at`     |
| `db_kiotviet_logs`  | Log hoạt động             | `type`, `message`, `created_at`  |
| `db_dathang`        | Gắn thông tin KiotViet    | `kiotviet_order_id`, `synced_at` |
| `db_sanpham`        | Gắn thông tin sản phẩm KV | `kiotviet_id`, `synced_at`       |

## ⚙️ File Cần Thiết

### 📁 `sources/lib/kiotviet_api.php`

> **Class chính xử lý kết nối KiotViet API**

```php
class KiotVietAPI {
    private $retailer = '**[TÊN KẾT NỐI CỦA BẠN]**';
    private $client_id = '**[CLIENT_ID]**';
    private $client_secret = '**[CLIENT_SECRET]**';
    private $base_url = 'https://public.kiotapi.com';

    // Các hàm chính:
    // - get_access_token()
    // - create_order_in_kiotviet($order_id)
    // - get_or_create_customer()
    // - create_or_get_default_product_in_kiotviet()
}
```

### 📁 `sources/ajax/checkout.php`

```php
if ($order_id) {
    $kiotviet_config = [
        'retailer' => '**[TÊN KẾT NỐI]**',
        'client_id' => '**[CLIENT_ID]**',
        'client_secret' => '**[CLIENT_SECRET]**',
        'base_url' => 'https://public.kiotapi.com'
    ];

    $kiotviet = new KiotVietAPI($kiotviet_config, $d);
    $result = $kiotviet->create_order_in_kiotviet($order_id);
}
```

## 🔄 Luồng Xử Lý

1. Khách đặt hàng trên website → submit → tạo đơn hàng
2. Lấy token từ KiotViet nếu hết hạn
3. Gửi đơn hàng lên KiotViet
4. Nếu sản phẩm chưa có trên KiotViet → tạo mới
5. Nếu khách hàng chưa có trên KiotViet → tạo mới

## ✅ Xác Nhận Thành Công

```sql
SELECT * FROM db_dathang WHERE kiotviet_order_id IS NOT NULL ORDER BY id DESC;
SELECT * FROM db_kiotviet_logs WHERE type = 'success';
```

## 🧪 Test Case Thành Công

| Trường         | Giá Trị Test             | Kết Quả       |
|----------------|--------------------------|---------------|
| Website Order  | `DH-NU6V5`               | ✅ Thành công |
| Order ID KV    | `14575144`               | ✅ Thành công |
| Sản phẩm       | `KEO DÁN ĐA NĂNG`         | ✅ Auto Sync  |
| Khách hàng     | `test.kiotviet@email.com`| ✅ Thành công |

## 📈 Theo Dõi Logs

```sql
-- Log thành công
SELECT created_at, message FROM db_kiotviet_logs WHERE type = 'success';

-- Log lỗi
SELECT created_at, message FROM db_kiotviet_logs WHERE type = 'error';

-- Thống kê
SELECT COUNT(*) as total_orders, COUNT(kiotviet_order_id) as synced_orders FROM db_dathang;
```

## 🔧 Áp Dụng Cho Dự Án Khác

Checklist:
- Tạo bảng `db_kiotviet_config`, `db_kiotviet_logs`
- Copy và điều chỉnh `KiotVietAPI.php`
- Tích hợp vào quy trình checkout
- Test kỹ với đơn hàng và sản phẩm thật

## 🔐 Thông Tin Bảo Mật

> 🚫 **KHÔNG ĐƯỢC commit các thông tin như:**
> - `client_id`
> - `client_secret`
> - `access_token`

Thay bằng biến môi trường `.env` hoặc định nghĩa cấu hình bên ngoài file mã nguồn.

## 📢 Ghi Chú

Tài liệu chính thức:  
**[https://www.kiotviet.vn/huong-dan-su-dung-public-api-retail](https://www.kiotviet.vn/huong-dan-su-dung-public-api-retail)**


### 🔍 Test đơn hàng KiotViet với Postman

#### 1. **Lấy access token**
- **URL:**  
  `https://id.kiotviet.vn/connect/token`
- **Method:** `POST`
- **Header:**
  ```
  Content-Type: application/x-www-form-urlencoded
  ```
- **Body (x-www-form-urlencoded):**
  ```
  grant_type: password
  client_id: **<YOUR_CLIENT_ID>**
  client_secret: **<YOUR_CLIENT_SECRET>**
  username: **<YOUR_USERNAME>**
  password: **<YOUR_PASSWORD>**
  ```
- **Kết quả:** Trả về `access_token` dùng để gọi các API tiếp theo

#### 2. **Tạo đơn hàng mới**
- **URL:**  
  `https://public.kiotapi.com/orders`
- **Method:** `POST`
- **Header:**
  ```
  Authorization: Bearer <access_token>
  Retailer: <YOUR_RETAILER_NAME>
  Content-Type: application/json
  ```
- **Body (JSON):**
  ```json
  {
    "branchId": 12345,
    "customerId": 67890,
    "orderDetails": [
      {
        "productCode": "SP0001",
        "quantity": 1,
        "price": 100000
      }
    ],
    "totalPayment": 100000
  }
  ```

#### 3. **Xem danh sách đơn hàng**
- **URL:**  
  `https://public.kiotapi.com/orders?pageSize=10&pageIndex=0`
- **Method:** `GET`
- **Header:**
  ```
  Authorization: Bearer <access_token>
  Retailer: <YOUR_RETAILER_NAME>
  ```

## 👤 Tác Giả

**Nguyễn Văn Hiệp**  
🔗 [Facebook](https://www.facebook.com/G.N.S.L.7/)  
☕ **Ủng hộ cà phê:**  
Số TK: `0601 5576 3127`  
Chủ TK: `NGUYEN VAN HIEP`  
Ngân hàng: `Sacombank - PGD LÊ VĂN QUỚI`
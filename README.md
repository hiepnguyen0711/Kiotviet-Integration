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
    private $retailer = '**[YOUR_RETAILER_NAME]**';
    private $client_id = '**[YOUR_CLIENT_ID]**';
    private $client_secret = '**[YOUR_CLIENT_SECRET]**';
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
        'retailer' => '**[YOUR_RETAILER_NAME]**',
        'client_id' => '**[YOUR_CLIENT_ID]**',
        'client_secret' => '**[YOUR_CLIENT_SECRET]**',
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

## 🧪 Test Trên Postman

- Gửi đơn hàng mới thông qua endpoint nội bộ (giả định):  
  `POST https://yourwebsite.com/sources/ajax/checkout.php`

- Body:
```json
{
  "id_order": "12345",
  "customer": {
    "name": "Nguyễn Văn A",
    "phone": "098xxxxxx",
    "email": "example@gmail.com"
  }
}
```

- Kiểm tra phản hồi:
  - `200 OK`
  - `kiotviet_order_id` đã có trong database

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

## 📢 Lưu Ý Bảo Mật

> 🚫 **KHÔNG ĐƯỢC commit các thông tin như:**
> - `client_id`
> - `client_secret`
> - `access_token`

Hãy sử dụng biến môi trường `.env` hoặc include file cấu hình ngoài.

## 👤 Tác Giả

**Nguyễn Văn Hiệp**  
🔗 [Facebook](https://www.facebook.com/G.N.S.L.7/)  
☕ **Ủng hộ cà phê:**  
Số TK: `0601 5576 3127`  
Chủ TK: `NGUYEN VAN HIEP`  
Ngân hàng: `Sacombank - PGD LÊ VĂN QUỚI`

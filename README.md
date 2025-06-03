# kiotvietapiforecommercewebsite

**Tích Hợp KiotViet API cho Website Bán Hàng – Đồng Bộ Sản Phẩm & Đơn Hàng Tự Động**

## 📌 Giới thiệu

Dự án này giúp tích hợp hệ thống bán hàng trên website với nền tảng quản lý KiotViet thông qua Public API. Hệ thống cho phép:

- Đồng bộ **sản phẩm** từ website lên KiotViet.
- Gửi **đơn hàng mới** từ website vào KiotViet để theo dõi và đóng đơn.
- Kiểm tra trạng thái đơn hàng theo thời gian thực từ hệ thống KiotViet.

## 🔐 Yêu cầu cấu hình

- PHP >= 7.3
- Có tài khoản KiotViet doanh nghiệp
- Cấu hình **App ID**, **Secret Key**, và **Tên cửa hàng KiotViet**

> 🔑 Thay các thông tin sau trong file config:
> 
> - **App ID:** `**YOUR_APP_ID**`
> - **Secret Key:** `**YOUR_SECRET_KEY**`
> - **Retailer:** `**YOUR_RETAILER_NAME**`

Tài liệu chính thức: https://www.kiotviet.vn/huong-dan-su-dung-public-api-retail/

## 🚀 Hướng dẫn sử dụng

1. Clone source về và cấu hình kết nối API KiotViet.
2. Kết nối OAuth2 lấy token truy cập.
3. Sử dụng API `/orders` để đẩy đơn hàng mới khi người dùng đặt hàng.
4. Đơn hàng sau khi tạo sẽ hiển thị trong hệ thống KiotViet để xử lý.
5. Thay thế "giá sỉ/lẻ" bằng 1 cột chờ duyệt, khi duyệt thì hiển thị bình thường trên frontend.

## 🧪 Hướng dẫn test đơn hàng trên Postman

1. Lấy token từ KiotViet qua OAuth2.
2. Tạo đơn hàng mới qua API:

```
POST https://public.kiotapi.com/orders
Headers:
  Authorization: Bearer YOUR_ACCESS_TOKEN
  Content-Type: application/json

Body:
{
  "branchId": 12345,
  "customerId": 67890,
  "orderDetails": [
    {
      "productId": 11111,
      "quantity": 1,
      "price": 150000
    }
  ]
}
```

3. Kiểm tra đơn hàng trong hệ thống quản trị KiotViet của bạn.

## 👤 Tác giả

**Nguyễn Văn Hiệp**  
📘 [Facebook cá nhân](https://www.facebook.com/G.N.S.L.7/)

## ☕ Donate Cà Phê

```
Số TK: 0601 5576 3127  
Họ tên: NGUYEN VAN HIEP  
Ngân hàng: Sacombank  
Chi nhánh: PGD LÊ VĂN QUỚI
```

---

Mọi đóng góp hoặc cải tiến xin gửi qua GitHub hoặc liên hệ trực tiếp Facebook tác giả. Cảm ơn bạn đã sử dụng dự án!

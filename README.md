# Medication App Backend

Backend Node.js + Express + MySQL cho ứng dụng React Native Expo nhắc uống thuốc.

## 1. Chuẩn bị

- Node.js 20 trở lên
- MySQL 8.0.16 trở lên
- Postman

## 2. Tạo database

Mở MySQL Workbench hoặc terminal MySQL, chạy toàn bộ file:

```text
sql/schema.sql
```

Database được tạo tên `medication_app` với 3 bảng:

- `THONGTIN`: hồ sơ người dùng
- `DANHSACHTHUOC`: danh sách thuốc hiện tại
- `DSBAOCAO`: bản ghi lịch sử theo ngày và trạng thái

`DSBAOCAO` lưu bản chụp tên/số lượng/màu/giờ uống. Vì vậy báo cáo cũ vẫn giữ đúng nội dung ngay cả khi thuốc bị sửa hoặc xóa.

## 3. Cài và chạy backend

```bash
npm install
cp .env.example .env
```

Sửa mật khẩu MySQL trong `.env`, sau đó:

```bash
npm run dev
```

Kiểm tra:

```text
GET http://localhost:3000/api/health
```

## 4. API

### Thông tin cá nhân

**POST `/api/thong-tin`**

```json
{
  "hoTen": "Anh Thư",
  "email": "example@gmail.com",
  "soDienThoai": "0123456789",
  "anhDaiDienUrl": "https://example.com/avatar.jpg"
}
```

**GET `/api/thong-tin/1`**

**PUT `/api/thong-tin/1`** dùng body giống POST.

### Danh sách thuốc

**POST `/api/thuoc`**

```json
{
  "thongTinId": 1,
  "tenThuoc": "Aspirin",
  "soLuong": 1,
  "mauThuoc": "Trắng",
  "gioUong": "08:00"
}
```

**GET `/api/thuoc?thongTinId=1`**

**GET `/api/thuoc/1`**

**PUT `/api/thuoc/1`** dùng body giống POST.

**DELETE `/api/thuoc/1`**

### Báo cáo

Tạo một dòng báo cáo cho một thuốc:

**POST `/api/bao-cao`**

```json
{
  "thongTinId": 1,
  "thuocId": 1,
  "ngayUong": "2026-08-04"
}
```

Tạo báo cáo của toàn bộ thuốc đang hoạt động trong một ngày:

**POST `/api/bao-cao/tao-theo-ngay`**

```json
{
  "thongTinId": 1,
  "ngayUong": "2026-08-04"
}
```

Lấy báo cáo:

```text
GET /api/bao-cao?thongTinId=1
GET /api/bao-cao?thongTinId=1&tuNgay=2026-08-01&denNgay=2026-08-31
```

Cập nhật trạng thái:

**PATCH `/api/bao-cao/1/trang-thai`**

```json
{
  "trangThai": "DA_UONG"
}
```

Giá trị hợp lệ:

- `CHUA_UONG`
- `DA_UONG`
- `UONG_TRE`

## 5. Kết nối từ Expo

Không dùng `localhost` trên điện thoại thật. Dùng IP LAN của máy chạy backend, ví dụ:

```ts
const API_BASE_URL = 'http://192.168.1.10:3000/api';
```

Android Emulator thường dùng:

```ts
const API_BASE_URL = 'http://10.0.2.2:3000/api';
```

Ví dụ thêm thuốc:

```ts
await fetch(`${API_BASE_URL}/thuoc`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    thongTinId: 1,
    tenThuoc,
    soLuong: Number(soLuong),
    mauThuoc: mauSac,
    gioUong: gioUong.toLocaleTimeString('vi-VN', {
      hour: '2-digit',
      minute: '2-digit',
      hour12: false,
    }),
  }),
});
```

## 6. Lưu ý kỹ thuật

- Số điện thoại dùng `VARCHAR(10)`, không dùng kiểu số, để không mất số `0` đầu.
- Giờ uống dùng `TIME`, không lưu chuỗi tùy ý.
- URL ảnh chỉ là đường dẫn. Upload file ảnh cần một dịch vụ lưu file riêng.
- Backend hiện chưa có đăng nhập/JWT. Chỉ nên dùng cho prototype hoặc mạng nội bộ trước khi bổ sung xác thực.

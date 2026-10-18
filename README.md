# 🎬 MovieGo — Website đặt vé xem phim

Ứng dụng web đặt vé xem phim full-stack: người dùng xem phim đang chiếu / sắp chiếu, xem trailer, chọn suất chiếu theo rạp, chọn ghế trên sơ đồ và thanh toán; quản trị viên quản lý phim, rạp, phòng, suất chiếu và xem báo cáo doanh thu.

| | |
|---|---|
| **Back-end** | Java 17 · Spring Boot · Spring Security (JWT) · Spring Data JPA / Hibernate · MySQL · Apache POI |
| **Front-end** | React 19 · Vite · React Router · Axios |
| **Triển khai** | Docker · Render (BE) · Vercel (FE) · Aiven MySQL |

---

## ✨ Chức năng

**Người dùng**
- Đăng ký / đăng nhập (JWT)
- Danh sách phim đang chiếu, sắp chiếu; lọc theo thể loại, tìm theo tên
- Chi tiết phim: poster, trailer YouTube, đạo diễn, diễn viên, điểm đánh giá, nhãn độ tuổi
- Lịch chiếu theo ngày, nhóm theo rạp; sơ đồ ghế theo thời gian thực (ghế thường / VIP / ghế đôi, ghế đã đặt bị khóa)
- Đặt vé → thanh toán → xem "Vé của tôi", hủy vé (nhả ghế)

**Quản trị viên**
- CRUD phim, rạp, phòng chiếu, suất chiếu; sinh sơ đồ ghế hàng loạt cho phòng
- Chặn tạo suất chiếu trùng giờ trong cùng phòng
- Báo cáo doanh thu theo phim / theo rạp, xuất file Excel

## 🔧 Điểm kỹ thuật

- **Xác thực & phân quyền:** JWT stateless, `ROLE_USER` / `ROLE_ADMIN` ở tầng `SecurityFilterChain`; người dùng chỉ thao tác được trên đơn của chính mình (kiểm tra chủ sở hữu ở tầng service, trả **403**), user luôn lấy từ token thay vì tin tham số client gửi lên.
- **Chống đặt trùng ghế:** kiểm tra ghế đã có vé trước khi đặt + ràng buộc `UNIQUE (showtime_id, seat_id)` ở database làm lớp bảo vệ cuối.
- **Tính giá vé:** giá suất chiếu + phụ thu theo loại ghế (VIP, ghế đôi).
- **Xử lý lỗi tập trung:** `@RestControllerAdvice` trả về một định dạng lỗi thống nhất `{ status, message, fieldErrors, path }`.
- **Đa ngôn ngữ (i18n):** thông báo lỗi và lỗi validation đọc từ `messages_vi` / `messages_en` theo header `Accept-Language`.
- **Truy vấn:** Spring Data derived query, native query động cho tìm kiếm nhiều điều kiện, native query tổng hợp cho báo cáo doanh thu, phân trang `Pageable`.
- **Dữ liệu demo tự động:** khi khởi động, ứng dụng lấy phim thật từ **TMDB API** (poster, trailer, mô tả tiếng Việt, phân loại độ tuổi) và sinh rạp, phòng, sơ đồ ghế, suất chiếu 7 ngày tới, đơn đặt vé mẫu. Seeder chỉ thêm phần còn thiếu nên chạy lại nhiều lần không bị trùng.
- **Cấu hình theo môi trường:** mọi thông tin bí mật (DB, JWT secret, API key, CORS) đọc từ biến môi trường; đóng gói bằng Docker multi-stage.

## 🗂️ Cấu trúc

```
MovieGo/
├── backend/            Spring Boot REST API
│   └── src/main/java/com/kiettran/webbanve/
│       ├── controller/     REST endpoint
│       ├── service/        nghiệp vụ (interface + impl)
│       ├── repository/     Spring Data JPA
│       ├── entity/  dto/  mapper/  enums/
│       ├── auth/  security/  config/
│       ├── exception/      xử lý lỗi tập trung
│       └── seed/           tạo dữ liệu demo (TMDB)
├── frontend/           React + Vite
└── API_SPEC.md         đặc tả API
```

Mô hình dữ liệu: `User` – `Role` (n-n), `Cinema` 1-n `Room` 1-n `Seat`, `Movie` 1-n `Showtime` n-1 `Room`, `Booking` 1-n `Ticket` (mỗi vé gắn 1 ghế của 1 suất chiếu).

## 🚀 Chạy ở máy

**Yêu cầu:** JDK 17, MySQL 8, Node.js 18+

**Back-end**
```bash
cd backend
cp secrets.properties.example secrets.properties   # điền mật khẩu MySQL + TMDB API key
./mvnw spring-boot:run                              # http://localhost:8080
```
Tạo sẵn database rỗng `movie_ticket`; bảng được Hibernate tự tạo. TMDB API key miễn phí tại [themoviedb.org](https://www.themoviedb.org/settings/api).
Đặc tả các endpoint: xem [`API_SPEC.md`](API_SPEC.md).

**Front-end**
```bash
cd frontend
npm install
npm run dev                                         # http://localhost:5173
```

**Tài khoản demo**

| Vai trò | Username | Password |
|---|---|---|
| Admin | `admin` | `admin123` |
| User | `nguyenvanan` | `123456` |

## ☁️ Biến môi trường khi triển khai

| Back-end | Ý nghĩa |
|---|---|
| `DB_URL`, `DB_USERNAME`, `DB_PASSWORD` | kết nối MySQL |
| `JWT_SECRET` | khóa ký JWT |
| `TMDB_API_KEY` | lấy dữ liệu phim demo |
| `CORS_ALLOWED_ORIGINS` | domain front-end, vd `https://*.vercel.app` |

| Front-end | Ý nghĩa |
|---|---|
| `VITE_API_URL` | địa chỉ back-end |

---

Dữ liệu phim được cung cấp bởi [TMDB](https://www.themoviedb.org/). *This product uses the TMDB API but is not endorsed or certified by TMDB.*

# Báo Cáo Bài 2: Quản trị Tường lửa UFW cho Cụm Dịch vụ Multi-port

## 1. Các bước thực hiện cấu hình Tường lửa (UFW)

### Bước 1: Khai báo chính sách mặc định (Default Policies)
Thiết lập chính sách bảo mật mặc định: Từ chối tất cả lưu lượng truy cập đi vào (incoming) và cho phép toàn bộ lưu lượng đi ra (outgoing):

```bash
# Chặn mặc định tất cả kết nối đi vào máy chủ
sudo ufw default deny incoming

# Cho phép tất cả kết nối từ máy chủ đi ra ngoài Internet
sudo ufw default allow outgoing
```

**Kết quả thực thi:**
```text
Default incoming policy changed to 'deny'
(be sure to update your rules accordingly)
Default outgoing policy changed to 'allow'
(be sure to update your rules accordingly)
```

---

### Bước 2: Khai báo các quy tắc cho phép cổng dịch vụ (Allow Rules)
Cấu hình mở các cổng dịch vụ cần thiết phục vụ quản trị và truy cập từ bên ngoài Internet:

```bash
# 1. Cho phép cổng SSH (22/tcp) để duy trì kết nối quản trị từ xa
sudo ufw allow 22/tcp comment 'Allow SSH Administration'

# 2. Cho phép cổng Web HTTP (80/tcp) phục vụ người dùng truy cập web
sudo ufw allow 80/tcp comment 'Allow Web HTTP Server'

# 3. Cho phép cổng ứng dụng Spring Boot (8082/tcp) phục vụ kiểm thử API
sudo ufw allow 8082/tcp comment 'Allow Spring Boot API Test'
```

**Kết quả thực thi:**
```text
Rules updated
Rules updated (v6)
Rules updated
Rules updated (v6)
Rules updated
Rules updated (v6)
```

---

### Bước 3: Bảo vệ và Cô lập cổng Cơ sở dữ liệu MySQL (Port 3306)
- **Yêu cầu bảo mật:** Cổng cơ sở dữ liệu MySQL (`3306/tcp`) **không** được mở cho truy cập công khai từ Internet.
- **Cơ chế thực hiện:** Do đã áp dụng chính sách `default deny incoming` ở Bước 1 và **không** thêm bất kỳ luật `allow` nào cho cổng 3306, nên cổng MySQL sẽ tự động bị tường lửa chặn đối với mọi kết nối từ bên ngoài Internet, trong khi ứng dụng cục bộ (`localhost` / `127.0.0.1`) vẫn kết nối nội bộ bình thường.

---

### Bước 4: Kích hoạt tường lửa UFW
Kích hoạt hệ thống tường lửa và áp dụng toàn bộ các chính sách đã khai báo:

```bash
sudo ufw enable
```

**Kết quả thực thi:**
```text
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

---

## 2. Kiểm tra kết quả cấu hình

### 2.1. Kiểm tra trạng thái tường lửa chi tiết (`sudo ufw status verbose`)

Chạy lệnh kiểm tra cấu hình chi tiết của UFW:

```bash
sudo ufw status verbose
```

**Kết quả đầu ra (Output):**

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere                   # Allow SSH Administration
80/tcp                     ALLOW IN    Anywhere                   # Allow Web HTTP Server
8082/tcp                   ALLOW IN    Anywhere                   # Allow Spring Boot API Test
22/tcp (v6)                ALLOW IN    Anywhere (v6)              # Allow SSH Administration
80/tcp (v6)                ALLOW IN    Anywhere (v6)              # Allow Web HTTP Server
8082/tcp (v6)              ALLOW IN    Anywhere (v6)              # Allow Spring Boot API Test
```

---

### 2.2. Kiểm tra danh sách quy tắc theo số thứ tự (`sudo ufw status numbered`)

```bash
sudo ufw status numbered
```

**Kết quả đầu ra (Output):**

```text
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere                   # Allow SSH Administration
[ 2] 80/tcp                     ALLOW IN    Anywhere                   # Allow Web HTTP Server
[ 3] 8082/tcp                   ALLOW IN    Anywhere                   # Allow Spring Boot API Test
[ 4] 22/tcp (v6)                ALLOW IN    Anywhere (v6)              # Allow SSH Administration
[ 5] 80/tcp (v6)                ALLOW IN    Anywhere (v6)              # Allow Web HTTP Server
[ 6] 8082/tcp (v6)              ALLOW IN    Anywhere (v6)              # Allow Spring Boot API Test
```

---

### 2.3. Đánh giá và xác nhận an ninh mạng
1. **Trạng thái tường lửa:** `Status: active` — Tường lửa hoạt động ổn định và tự động bật cùng hệ thống.
2. **Các cổng công khai:**
   - Cổng `22/tcp` (SSH): Cho phép quản trị viên kết nối từ mọi nơi.
   - Cổng `80/tcp` (Web HTTP): Cho phép người dùng truy cập trang web.
   - Cổng `8082/tcp` (Spring Boot Backend): Cho phép kiểm thử trực tiếp backend qua mạng.
3. **Cổng bảo mật MySQL (3306/tcp):** Hoàn toàn vắng mặt trong danh sách `ALLOW IN`, đảm bảo không bị tấn công dò mật khẩu (brute-force) hoặc quét lỗ hổng từ Internet.

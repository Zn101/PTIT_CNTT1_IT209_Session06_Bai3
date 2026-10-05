# Bài 3: Cấu hình tường lửa UFW và chẩn đoán cổng mạng

**Session 06 – Exercise 3**

## 1. Mục tiêu

- Dùng UFW (Uncomplicated Firewall) để bảo vệ máy chủ Cloud VPS.
- Chỉ mở các cổng thực sự cần thiết: SSH (22) và ứng dụng Web (8080/tcp).
- Dùng `ufw status`, `ss`, `curl` để kiểm tra cổng kết nối.

## 2. Các bước thực hiện

### Bước 1 – Thiết lập chính sách mặc định

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Chặn toàn bộ kết nối đi vào, cho phép toàn bộ kết nối đi ra.

### Bước 2 – Mở cổng SSH và cổng ứng dụng Web

```bash
sudo ufw allow 22/tcp
sudo ufw allow 8080/tcp
```

> Phải mở cổng 22 **trước khi** bật tường lửa, nếu không sẽ mất kết nối SSH tới máy chủ.

### Bước 3 – Kích hoạt tường lửa

```bash
sudo ufw enable
```

Nhấn `y` để xác nhận cảnh báo `Command may disrupt existing ssh connections`.

## 3. Kiểm tra

### 3.1. Trạng thái tường lửa

```bash
sudo ufw status verbose
```

Kết quả:

```text
<!-- DÁN KẾT QUẢ THỰC TẾ CỦA LỆNH sudo ufw status verbose VÀO ĐÂY -->
```

<!-- Hoặc chèn ảnh chụp màn hình: ![ufw status verbose](./ufw-status.png) -->

### 3.2. Các cổng đang lắng nghe trên máy chủ

```bash
ss -tlnp
```

Kết quả:

```text
<!-- DÁN KẾT QUẢ THỰC TẾ CỦA LỆNH ss -tlnp VÀO ĐÂY -->
```

### 3.3. Kiểm tra kết nối tới cổng 8080 (tùy chọn)

```bash
curl -I http://localhost:8080
```

```text
<!-- DÁN KẾT QUẢ VÀO ĐÂY (nếu ứng dụng đã chạy trên cổng 8080) -->
```

## 4. Đối chiếu kết quả mong đợi

| Yêu cầu | Kết quả |
| --- | --- |
| `Status: active` | ☐ |
| Default: `deny (incoming)`, `allow (outgoing)` | ☐ |
| `22/tcp` – `ALLOW IN` – `Anywhere` | ☐ |
| `8080/tcp` – `ALLOW IN` – `Anywhere` | ☐ |

## 5. Giải thích

- **`ufw status verbose`** cho biết tường lửa *cho phép* cổng nào đi qua.
- **`ss -tlnp`** cho biết tiến trình nào *đang thực sự lắng nghe* trên cổng nào (`-t` TCP, `-l` listening, `-n` hiển thị số cổng, `-p` hiển thị tiến trình).
- Hai lệnh bổ sung cho nhau: một cổng chỉ truy cập được từ bên ngoài khi **vừa** được UFW cho phép **vừa** có tiến trình lắng nghe. Cổng 8080 được mở trên UFW nhưng nếu ứng dụng chưa chạy thì `ss` sẽ chưa hiển thị cổng này và `curl` sẽ báo `Connection refused`.
- Nếu VPS có thêm Cloud Firewall của nhà cung cấp (ví dụ DigitalOcean), cũng cần mở cổng 8080 ở lớp đó.

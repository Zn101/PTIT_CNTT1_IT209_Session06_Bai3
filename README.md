# Bài 4: Quản lý tiến trình nền với nohup và tín hiệu Kill

**Session 06 – Exercise 4**

## 1. Mục tiêu

- Chạy tiến trình nền độc lập với phiên Terminal bằng `nohup` và `&`.
- Giám sát tiến trình bằng `ps`, `pgrep`, `top`/`htop`.
- Tắt tiến trình an toàn bằng `kill` với tín hiệu SIGTERM (15), chỉ dùng SIGKILL (9) khi cần.

## 2. Các bước thực hiện

### Bước 1 – Tạo script `loop-monitor.sh`

```bash
cat << 'EOF' > loop-monitor.sh
#!/bin/bash
while true; do
    echo "System time: $(date)" >> /tmp/monitor.log
    sleep 5
done
EOF
```

Mã nguồn: [loop-monitor.sh](./loop-monitor.sh)

### Bước 2 – Gán quyền thực thi

```bash
chmod +x loop-monitor.sh
```

### Bước 3 – Khởi chạy nền bằng nohup

```bash
nohup ./loop-monitor.sh > /dev/null 2>&1 &
```

Kết quả (số job và PID do shell in ra):

```text
<!-- DÁN KẾT QUẢ THỰC TẾ VÀO ĐÂY, ví dụ: [1] 12345 -->
```

### Bước 4 – Tìm PID của tiến trình

```bash
pgrep -f loop-monitor.sh
ps aux | grep loop-monitor.sh
```

Kết quả:

```text
<!-- DÁN KẾT QUẢ THỰC TẾ VÀO ĐÂY -->
```

**PID của tiến trình:** `<PID>`

### Bước 5 – Kiểm tra log đang được ghi

```bash
tail -n 10 /tmp/monitor.log
```

Kết quả:

```text
<!-- DÁN KẾT QUẢ THỰC TẾ VÀO ĐÂY -->
```

### Bước 6 – Kiểm tra tiến trình vẫn chạy sau khi ngắt SSH

Thoát phiên SSH (`exit`), đăng nhập lại rồi chạy:

```bash
pgrep -f loop-monitor.sh
tail -n 10 /tmp/monitor.log
```

Kết quả:

```text
<!-- DÁN KẾT QUẢ THỰC TẾ VÀO ĐÂY -->
```

### Bước 7 – Tắt tiến trình bằng SIGTERM

```bash
kill -15 <PID>
```

Kiểm tra lại:

```bash
ps aux | grep loop-monitor.sh
```

Kết quả:

```text
<!-- DÁN KẾT QUẢ THỰC TẾ VÀO ĐÂY -->
```

Chỉ khi tiến trình vẫn còn sau SIGTERM mới dùng:

```bash
kill -9 <PID>
```

<!-- Ghi rõ: có phải dùng kill -9 hay không -->

## 3. Đối chiếu kết quả mong đợi

| Yêu cầu | Kết quả |
| --- | --- |
| Script có quyền thực thi và chạy nền bằng `nohup` | ☐ |
| `/tmp/monitor.log` được ghi thêm một dòng mỗi 5 giây | ☐ |
| Tiến trình vẫn chạy sau khi tắt Terminal và đăng nhập lại | ☐ |
| Sau `kill -15`, tiến trình biến mất khỏi `ps aux` | ☐ |

## 4. Giải thích

- **`nohup`**: khi phiên SSH đóng, shell gửi tín hiệu SIGHUP tới các tiến trình con. `nohup` làm tiến trình bỏ qua SIGHUP nên script tiếp tục chạy.
- **`&`**: đưa tiến trình xuống nền, trả lại dấu nhắc lệnh ngay.
- **`> /dev/null 2>&1`**: bỏ cả stdout và stderr, tránh việc `nohup` tạo tệp `nohup.out`. Script tự ghi log bằng `>> /tmp/monitor.log`.
- **`pgrep -f`**: tìm PID theo toàn bộ dòng lệnh, vì tên tiến trình thực tế là `bash`.
- **`ps aux | grep loop-monitor.sh`** luôn hiển thị thêm một dòng của chính lệnh `grep`. Dòng đó không phải là script.
- **SIGTERM (15)**: yêu cầu tiến trình tự kết thúc, tiến trình có thể bắt tín hiệu để dọn dẹp trước khi thoát.
- **SIGKILL (9)**: kernel dừng tiến trình ngay lập tức, tiến trình không thể bắt hay bỏ qua, không kịp dọn dẹp. Vì vậy chỉ dùng khi SIGTERM không có tác dụng.

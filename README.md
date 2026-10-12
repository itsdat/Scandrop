# Scandrop

Send files between phones with just a camera: no Wi-Fi, Bluetooth, or mobile data.
Animated QR codes with fountain coding, runs offline as a single HTML file (PWA).

Gửi file giữa hai điện thoại chỉ bằng camera, không cần Wi-Fi, Bluetooth hay mạng di động.

## Cách hoạt động

**Người gửi:** file → (nén ảnh) → nén deflate → cắt thành các khối → mỗi khung là một mã QR (byte mode, không dùng base64) → phát lặp như video.

**Người nhận:** camera quét liên tục → giải mã bằng [jsQR](https://github.com/cozmo/jsQR) → ghép khối → kiểm tra CRC32 → hiện ảnh/video và nút tải về.

Dùng **fountain code** (XOR ngẫu nhiên có seed): các khung đầu là khối gốc, các khung sau là tổ hợp XOR của vài khối. Quét lỡ khung nào cũng không sao, chỉ cần quét đủ số khung, không phải đợi đúng khung bị thiếu.

Người nhận không cần mạng lúc quét, vì mọi thứ giải mã chạy trong trình duyệt. Chỉ cần trang đã được mở và cache một lần trước đó.

## Sử dụng

1. Mở trang trên cả hai điện thoại **một lần khi có mạng**, rồi thêm vào màn hình chính.
2. Máy gửi: tab **Gửi** → chọn file → **Bắt đầu phát**.
3. Máy nhận: tab **Nhận** → **Bật camera quét** → hướng vào màn hình máy gửi, giữ cố định và lấy nét rõ.

Nếu quét lỗi hoặc chậm: giảm số khung/giây, giảm kích thước khối xuống 200, tăng độ sáng màn hình gửi.

## Hiệu năng

Tốc độ thực tế chỉ vài KB/s, nên dùng cho file nhỏ.

| File | Thời gian ước tính |
|---|---|
| Ảnh đã nén 100–300KB | 15–60 giây |
| Video vài MB | vài phút |
| Video lớn (>10MB) | không thực tế |

Ảnh được tự động resize và nén JPEG trước khi gửi (mặc định cạnh dài 800px, chất lượng 60%).

## Triển khai (GitHub Pages)

Camera (`getUserMedia`) cần HTTPS, nên không chạy khi mở file trực tiếp.

1. Đặt `index.html` và `sw.js` cùng thư mục gốc của repo.
2. **Settings → Pages → Deploy from a branch** → `main` / `/ (root)`.
3. Truy cập `https://<user>.github.io/<repo>/`.

Khi sửa `index.html`, đổi tên cache trong `sw.js` (`qr-transfer-v1` → `v2`), nếu không máy đã cache sẽ dùng bản cũ.

## Giới hạn

- App camera mặc định của điện thoại chỉ quét được từng mã QR một, không ghép được nhiều khung. Người nhận phải dùng trang này.
- Nén dữ liệu dùng `CompressionStream` / `DecompressionStream` (Chrome 80+, Safari 16.4+). Nếu trình duyệt không hỗ trợ, file vẫn gửi được nhưng không được nén.
- Chưa có mã hóa. Ai nhìn thấy màn hình máy gửi đều có thể ghi lại nội dung.

## Cấu trúc khung

Mỗi khung gồm header 15 byte + dữ liệu:

| Byte | Nội dung |
|---|---|
| 0 | cờ (bit 0: đã nén deflate) |
| 1–2 | mã phiên |
| 3–6 | tổng kích thước |
| 7–10 | CRC32 toàn bộ dữ liệu |
| 11–14 | chỉ số khung |

Dữ liệu sau khi ghép: `[2 byte độ dài meta][meta JSON {n: tên, t: mime}][nội dung file]`.

## Thư viện

- [qrcode-generator](https://github.com/kazuhikoarase/qrcode-generator) (MIT): tạo mã QR
- [jsQR](https://github.com/cozmo/jsQR) (Apache-2.0): đọc mã QR

Cả hai được nhúng thẳng vào `index.html`, không gọi CDN.

## License

MIT

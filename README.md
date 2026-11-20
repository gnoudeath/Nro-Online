# Nro-Online (server)

## Chạy server
1. Cài **Java 17** và **Maven**, bật MySQL (XAMPP), import file `Sql/Sql Nro.sql`.
2. Sửa `config/server.properties` (tên database, user, mật khẩu, địa chỉ máy chủ).
3. Build: `mvn package` (hoặc dùng nút Run trong VS Code). Chạy: `run.cmd`
   (`java -jar target/VIP-1.0-RELEASE-jar-with-dependencies.jar`).

## Ghi chú
- Thư mục `target/` KHÔNG nằm trong Git (sinh ra khi build) – clone về phải build một lần.
- Thư mục `log/` phải tồn tại (server ghi log vào đó) – đã có sẵn `log/.gitkeep`, đừng xoá.
- `resources/normal/image/1..4` là ảnh cho các mức zoom của client (mobile/Java/PC) – cần giữ cả 4.
- Clone nhanh hơn: `git clone --depth 1 <url>`

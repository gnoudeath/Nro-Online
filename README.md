# Nro-Online (server game)

Hướng dẫn chạy server trên máy cá nhân (Windows) bằng **VS Code**.

## 1. Cài đặt một lần
| Cần có | Ghi chú |
|---|---|
| **JDK 17** | Dự án biên dịch bằng Java 17 (`pom.xml`). Kiểm tra: `java -version` |
| **Maven** | Kiểm tra: `mvn -v` (hoặc để extension VS Code tự dùng Maven) |
| **XAMPP (MySQL)** | Bật module **MySQL** |
| **Navicat** (hoặc phpMyAdmin) | Để import database |
| **VS Code** + extension **Extension Pack for Java** | Mở bằng *Extensions* → tìm "Extension Pack for Java" |

## 2. Tạo database (một lần)
1. Bật MySQL trong XAMPP.
2. Trong Navicat: tạo database mới (ví dụ `nro-d`), bảng mã `utf8mb4`.
3. Chuột phải database → **Chạy tệp SQL** → chọn `Sql/Sql Nro.sql`.

## 3. Cấu hình
Mở `config/server.properties` và sửa cho đúng máy của bạn:
```
server.db.ip=localhost
server.db.port=3306
server.db.name=nro-d          # đúng tên database vừa tạo
server.db.us=root
server.db.pw=                 # XAMPP mặc định để trống
server.port=14445
server.sv1=Nro:127.0.0.1:14445:0,0,0   # địa chỉ client dùng để kết nối (127.0.0.1 = cùng máy)
```
> Muốn máy khác trong mạng LAN vào chơi: đổi `127.0.0.1` thành IP LAN của máy chạy server (xem bằng `ipconfig`) và mở cổng 14445 trong Windows Firewall.

## 4. Chạy server trong VS Code
1. Mở VS Code → **File → Open Folder** → chọn thư mục project (có file `pom.xml`).
2. Chờ góc dưới phải VS Code hiện xong "Java: Ready" (lần đầu Maven tải thư viện, vài phút).
3. Mở file `src/main/java/nro/server/ServerManager.java` → bấm **Run** phía trên hàm `main`
   (hoặc phím F5).
4. Nhìn cửa sổ **Terminal / Debug Console**: thấy server báo khởi động và lắng nghe cổng `14445` là thành công.

> Phải mở thư mục project làm thư mục gốc (không mở thư mục `src`), vì server đọc các thư mục `resources/`, `config/`, `data/`, `log/` theo đường dẫn tương đối.

### Cách nhẹ hơn: build ra file .jar rồi chạy bằng Java thuần
Không cần mở IDE khi chạy (đỡ tốn RAM):
```
mvn package -DskipTests
java -Xmx1g -jar target/VIP-1.0-RELEASE-jar-with-dependencies.jar
```
(`run.cmd` cũng làm việc tương tự bước thứ hai.) Thư mục `target/` không nằm trong Git nên phải build một lần sau khi clone.

## 5. Kết nối client
- Trong client, địa chỉ máy chủ phải trùng `server.sv1` (ví dụ `127.0.0.1:14445`).
- Server còn cần **máy chủ đăng nhập (login server) ở cổng 8888** (`Manager.java`). Nếu bạn không có login server riêng, cần dùng bản đăng nhập nội bộ (`LocalLogin`) đã chuẩn bị riêng, nếu không client sẽ kẹt ở màn hình đăng nhập.

## 6. Lỗi thường gặp
| Lỗi | Cách xử lý |
|---|---|
| `Communications link failure` / không kết nối được DB | MySQL chưa bật, hoặc sai `server.db.name`, `server.db.us`, `server.db.pw` |
| `Unknown database` | Chưa tạo database hoặc sai tên |
| `FileNotFoundException: log/...` | Thiếu thư mục `log/` (đã có sẵn `log/.gitkeep`, đừng xoá) |
| `Address already in use` (cổng 14445) | Đã có một server khác đang chạy, tắt nó đi hoặc đổi `server.port` |
| Client không vào được | Sai IP/cổng trong `server.sv1`, hoặc Firewall chặn |
| Khởi động chậm | Bình thường: server đọc toàn bộ ảnh `resources/normal/image/1..4` lúc bật |

## 7. Ghi chú về repo
- `resources/normal/image/1..4` là ảnh cho các mức zoom của client (mobile/Java/PC) – cần giữ cả 4.
- `target/`, `.idea/`, nội dung `log/` đã được đưa vào `.gitignore`.
- Clone nhanh hơn: `git clone --depth 1 https://github.com/gnoudeath/Nro-Online.git`

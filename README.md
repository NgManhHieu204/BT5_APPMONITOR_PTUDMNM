# MÔN HỌC: PHÁT TRIỂN ỨNG DỤNG VỚI MÃ NGUỒN MỞ
# HỌ VÀ TÊN: NGUYỄN MẠNH HIẾU - MSSV: K225480106020 - LỚP: K58KTP
_____
## BÀI TẬP 5: APP MONITOR + ALERT DATA REALTIME

## PHẦN 1: LÝ THUYẾT

### 1. Docker là gì?

**Docker** là một nền tảng mã nguồn mở cho phép người dùng tự động hóa việc triển khai, mở rộng và quản lý các ứng dụng dưới dạng container độc lập và có thể thực thi ở bắt kì đâu. Container đóng gói mã nguồn ứng dụng cùng với mọi thư viện, tệp cấu hình và các phụ thuộc cần thiết để container chạy trơn tru, giúp giải quyết triệt để vấn đề "code chạy được trên máy này nhưng lỗi trên máy khác hoặc máy chủ".

### 2. Các từ khóa quan trọng trong file docker-compose.yml

File `docker-compose.yml` sử dụng định dạng YAML để định nghĩa và chạy các ứng dụng Docker gồm nhiều container (multi-container).

* **services**: Định nghĩa danh sách các ứng dụng (container) sẽ được chạy trong hệ thống. (VD: Khai báo service `mariadb` hoặc `grafana`).

* **image**: Chỉ định tên của bản image Docker sẽ được kéo (pull) từ Docker Hub về để chạy container. (VD: `image: mariadb:latest`).

* **ports**: Ánh xạ cổng mạng từ máy chủ gốc (host) vào cổng của container bên trong. (VD: `ports: - "3306:3306"` - Cổng 3306 của máy host trỏ vào cổng 3306 của container).

* **environment**: Thiết lập các biến môi trường cấu hình cho container. (VD: Truyền mật khẩu cho database qua `MYSQL_ROOT_PASSWORD: root_password`).

* **volumes**: Gắn kết một thư mục trên máy host hoặc ổ đĩa ảo của Docker vào container để lưu trữ dữ liệu vĩnh viễn (persistent data), tránh mất dữ liệu khi container bị xóa. (Ví dụ: `volumes: - db_data:/var/lib/mysql`).

* **networks**: Định nghĩa một mạng nội bộ ảo để các container trong cùng một file compose có thể giao tiếp với nhau thông qua tên service thay vì dùng địa chỉ IP.

### 3. Ưu điểm khi triển khai ứng dụng sử dụng Docker

* **Tính nhất quán (Consistency):** Đảm bảo môi trường chạy ứng dụng đồng nhất từ khâu code (Dev) -> kiểm thử (Test) -> vận hành thực tế (Production).

* **Tiết kiệm tài nguyên:** Khác với máy ảo (VM) truyền thống phải dùng cả một hệ điều hành nặng nề, Docker container chia sẻ chung nhân hệ điều hành (Kernel) của máy host nên cực kỳ nhẹ, khởi động trong vài giây và tốn ít RAM/CPU.

* **Tính cô lập (Isolation):** Mỗi container chạy độc lập, nếu một container bị lỗi hoặc bị tấn công thì không làm sập các container khác.

* **Dễ dàng mở rộng (Scalability):** Dễ dàng tăng giảm số lượng container để chịu tải ứng dụng chỉ bằng các công cụ như Docker Swarm hoặc Kubernetes.

### 4. Triển khai App Docker trên máy chủ Offline (Không có Internet)

Để đưa một ứng dụng đã chạy test OK trên laptop lên một máy chủ thực tế (Production) không có kết nối mạng, ta thực hiện quy trình Xuất/Nhập (Export/Import) như sau:

* **Bước 1 (Trên Laptop):** Sử dụng lệnh `docker save` để đóng gói toàn bộ các image cần thiết của ứng dụng thành file nén (VD: `.tar`). Lệnh mẫu: `docker save -o myapp_images.tar mariadb:latest grafana:latest node-red:latest`
  
* **Bước 2 (Vận chuyển):** Copy file nén `myapp_images.tar` cùng thư mục chứa file code và `docker-compose.yml` sang USB/Ổ cứng di động và copy vào máy chủ Offline.

* **Bước 3 (Trên Máy chủ Offline):** Sử dụng lệnh `docker load` để giải nén file `.tar` vào kho image nội bộ của máy chủ. Lệnh mẫu: `docker load -i myapp_images.tar`

* **Bước 4 (Khởi chạy):** Di chuyển vào thư mục chứa file compose và chạy lệnh `docker compose up -d` để bật hệ thống lên như bình thường.

## PHẦN 2: THỰC HÀNH

### 1. TRIỂN KHAI CƠ SỞ HẠ TẦNG

* Tạo thư mục chứa: Sử dụng lệnh `mkdir baitap5 && cd baitap5`

  <img width="562" height="58" alt="image" src="https://github.com/user-attachments/assets/9421e944-2dd3-4d12-9c25-c593afec47b5" />

* Tạo file doker-compose.yml: Sử dụng lệnh `nano docker-compose.yml`, khi cửa sổ nano hiện lên thêm nội dung và bấm `Ctrl + O` để lưu và `Ctrl + X` để thoát

  <img width="1416" height="797" alt="Screenshot 2026-06-06 211617" src="https://github.com/user-attachments/assets/083c1a71-fdb6-4dd8-85f8-70c53d3f9493" />

* Sử dụng lệnh `docker compose up -d` để khởi chạy

  <img width="1415" height="447" alt="image" src="https://github.com/user-attachments/assets/298434c4-8cef-45ec-aa18-6312f196f41c" />

### 2. CHUẨN BỊ MÔI TRƯỜNG VÀ CẤU HÌNH BAN ĐẦU

* Lấy API key Thời tiết (OpenWeatherMap): Truy cập vào trang `https://openweathermap.org/` và tạo một tài khoản miễn phí. Sau khi tạo xong ở góc trên bên phải tài khoản -> Chọn My API keys. Lúc này sẽ thấy một chuỗi mã ký tự dài (Key) -> bấm Generate để tạo.

  <img width="1918" height="1020" alt="Screenshot 2026-06-06 213949" src="https://github.com/user-attachments/assets/8ba4ae92-3726-4f8a-ab2d-d73ee99c03f1" />

* Tạo bảng lưu trữ dữ liệu tức thời trong MariaDB: Sử dụng câu lệnh `docker exec` để gọi trực tiếp vào container MariaDB và thực thi mã SQL tạo bảng mà không cần thông qua giao diện phpMyAdmin. Cấu trúc bảng `weather_realtime` được khởi tạo như sau:

  | Tên cột | Kiểu dữ liệu | Ràng buộc / Ý nghĩa |
  | :--- | :--- | :--- |
  | **`id`** | INT | Khóa chính (Primary Key), Tự động tăng (AUTO_INCREMENT) |
  | **`nhiet_do`** | FLOAT | Lưu giá trị nhiệt độ tức thời (dạng số thập phân) |
  | **`do_am`** | FLOAT | Lưu giá trị độ ẩm tức thời (dạng số thập phân) |
  | **`thoi_gian`** | TIMESTAMP | Tự động cập nhật thời gian thực tế mỗi khi có dữ liệu mới ghi vào |

* Cài đặt thư viện mở rộng cho Node-RED: Truy cập vào địa chỉ `http://192.168.153.130:1880` khi màn hình Node-RED hiện ra bấm vào Menu -> Chọn Manage palette và chuyển sang tab Install. Lần lượt gõ tìm kiếm và Install 3 gói thư viện `node-red-node-mysql` (để ghi dữ liệu vào MariaDB); `node-red-contrib-influxdb` (để ghi dữ liệu vào InfluxDB); `node-red-contrib-telegrambot` (để gửi cảnh báo qua Telegram)

  <img width="1918" height="1023" alt="image" src="https://github.com/user-attachments/assets/bbeba780-1df6-4b04-b233-06359f9a96d7" />

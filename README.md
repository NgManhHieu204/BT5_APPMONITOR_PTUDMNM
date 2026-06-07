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

### 1. Tổng quan hệ thống

* **Giới thiệu bài toán**

  Dự án xây dựng một hệ thống giám sát và cảnh báo tự động theo thời gian thực (App Monitor + Alert Data Realtime) áp dụng mô hình kiến trúc vi dịch vụ (Microservices) chạy trên nền tảng Docker Container. Đối tượng giám sát thực tế được lựa chọn ở đây là **Nhiệt độ và Độ ẩm thời tiết tại Thái Nguyên**, thu thập trực tiếp thông qua OpenWeatherMap API.

* **Các thành phần cốt lõi trong hệ thống:**

  * **Node-RED:** Đóng vai trò là trung tâm điều phối dữ liệu (ETL Workflow). Thực hiện tác vụ lấy dữ liệu thời tiết liên tục (mỗi phút/lần), phân tích dị thường (nhiệt độ quá cao/độ ẩm quá thấp), ghi đồng thời vào 2 cơ sở dữ liệu và kích hoạt bot Telegram gửi cảnh báo.

  * **MariaDB:** Cơ sở dữ liệu quan hệ (RDBMS) dùng để lưu trữ trạng thái thời tiết tức thời (mới nhất), phục vụ truy vấn nhanh cho giao diện Web.
  
  * **InfluxDB (v2.7):** Cơ sở dữ liệu chuỗi thời gian (Time-series Database) tối ưu cho việc lưu trữ dữ liệu lịch sử liên tục, phục vụ vẽ biểu đồ phân tích xu hướng.

  * **Flask API (Python):** Dịch vụ Backend nội bộ kết nối trực tiếp với MariaDB, cung cấp REST API trả về dữ liệu nhiệt độ/độ ẩm định dạng JSON.

  * **Nginx:** Web Server phân phối giao diện Frontend tĩnh và điều hướng luồng request từ trình duyệt sang Flask API một cách an toàn.

  * **Grafana:** Nền tảng trực quan hóa dữ liệu, kết nối với InfluxDB để vẽ biểu đồ trực quan và nhúng thẳng vào giao diện Frontend qua thẻ Iframe.

  * **Cloudflare Tunnel:** Giải pháp mạng ảo hóa giúp Public các dịch vụ (Grafana, Web, Node-RED) ra Internet toàn cầu qua Sub-domain bảo mật HTTPS mà không cần mở Port NAT trên Router vật lý.

* **Kiến trúc hệ thống**

   ```text
   OpenWeatherMap API
          |
          v
      Node-RED  --------> MariaDB (Tức thời) <--- Flask API (REST API)
          |
          +-------------> InfluxDB (Lịch sử) <--- Grafana (Biểu đồ)
          |
          v
    Telegram Alert
 
    (Nginx Frontend Web sẽ gọi Flask API & nhúng trực tiếp biểu đồ Grafana)
   ```

* **Cấu trúc thư mục Project**

  ```text
  baitap5/
  ├── docker-compose.yml     <- Tệp cấu hình chính điều phối toàn bộ 7 services
  ├── flask_api/             <- Thư mục chứa dịch vụ Backend
  │   ├── Dockerfile         <- Lệnh build Image cho Python Flask
  │   ├── app.py             <- Mã nguồn logic API
  │   └── requirements.txt   <- Danh sách thư viện phụ thuộc
  ├── nginx/                 <- Thư mục cấu hình Web Server Frontend
  │   ├── nginx.conf         <- Tệp cấu hình điều hướng (Reverse Proxy)
  │   └── frontend/
  │       └── index.html     <- Giao diện hiển thị thời tiết & nhúng biểu đồ
  └── [Các thư mục Volume]   <- nodered_data, mariadb_data, influxdb_data... (Tự sinh)
  ```


### 2. Triển khai cơ sở hạ tầng

* Tạo thư mục chứa: Sử dụng lệnh `mkdir baitap5 && cd baitap5`

  <img width="562" height="58" alt="image" src="https://github.com/user-attachments/assets/9421e944-2dd3-4d12-9c25-c593afec47b5" />

* Tạo file doker-compose.yml: Sử dụng lệnh `nano docker-compose.yml`, khi cửa sổ nano hiện lên thêm nội dung và bấm `Ctrl + O` để lưu và `Ctrl + X` để thoát

  <img width="1416" height="797" alt="Screenshot 2026-06-06 211617" src="https://github.com/user-attachments/assets/083c1a71-fdb6-4dd8-85f8-70c53d3f9493" />

* Sử dụng lệnh `docker compose up -d` để khởi chạy

  <img width="1415" height="447" alt="image" src="https://github.com/user-attachments/assets/298434c4-8cef-45ec-aa18-6312f196f41c" />

### 3. Chuẩn bị môi trường và cấu hình ban đầu

* Lấy API key Thời tiết (OpenWeatherMap): Truy cập vào trang `https://openweathermap.org/` và tạo một tài khoản miễn phí. Sau khi tạo xong ở góc trên bên phải tài khoản -> Chọn My API keys. Lúc này sẽ thấy một chuỗi mã ký tự dài (Key) -> bấm Generate để tạo.

  <img width="1918" height="1020" alt="Screenshot 2026-06-06 213949" src="https://github.com/user-attachments/assets/8ba4ae92-3726-4f8a-ab2d-d73ee99c03f1" />

* Khởi tạo Telegram Bot và nhóm cảnh báo

  * Lấy Bot Token: Mở Telegram, tìm kiếm @BotFather, gõ lệnh /newbot tạo một con bot mới và lưu lại API Token mà BotFather cấp
 
    <img width="543" height="653" alt="image" src="https://github.com/user-attachments/assets/a7681584-b6b2-416f-9d58-ead5702fe268" />

  * Tạo Nhóm (Group) và Lấy Chat ID: Tạo một Group Telegram -> Add 1 người bạn, Add con Bot vừa tạo và Add tài khoản có ID 1875746636. Invite Link của nhóm -> `https://t.me/+Y4xcPNnDo_UyMzFl`. Lấy Chat ID của nhóm bằng cách chat /test vào nhóm sau đó mở trình duyệt web và truy cập vào đường link sau :
`https://api.telegram.org/bot<TOKEN>/getUpdates` (thay <TOKEN> bằng Token của bot)
 
    <img width="1918" height="1026" alt="image" src="https://github.com/user-attachments/assets/2639fc55-0b85-46a2-bc5b-39036d6f362b" />    

* Tạo bảng lưu trữ dữ liệu tức thời trong MariaDB: Sử dụng câu lệnh `docker exec` để gọi trực tiếp vào container MariaDB và thực thi mã SQL tạo bảng mà không cần thông qua giao diện phpMyAdmin. Cấu trúc bảng `weather_realtime` được khởi tạo như sau:

  | Tên cột | Kiểu dữ liệu | Ràng buộc / Ý nghĩa |
  | :--- | :--- | :--- |
  | **`id`** | INT | Khóa chính (Primary Key), Tự động tăng (AUTO_INCREMENT) |
  | **`nhiet_do`** | FLOAT | Lưu giá trị nhiệt độ tức thời (dạng số thập phân) |
  | **`do_am`** | FLOAT | Lưu giá trị độ ẩm tức thời (dạng số thập phân) |
  | **`thoi_gian`** | TIMESTAMP | Tự động cập nhật thời gian thực tế mỗi khi có dữ liệu mới ghi vào |

* Cài đặt thư viện mở rộng cho Node-RED: Truy cập vào địa chỉ `http://192.168.153.130:1880` khi màn hình Node-RED hiện ra bấm vào Menu -> Chọn Manage palette và chuyển sang tab Install. Lần lượt gõ tìm kiếm và Install 3 gói thư viện `node-red-node-mysql` (để ghi dữ liệu vào MariaDB); `node-red-contrib-influxdb` (để ghi dữ liệu vào InfluxDB); `node-red-contrib-telegrambot` (để gửi cảnh báo qua Telegram)

  <img width="1918" height="1023" alt="image" src="https://github.com/user-attachments/assets/bbeba780-1df6-4b04-b233-06359f9a96d7" />

* Cấu hình Cloudflare Tunnel:

  * Lấy Tunnel Token: Đăng nhập Cloudflare Zero Trust -> Networks -> Tunnels và tạo 1 Tunnels mới (monitor-tunnel) và lưu chuỗi Tunnel Token (nằm sau chữ --token)
 
    <img width="1918" height="1021" alt="image" src="https://github.com/user-attachments/assets/253f7518-e6b2-4900-af24-362597e66c43" />

  * Cấu hình lại file docker-compose.yml và khởi chạy lại bằng lệnh `docker compose up -d`
 
    <img width="1468" height="751" alt="image" src="https://github.com/user-attachments/assets/7b73fda7-5118-4173-bb65-c7d2b190c656" />

    <img width="1470" height="238" alt="image" src="https://github.com/user-attachments/assets/bc038304-f901-4b45-b327-463d3a86e2e7" />

  * Gắn tên miền (Public Hostnames): quay lại web Cloudflare, vào mục Public Hostname của Tunnel đó và Add 2 Route: `Sub-domain 1: nodered -> Domain: nguyenmanhhieu.id.vn -> Service URL: http://nodered:1880`, `Sub-domain 2: grafana -> Domain: nguyenmanhhieu.id.vn -> Service URL: http://grafana:3000`
 
    <img width="1918" height="1025" alt="image" src="https://github.com/user-attachments/assets/459468a2-5c61-4c0a-9a33-7aef511c717e" />

  * Giờ đây có thể truy cập Node-RED bằng `https://nodered.nguyenmanhhieu.id.vn/` và Grafana bằng `https://grafana.nguyenmanhhieu.id.vn/`

### 4. Xây dựng luồng điều phối dữ liệu (Node-RED)

* Khai báo kết nối cơ sở dữ liệu MariaDB: ở cột bên trái kéo một node có tên là `mysql` và cấu hình Host, Port, User, Password, Database. Bấm Add, sau đó bấm Done để đóng bảng lại.

  <img width="1918" height="1017" alt="image" src="https://github.com/user-attachments/assets/f17f782d-f3d3-4160-9851-638dde8272d5" />

* Tạo luồng nhánh 1 **Lấy API -> Lưu Database**: ở cột bên trái và kéo lần lượt các node và cấu hình:

  * Node `inject` (vai trò làm đồng hồ hẹn giờ): đổi tên thành: Hẹn giờ 1 phút, mục Repeat: Chọn interval, cài đặt thời gian: every 1 minutes. Bấm Done
 
    <img width="737" height="806" alt="image" src="https://github.com/user-attachments/assets/cba1d90d-1965-4967-96f4-ffc75b8d7c30" />

  * Node `http request` (vai trò đi lấy dữ liệu thời tiết): mục Method: chọn GET, mục URL: `https://api.openweathermap.org/data/2.5/weather?q=Thai%20Nguyen,VN&appid=API_KEY&units=metric` (thay chữ API_KEY bằng mã API, units=metric là để đổi sang độ C), mục Return: Chọn a parsed JSON object (Để tự giải mã JSON), Name: Gọi API Weather TN. Bấm Done.
 
    <img width="670" height="857" alt="image" src="https://github.com/user-attachments/assets/d1b91f47-cc92-4c2a-92b0-012f2c2b21e6" />

  * Node `function` (vai trò lấy dữ liệu và tạo lệnh SQL): Name: Bóc tách & Tạo lệnh SQL, ô Code thêm Code và Bấm Done
 
    <img width="1135" height="851" alt="image" src="https://github.com/user-attachments/assets/09f5b331-fac7-4ba4-806c-494237e40834" />

  * Thêm dode `debug` nối vào sau Node `mysql` để hiển thị xem MariaDB có báo lỗi gì không. Cuối cùng bấm Deploy để triển khai, ở cột Debug bên phải, câu lệnh INSERT INTO weather_realtime (nhiet_do, do_am) VALUES (35.26, 56) báo hiệu dữ liệu thời tiết thực tế (Nhiệt độ 35.26°C, độ ẩm 56%) đang được đẩy vào bảng thành công cứ mỗi phút một lần
 
    <img width="1918" height="1021" alt="Screenshot 2026-06-07 181346" src="https://github.com/user-attachments/assets/925571b2-81d3-4bb3-bee4-ff219114babb" />

* Tạo luồng nhánh 2 **Lưu dữ liệu vào InfluxDB**: ở cột bên trái và kéo các node và cấu hình:

  * Thêm Node `function` mới đặt tên là `Format cho InfluxDB`, nối dây từ đuôi của Node `Bóc tách & Tạo lệnh SQL` vào và  thêm Code cho node
 
    <img width="842" height="495" alt="image" src="https://github.com/user-attachments/assets/d61f01b7-62b8-4975-8d48-dc564b1c4767" />

  * Node `influxdb out` nối vào đuôi Node Format cho InfluxDB và cấu hình như sau: 
 
    <img width="713" height="566" alt="image" src="https://github.com/user-attachments/assets/0d98c560-8742-4b52-b749-f238434c213e" />

    <img width="667" height="551" alt="image" src="https://github.com/user-attachments/assets/cf82bc32-ce5d-42fd-8d11-310fb781de22" />

* Tạo luồng nhánh 3 **Cảnh báo Telegram (Alert)**: ở cột bên trái và kéo các node và cấu hình:

  * Node `switch` nối từ đuôi Node `Bóc tách & Tạo lệnh SQL` vào và cấu hình:
 
    <img width="661" height="437" alt="image" src="https://github.com/user-attachments/assets/26a8b3a0-3704-4def-9a12-8dc0c4dad563" />

  * Node `function` mới nối từ đuôi Node `switch` vào, dổi tên thành `Soạn tin Telegram` và thêm code:
 
    <img width="832" height="511" alt="image" src="https://github.com/user-attachments/assets/9ccfa5cf-77f9-4a22-803c-dc8646d9df9f" />

  * Node `telegram sender` nối từ đuôi Node `Soạn tin Telegram` vào và cấu hình:
 
    <img width="948" height="821" alt="Screenshot 2026-06-07 184239" src="https://github.com/user-attachments/assets/7d07d04c-e6e2-49c7-948f-ad20097dfd19" />

* Sau khi đã cấu hình đầy đủ các node của 3 nhánh bấm Deploy để hệ thống lưu lại và chính thức chạy vòng lặp mới. Thành quả là bot cứ cách 1 phút gửi thông báo 1 lần nếu nhiệt độ vượt ngưỡng đặt ra

  <img width="1913" height="1043" alt="image" src="https://github.com/user-attachments/assets/fbaeb83c-2005-4a20-bb85-258c1206c47e" />

* Thêm 1 Node `fillter` nối giữa Node `Kiểm tra ngưỡng nhiệt` và `Node Soạn tin Telegram` để chỉ gửi thông báo nhiệt độ 1 lần và khi nhiệt độ thay đổi thì mới gửi lại để tránh bị spam

  <img width="652" height="552" alt="image" src="https://github.com/user-attachments/assets/58044492-3a1c-42cd-93b7-85388f8c662e" />

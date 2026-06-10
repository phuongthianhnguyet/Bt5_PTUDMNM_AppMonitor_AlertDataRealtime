# Bt5_PTUDMNM_AppMonitor_AlertDataRealtime
## YÊU CẦU BÀI TẬP
### A. LÝ THUYẾT
```
+ docker là gì? 
+ các keyword được sử dụng trong docker-compose.yml
  để mô tả 1 service, network, volume,...
  liệt kê + ý nghĩa của từ khoá đó + ví dụ minh hoạ
+ ưu điểm khi triển app sử dụng docker là gì?
+ dùng docker: tạo app, test app OK trên laptop cá nhân
  giờ muốn triển khai app này trên máy chủ thật ko có internet
  thì các bước cần làm là?
```
### B. Thực hành áp dụng: APP MONITOR + ALERT DATA REALTIME
```
sử dụng docker compose có nhiều serivce 
và các thành phần cần thiết để tạo thành ứng dụng:
 + nodered liên tục lấy dữ liệu từ nguồn nào đó (chứng khoán, thời tiết, giá vàng,...)
   nguồn thực tế, số liệu luôn động sau thời gian ngắn
 + nodered lưu trữ dữ liệu vào 2 database: mariadb để lưu giá trị tức thời
   lưu lịch sử vào influxdb
 + sử dụng grafana để trực quan hoá dữ liệu: vẽ biểu đồ
 + sử dụng nginx để làm webserver
   chạy 1 trang web html+js+css làm front-end
   js: lấy dữ liệu tức thời trong mariadb qua (ajax | socket) 
       gọi api (api tự build bằng Flask giống bt1)
       api trả về giá trị tức thời trong mariadb
       hiển thị lên web, auto hiển thị số mới khi thay đổi
   sử dụng iframe để gọi grafana
   hiển thị biểu đồ dữ liệu lịch sử của thông số đã lưu
 + QUAN SÁT DỮ LIỆU LỊCH SỬ => GIÁ TRỊ BẤT THƯỜNG
   (VD MIỀN A..B: OK, DƯỚI A: ALERT LOW, TRÊN B: ALERT HIGH)
 + nodered: kết hợp bot Telegram
   khi dữ liệu not OK, thì gửi tin nhắn từ bot => group trên telegram
   group đã add bot vào: (nhóm đã có 2 người), add thêm 1875746636 thành 3 người
   mỗi khi bot gửi dữ liệu vào nhóm: mọi member of group đều nhận đc
   nội dung alert: tường minh, có value gây alert

 xuất tất cả các container ra file nén.
 xoá mọi container đang chạy
 load lại các container  từ file nén để khôi phục các container đã xoá
```
# BÀI LÀM
### A. lÝ THUYẾT
#### 1. Đocker là gì?
  Docker là một nền tảng mã nguồn mở cho phép đóng gói ứng dụng và tất cả các thành phần phụ thuộc của nó (libraries, dependencies, cấu hình...) vào trong một đơn vị duy nhất gọi là Container.
##### Vấn đề Docker giải quyết
##### Các khái niệm cốt lõi
| Khái niệm            | Định nghĩa                                                                                 | Chức năng                                                                         |
| -------------------- | ------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------- |
| Docker               | Nền tảng mã nguồn mở cho phép đóng gói ứng dụng và các thành phần phụ thuộc vào container. | Đảm bảo ứng dụng hoạt động nhất quán trên nhiều môi trường khác nhau.             |
| Image                | Mẫu đóng gói chứa mã nguồn, thư viện, cấu hình và môi trường thực thi của ứng dụng.        | Là cơ sở để tạo và khởi chạy container.                                           |
| Container            | Môi trường thực thi độc lập được tạo từ Docker Image.                                      | Chạy ứng dụng một cách cô lập với hệ điều hành và các ứng dụng khác.              |
| Dockerfile           | Tệp cấu hình mô tả các bước xây dựng Docker Image.                                         | Tự động hóa quá trình tạo Image từ mã nguồn ứng dụng.                             |
| Docker Compose       | Công cụ quản lý nhiều container thông qua tệp cấu hình `docker-compose.yml`.               | Triển khai và quản lý toàn bộ hệ thống bằng một lệnh duy nhất.                    |
| Volume               | Không gian lưu trữ dữ liệu độc lập với vòng đời của container.                             | Duy trì dữ liệu khi container bị xóa hoặc khởi động lại.                          |
| Network              | Mạng nội bộ được Docker tạo ra để các container giao tiếp với nhau.                        | Cho phép các dịch vụ trong hệ thống kết nối và trao đổi dữ liệu.                  |
| Registry             | Kho lưu trữ Docker Image trên môi trường cục bộ hoặc trực tuyến.                           | Hỗ trợ chia sẻ, phân phối và tái sử dụng Docker Image.                            |
| Service              | Thành phần ứng dụng được định nghĩa trong `docker-compose.yml`.                            | Quản lý việc khởi tạo, cấu hình và vận hành các container.                        |
| Port Mapping         | Cơ chế ánh xạ cổng giữa máy chủ và container.                                              | Cho phép truy cập dịch vụ bên trong container từ bên ngoài.                       |
| Environment Variable | Các biến môi trường được truyền vào container khi khởi động.                               | Cấu hình ứng dụng linh hoạt mà không cần thay đổi mã nguồn.                       |
| Bind Mount           | Cơ chế liên kết thư mục hoặc tệp từ máy chủ vào container.                                 | Đồng bộ dữ liệu giữa máy chủ và container trong quá trình phát triển và vận hành. |

#### 2. Các keyword trong docker-compose.yml
File ``` docker-compose.yml ``` là file cấu hình YAML dùng để mô tả toàn bộ hệ thống multi-container. Cấu trúc tổng quát:
```
version: '3.8'      # phiên bản cú pháp Compose

services:           # định nghĩa các container
  ten_service:
    ...

networks:           # định nghĩa mạng ảo
  ten_network:
    ...

volumes:            # định nghĩa ổ đĩa ảo
  ten_volume:
    ...
```
#### 2.1. Keyworrds cấp cao nhất (top-level)

| Keyword  | Ý nghĩa                                                                            |
| -------- | ---------------------------------------------------------------------------------- |
| version  | Khai báo phiên bản Docker Compose được sử dụng trong tệp cấu hình.                 |
| services | Khai báo các service (container) cần triển khai trong hệ thống.                    |
| networks | Định nghĩa các mạng ảo dùng để kết nối và giao tiếp giữa các service.              |
| volumes  | Định nghĩa các vùng lưu trữ dữ liệu dùng chung hoặc lưu trữ lâu dài cho container. |
#### 2.2. Keywords mô tả Service
#### 2.3. Keywords mô tả Network
```
networks:
  backend_net:
    driver: bridge          # loại mạng (bridge là mặc định)

  monitoring_net:
    driver: bridge
    ipam:                   # cấu hình IP thủ công
      config:
        - subnet: 172.20.0.0/16
```
| Driver | Ý nghĩa |
| :--- | :--- |
| `bridge` | Mạng ảo riêng — các container trong cùng bridge giao tiếp được với nhau qua tên service |
| `host` | Container dùng thẳng network interface của máy host (không cô lập network) |
| `none` | Container không có kết nối mạng |
#### 2.4. Keywords mô tả Volume
```
volumes:
  mariadb_data:       # Docker tự quản lý vị trí lưu trữ
    driver: local

  influxdb_data:
    driver: local

  grafana_data:
    driver: local
```
*** Lưu ý: Named volume được lưu tại /var/lib/docker/volumes/ trên má host. Dữ liệu tồn tại độc lập với vòng đời container.
#### 2.5. Ví dụ đầy đủ.
#### 3. Ưu điểm khi triển khai ứng dụng bằng Docker
| Ưu điểm | Giải thích ngắn gọn |
| :--- | :--- |
| **Nhẹ & Nhanh** | Khởi động trong vài giây, tốn ít RAM/CPU hơn ảo hóa truyền thống (VM) vì dùng chung nhân hệ điều hành của máy host. |
| **Nhất quán (Write Once, Run Anywhere)** | Đóng gói mọi thứ (code, thư viện, cấu hình) vào một Container. Chạy mượt mà từ máy cá nhân lên đến server Production mà không sợ lỗi "vừa chạy được trên máy em cơ mà". |
| **Cô lập an toàn (Isolation)** | Mỗi container là một môi trường độc lập. Lỗi ứng dụng ở container này hoàn toàn không ảnh hưởng đến container khác hoặc máy host. |
| **Quản lý phiên bản (Version Control)** | Docker Image có cơ chế lưu vết theo từng layer (tương tự Git). Dễ dàng rollback (quay lại) phiên bản cũ nếu phiên bản mới gặp lỗi. |
| **Hệ sinh thái lớn (Docker Hub)** | Sở hữu kho lưu trữ khổng lồ chứa hàng triệu image có sẵn (MySQL, Node.js, Nginx...), chỉ cần kéo về (pull) là dùng ngay, tiết kiệm thời gian cài đặt. |
| **Dễ dàng mở rộng (Scalability)** | Rất linh hoạt trong việc tăng/giảm số lượng container để đáp ứng lượng traffic, phối hợp hoàn hảo với các công cụ điều phối như Kubernetes. |
#### 4. Triển khai ứng dụng lên máy chủ KHÔNG có internet.
### B. Thực hành: 
#### 1. TỔNG QUAN.
#### 1.1. Giới thiệu bài toán.
##### Các thành phần cốt lõi của hệ thống:
1. Node-RED
2. MariaDB:
3. InfluxDB(v1.8):
4. Flask API (Python):
5. Nginx:
6. Grafana:
#### 1.2. Kiến trúc hệ thống:
#### 1.3. Danh sách servire và các cổng.
#### 2. Cấu trúc thư mục project.
##### Chuẩn bị môi trường.
#### 3. Xây dựng và cấu hình hệ thống.
#### 3.1. Cấu hình Docker Compose.
Kiểm tra trạng thái bằng lệnh
```
docker compose ps
```
<img width="1846" height="729" alt="image" src="https://github.com/user-attachments/assets/1cdba5c0-7f22-470f-bae6-9b1e7e1a6bd7" />
### Bước 4. Cấu hình chi tiết InfluxDB Data Source

Chuyển sang tab API Tokens (ngay bên cạnh)  Bấm + GENERATE API TOKEN Chọn All Access Token Đặt tên bừa (ví dụ nodered) Bấm Save.
Click vào cái Token vừa tạo và bấm Copy to Clipboard để lấy mã Token mới.

<img width="960" height="482" alt="image" src="https://github.com/user-attachments/assets/24848fbc-917f-4894-9ab8-aec12760f6af" />
Tại trang cấu hình hiện ra, bạn điền chuẩn xác các thông số như sau:

Query Language: Chọn Flux (Đây là ngôn ngữ truy vấn bắt buộc cho InfluxDB bản 2.x mà bạn đang dùng).

HTTP:

URL: Điền chính xác http://influxdb:8086 (Do Grafana và InfluxDB chạy chung mạng Docker).

Auth: Giữ nguyên mặc định (không tích thêm gì).

InfluxDB Details: (Kéo xuống dưới cùng để thấy mục này)

Organization: Điền app-monitor

Token: Dán cái mã Token siêu dài của InfluxDB mà bạn vừa copy thành công ở bước trước vào đây.

Default Bucket: Điền weather

Cuối cùng, kéo xuống dưới cùng bấm nút Save & test.

<img width="960" height="482" alt="image" src="https://github.com/user-attachments/assets/24848fbc-917f-4894-9ab8-aec12760f6af" />


<img width="960" height="508" alt="image" src="https://github.com/user-attachments/assets/446393f1-b170-48bc-af8e-998b54fe8695" />

<img width="954" height="493" alt="image" src="https://github.com/user-attachments/assets/0d5a94d8-9bd4-4a04-bfc1-ef10d67e1918" />

### Bước 5. Tạo Dashboard mới.
5.1. Tạo Dashboard mớiỞ menu bên trái, bấm vào biểu tượng Dashboards (Hình 4 ô vuông nhỏ) $\rightarrow$ chọn New $\rightarrow$ New Dashboard.Bấm vào nút + Add visualization.Chọn nguồn dữ liệu là InfluxDB mà bạn vừa cấu hình ở Bước 4.
5.2. Tạo Panel 1: Biểu đồ Nhiệt độ (Temperature)Màn hình thiết kế biểu đồ hiện ra, bạn làm 2 việc: Điền code lấy dữ liệu và Chỉnh giao diện.1. Nhập câu lệnh lấy dữ liệu (Query):Ở khung nhập code phía dưới (ô Query), xóa hết các dòng mẫu đi và dán đoạn code Flux này vào:

<img width="960" height="482" alt="image" src="https://github.com/user-attachments/assets/18528e5e-b45e-4ccb-aaf9-4b278a21fecb" />

<img width="960" height="481" alt="image" src="https://github.com/user-attachments/assets/778cba0c-bbbf-4887-86c3-ab4f3e41f644" />


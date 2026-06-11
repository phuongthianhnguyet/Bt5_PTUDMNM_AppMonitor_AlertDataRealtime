# Bt5_PTUDMNM_AppMonitor_AlertDataRealtime

## YÊU CẦU BÀI TẬP

### A. LÝ THUYẾT

```text
+ Docker là gì?
+ Các keyword được sử dụng trong docker-compose.yml
  để mô tả 1 service, network, volume,...
  liệt kê + ý nghĩa của từ khoá đó + ví dụ minh hoạ
+ Ưu điểm khi triển app sử dụng Docker là gì?
+ Dùng Docker: tạo app, test app OK trên laptop cá nhân
  giờ muốn triển khai app này trên máy chủ thật không có Internet
  thì các bước cần làm là?
```

### B. THỰC HÀNH ÁP DỤNG: APP MONITOR + ALERT DATA REALTIME

```text
Sử dụng Docker Compose có nhiều service
và các thành phần cần thiết để tạo thành ứng dụng:

+ Node-RED liên tục lấy dữ liệu từ nguồn nào đó
  (chứng khoán, thời tiết, giá vàng,...)
  nguồn thực tế, số liệu luôn động sau thời gian ngắn

+ Node-RED lưu trữ dữ liệu vào 2 database:
  - MariaDB để lưu giá trị tức thời
  - InfluxDB để lưu lịch sử

+ Sử dụng Grafana để trực quan hoá dữ liệu:
  - Vẽ biểu đồ dữ liệu lịch sử

+ Sử dụng Nginx làm Web Server:
  - Chạy trang web HTML + CSS + JavaScript

+ JavaScript:
  - Lấy dữ liệu tức thời trong MariaDB qua AJAX hoặc Socket
  - Gọi API (API tự build bằng Flask giống BT1)
  - API trả về giá trị tức thời trong MariaDB
  - Hiển thị lên Web
  - Tự động cập nhật khi dữ liệu thay đổi

+ Sử dụng iframe để gọi Grafana
  hiển thị biểu đồ dữ liệu lịch sử

+ Quan sát dữ liệu lịch sử => Giá trị bất thường
  - Trong miền A..B: OK
  - Dưới A: ALERT LOW
  - Trên B: ALERT HIGH

+ Node-RED kết hợp Telegram Bot
  khi dữ liệu bất thường:
  - Gửi tin nhắn từ Bot vào Group Telegram
  - Group đã add Bot vào
  - Add thêm user ID 1875746636 thành 3 thành viên

+ Nội dung cảnh báo:
  - Rõ ràng
  - Có giá trị gây Alert

+ Xuất tất cả container ra file nén
+ Xóa mọi container đang chạy
+ Load lại container từ file nén để khôi phục hệ thống
```

---

# A. LÝ THUYẾT

## 1. Docker là gì?

Docker là một nền tảng mã nguồn mở cho phép đóng gói ứng dụng và toàn bộ thành phần phụ thuộc của nó (libraries, dependencies, cấu hình...) vào trong một đơn vị duy nhất gọi là **Container**.

### 1.1. Các khái niệm cốt lõi

| Khái niệm            | Định nghĩa                                                                                 | Chức năng                                                                         |
| -------------------- | ------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------- |
| Docker               | Nền tảng mã nguồn mở cho phép đóng gói ứng dụng và các thành phần phụ thuộc vào container. | Đảm bảo ứng dụng hoạt động nhất quán trên nhiều môi trường khác nhau.             |
| Image                | Mẫu đóng gói chứa mã nguồn, thư viện, cấu hình và môi trường thực thi của ứng dụng.        | Là cơ sở để tạo và khởi chạy container.                                           |
| Container            | Môi trường thực thi độc lập được tạo từ Docker Image.                                      | Chạy ứng dụng một cách cô lập với hệ điều hành và các ứng dụng khác.              |
| Dockerfile           | Tệp cấu hình mô tả các bước xây dựng Docker Image.                                         | Tự động hóa quá trình tạo Image từ mã nguồn ứng dụng.                             |
| Docker Compose       | Công cụ quản lý nhiều container thông qua tệp cấu hình `docker-compose.yml`.               | Triển khai và quản lý toàn bộ hệ thống bằng một lệnh duy nhất.                    |
| Volume               | Không gian lưu trữ dữ liệu độc lập với vòng đời container.                                 | Duy trì dữ liệu khi container bị xóa hoặc khởi động lại.                          |
| Network              | Mạng nội bộ được Docker tạo ra để các container giao tiếp với nhau.                        | Cho phép các dịch vụ trong hệ thống kết nối và trao đổi dữ liệu.                  |
| Registry             | Kho lưu trữ Docker Image trên môi trường cục bộ hoặc trực tuyến.                           | Hỗ trợ chia sẻ, phân phối và tái sử dụng Docker Image.                            |
| Service              | Thành phần ứng dụng được định nghĩa trong `docker-compose.yml`.                            | Quản lý việc khởi tạo, cấu hình và vận hành các container.                        |
| Port Mapping         | Cơ chế ánh xạ cổng giữa máy chủ và container.                                              | Cho phép truy cập dịch vụ bên trong container từ bên ngoài.                       |
| Environment Variable | Các biến môi trường được truyền vào container khi khởi động.                               | Cấu hình ứng dụng linh hoạt mà không cần thay đổi mã nguồn.                       |
| Bind Mount           | Cơ chế liên kết thư mục hoặc tệp từ máy chủ vào container.                                 | Đồng bộ dữ liệu giữa máy chủ và container trong quá trình phát triển và vận hành. |

---

## 2. Các Keyword Trong Docker Compose

File `docker-compose.yml` là file cấu hình YAML dùng để mô tả toàn bộ hệ thống Multi-Container.

### 2.1. Cấu trúc tổng quát

```yaml
version: '3.8'

services:
  ten_service:
    ...

networks:
  ten_network:
    ...

volumes:
  ten_volume:
    ...
```

### 2.2. Keywords Cấp Cao Nhất (Top-Level)

| Keyword  | Ý nghĩa                                                                            |
| -------- | ---------------------------------------------------------------------------------- |
| version  | Khai báo phiên bản Docker Compose được sử dụng trong tệp cấu hình.                 |
| services | Khai báo các service (container) cần triển khai trong hệ thống.                    |
| networks | Định nghĩa các mạng ảo dùng để kết nối và giao tiếp giữa các service.              |
| volumes  | Định nghĩa các vùng lưu trữ dữ liệu dùng chung hoặc lưu trữ lâu dài cho container. |

### 2.3. Keywords Mô Tả Network

```yaml
networks:
  backend_net:
    driver: bridge

  monitoring_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

| Driver | Ý nghĩa                                                                                 |
| ------ | --------------------------------------------------------------------------------------- |
| bridge | Mạng ảo riêng, các container trong cùng bridge giao tiếp được với nhau qua tên service. |
| host   | Container dùng trực tiếp network interface của Host.                                    |
| none   | Container không có kết nối mạng.                                                        |

### 2.4. Keywords Mô Tả Volume

```yaml
volumes:
  mariadb_data:
    driver: local

  influxdb_data:
    driver: local

  grafana_data:
    driver: local
```

> **Lưu ý:** Named Volume được lưu tại `/var/lib/docker/volumes/` trên máy Host. Dữ liệu tồn tại độc lập với vòng đời container.

---

## 3. Ưu Điểm Khi Triển Khai Ứng Dụng Bằng Docker

| Ưu điểm           | Giải thích                                                        |
| ----------------- | ----------------------------------------------------------------- |
| Nhẹ & Nhanh       | Khởi động trong vài giây, tốn ít RAM/CPU hơn máy ảo truyền thống. |
| Nhất quán         | Đóng gói toàn bộ môi trường chạy vào Container.                   |
| Cô lập an toàn    | Mỗi container hoạt động độc lập với nhau.                         |
| Quản lý phiên bản | Hỗ trợ rollback nhờ cơ chế layer của Docker Image.                |
| Hệ sinh thái lớn  | Docker Hub cung cấp hàng triệu Image sẵn có.                      |
| Dễ mở rộng        | Dễ dàng scale kết hợp cùng Kubernetes.                            |

---

## 4. Triển Khai Ứng Dụng Lên Máy Chủ Không Có Internet

### Sơ đồ tổng quan

```text
[ LAPTOP CÁ NHÂN ] (Có Internet)
       │
       ├──► 1. Đóng gói Images
       │        docker save
       │        btc_monitor_backup.tar
       │
       └──► 2. Nén Source Code
                tar -czvf
                app_monitor_code.tar.gz

                     │
             USB / LAN Transfer
                     │
                     ▼

[ MÁY CHỦ THẬT ] (Offline)
       │
       ├──► 3. Giải nén Source Code
       ├──► 4. Docker Load Image
       └──► 5. Docker Compose Up
```

---

# B. THỰC HÀNH: APP MONITOR + ALERT DATA REALTIME

## 1. Tổng Quan

### 1.1. Giới Thiệu Bài Toán

Dự án xây dựng hệ thống giám sát và cảnh báo dữ liệu thời gian thực (App Monitor + Alert Data Realtime) theo mô hình Microservices chạy trên Docker Container.

Đối tượng giám sát được lựa chọn là **Thời tiết Thái Nguyên**, thu thập trực tiếp từ OpenWeatherMap API.

### Các thành phần chính

1. Node-RED
2. MariaDB
3. InfluxDB (v1.8)
4. Flask API (Python)
5. Nginx
6. Grafana

### Chuẩn Bị Môi Trường

---

## 2. Xây Dựng Và Cấu Hình Hệ Thống

### Bước 1. Cấu Hình Docker Compose

* Xây dựng file `docker-compose.yml`
* Quản lý toàn bộ hệ thống thông qua Docker Compose
* Khởi động toàn bộ dịch vụ bằng một lệnh

### Bước 2. Cấu Hình File `.env`

* Lưu tài khoản, mật khẩu và các biến môi trường
* Tăng tính bảo mật
* Dễ dàng thay đổi khi triển khai

### Bước 3. Khởi Tạo Cơ Sở Dữ Liệu

* Tạo file `init.sql`
* Tự động tạo Database và Table khi MariaDB khởi động lần đầu

### Bước 4. Xây Dựng Flask API

* Kết nối MariaDB
* Truy vấn dữ liệu mới nhất
* Trả dữ liệu JSON cho Frontend

### Bước 5. Đóng Gói Flask Bằng Docker

* Tạo Dockerfile
* Build thành Docker Image

### Bước 6. Xây Dựng Giao Diện Web

* HTML
* CSS
* JavaScript

Chức năng:

* Hiển thị thời tiết hiện tại
* Hiển thị thời gian cập nhật
* Hiển thị biểu đồ Grafana

### Bước 7. Cấu Hình Nginx

* Làm Web Server
* Reverse Proxy cho Flask API
* Tích hợp Grafana

### Bước 8. Khởi Động Hệ Thống

```bash
docker compose up -d --build
```

Kiểm tra:

```bash
docker ps
```

<img width="1846" height="729" alt="image" src="https://github.com/user-attachments/assets/1cdba5c0-7f22-470f-bae6-9b1e7e1a6bd7" />

---

## 3. Xây Dựng Luồng Xử Lý Trong Node-RED

### Bước 1. Cài Đặt Thư Viện

Truy cập:

```text
http://192.168.126.131:1880
```

Menu → Manage Palette → Install

Cài:

```text
node-red-node-mysql
node-red-contrib-influxdb
node-red-contrib-telegrambot
```

<img width="478" height="292" alt="image" src="https://github.com/user-attachments/assets/323cc131-add7-4ad4-ab58-5d5b22595a1c" />

### Bước 2. Tạo Telegram Bot

Tìm:

```text
@BotFather
```

Các bước:

* `/start`
* `/newbot`
* Đặt Name
* Đặt Username (kết thúc bằng bot)
* Lưu HTTP API Token

<img width="236" height="221" alt="image" src="https://github.com/user-attachments/assets/3cd1f02d-9ab3-4eee-bb04-e15d2b4dc794" />

### Bước 3. Thiết Kế Flow Thu Thập Dữ Liệu

#### Node Inject

<img width="473" height="433" alt="image" src="https://github.com/user-attachments/assets/eba34dc7-8614-4a1e-a44d-5da8361c592c" />

#### Node HTTP Request

<img width="478" height="451" alt="image" src="https://github.com/user-attachments/assets/53e086e1-5b4e-4b38-9a54-0e1bc444e860" />

#### Node Function

<img width="474" height="450" alt="image" src="https://github.com/user-attachments/assets/6bf370d4-bca1-4dca-8d4d-3419e89135a9" />

<img width="476" height="441" alt="image" src="https://github.com/user-attachments/assets/0c344880-fd93-48d6-aa73-8549484c3c62" />

### Bước 4. Cấu Hình Lưu Trữ Dữ Liệu

#### MariaDB

<img width="475" height="450" alt="image" src="https://github.com/user-attachments/assets/58f7bedd-83bd-4b06-8f07-83433c8ce84f" />

#### InfluxDB

<img width="481" height="451" alt="image" src="https://github.com/user-attachments/assets/cfc90cbc-3194-4472-813f-bf1d27ade23f" />

<img width="475" height="439" alt="image" src="https://github.com/user-attachments/assets/8bb2e5ae-ba05-4e65-8241-1d89ca90be20" />

#### Telegram Sender

<img width="286" height="428" alt="image" src="https://github.com/user-attachments/assets/46de3416-a4d1-4cbf-879c-dadaa1999d94" />

### Tạo Group Telegram

* Tạo New Group
* Add Bot
* Add user ID `1875746636`
* Cấp quyền Administrator cho Bot

### Bước 5. Deploy Và Kiểm Tra

Deploy Flow:

<img width="479" height="182" alt="image" src="https://github.com/user-attachments/assets/62a329f4-10cd-477b-b661-c5dc13e16395" />

Kết quả:

<img width="479" height="233" alt="image" src="https://github.com/user-attachments/assets/fee56b36-2174-41c9-92fe-7d0429f1dcbf" />

<img width="479" height="441" alt="image" src="https://github.com/user-attachments/assets/363df7a4-aa45-421c-bc21-63bb4cb2ba61" />

---

## 4. Grafana - Trực Quan Hóa Dữ Liệu

### Bước 1. Cấu Hình Data Source

Connections → Data Sources → Add Data Source

<img width="478" height="290" alt="image" src="https://github.com/user-attachments/assets/95b74d32-8351-4bd0-8613-296c82571f23" />

Cấu hình:

```text
Name: InfluxDB
Query Language: InfluxQL
URL: http://influxdb:8086
Database: app-monitor
```

### Bước 2. Tạo Dashboard

New → New Dashboard → Add Visualization

<img width="479" height="369" alt="image" src="https://github.com/user-attachments/assets/a68cf5c9-df2d-4427-aad2-cea33d629b07" />

### Bước 3. Nhúng Grafana Vào Website

Share → Share Internally → Copy Link

Bỏ chọn:

```text
Lock time range
```

<img width="481" height="431" alt="image" src="https://github.com/user-attachments/assets/6500e72c-678c-420b-a949-28bfbd14214b" />

Kết quả:

<img width="959" height="486" alt="image" src="https://github.com/user-attachments/assets/31310b40-2ec7-4273-b75f-9e5147c00836" />

---

## 5. Backup Và Khôi Phục Hệ Thống Docker

### Bước 1. Xuất Tất Cả Container Ra File Nén

```bash
docker save $(docker compose ps -q | xargs docker inspect --format '{{.Image}}') -o btc_monitor_backup.tar
```

<img width="707" height="203" alt="image" src="https://github.com/user-attachments/assets/f7929648-2723-49a6-a623-bc20822963f2" />

### Bước 2. Xóa Toàn Bộ Container

```bash
docker compose down
```

<img width="706" height="148" alt="image" src="https://github.com/user-attachments/assets/9b9aa38d-0e8a-4ed8-8704-37847b011eae" />

Khi F5 trình duyệt, hệ thống không còn truy cập được:

<img width="479" height="472" alt="image" src="https://github.com/user-attachments/assets/ddbc98e7-cd0b-4a24-99fa-c00dfb4908a5" />

### Bước 3. Load Lại Image Từ File Backup

```bash
docker load -i btc_monitor_backup.tar
```

<img width="579" height="131" alt="image" src="https://github.com/user-attachments/assets/1c98c5d5-5780-4672-805b-ea51a6a4a6c9" />

### Bước 4. Khởi Động Lại Toàn Bộ Hệ Thống

```bash
docker compose up -d
```

<img width="704" height="152" alt="image" src="https://github.com/user-attachments/assets/bff58392-94e9-4d1f-81ef-b5f95a061a06" />

Kết quả sau khi khôi phục:

<img width="470" height="477" alt="image" src="https://github.com/user-attachments/assets/3b91acbe-3663-477a-a68d-3c73dc6ca021" />

---

# KẾT LUẬN

Hệ thống App Monitor + Alert Data Realtime đã được triển khai thành công bằng Docker Compose với các thành phần:

* Node-RED thu thập dữ liệu thời tiết thời gian thực.
* MariaDB lưu dữ liệu tức thời.
* InfluxDB lưu dữ liệu lịch sử.
* Flask API cung cấp dữ liệu cho Frontend.
* Nginx làm Web Server.
* Grafana trực quan hóa dữ liệu.
* Telegram Bot gửi cảnh báo tự động khi phát hiện dữ liệu bất thường.

Ngoài ra, hệ thống hỗ trợ đóng gói, sao lưu và khôi phục hoàn chỉnh bằng Docker Image, giúp triển khai nhanh chóng trên các môi trường có hoặc không có Internet.

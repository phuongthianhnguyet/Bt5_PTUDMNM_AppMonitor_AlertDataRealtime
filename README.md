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
##### 3. Xây dựng và cấu hình hệ thống.
Bước 1. Cấu hình Docker Compose.
- Xây dựng file docker-compose.yml để quản lý toàn bộ các dịch vụ của hệ thống.
- Việc sử dụng Docker Compose giúp khởi động, dừng và quản lý toàn bộ hệ thống chỉ bằng 1 lệnh, đồng thời đảm bảo các container hoạt động trong cùng 1 mạng nội bộ.
Bước 2. Cấu hình biến môi trường .env
- Tạo file .env để lưu các thông tin cấu hình như mật khẩu, tài khoản...
- Việc tách riêng các thông tin giúp tăng tính năng bảo mật và dễ dàng thay đổi khi triển khai trên môi trường khác.
- Bước 3. Khởi tạo cơ sở đữ liệu
- Tạo file init.sql để tự động tạo cơ sở dữ liệu và bảng dữ liệu khi MariaDB khởi động lần đầu.
Bước 4. Xây dựng Flask API
- Flask được sử dụng làm Backend để cung cấp dữ liệu cho giao diện Web.
- API có nhiệm vụ kết nối MariaDB, lấy dữ liệu mới nhất và trả về dưới dạng JSON. Điều này giúp Frontend không cần truy cập trực tiếp vào cơ sở dữ liệu.
Bước 5. Đóng gói Flask bằng Docker
- Tạo file Dockerfile để đóng gói ứng dụng Flask thành Docker Image.
- Nhờ đó Flask có thể được triển khai dễ dàng cùng các dịch vụ khác thông qua Docker Compose.
Bước 6. Xây dựng giao diện Web.
- Giao diện được xây dựng bằng HTML, CSS và JavaScript.
- Trang Web có nhiệm vụ hiển thị thời tiết hiện tại, thời gian cập nhật và biểu đồ lịch sử dữ liệu từ Grafana.
Bước 7. Cấu hình Nginx
- Nginx được sử dụng làm Web Server để phục vụ giao diện người dùng và chuyển tiếp các yêu cầu API đến Flask.
- Ngoài ra, Nginx cũng hỗ trợ tích hợp Grafana vào cùng hệ thống.
Bước 8. Khởi động hệ thống.
- Sau khi hoàn tất cấu hình, sử dụng lệnh: docker compose up -d --build để build và khởi động toàn bộ hệ thống.
- Kiểm tra trạng thái các container bằng lệnh:docker ps
  
<img width="1846" height="729" alt="image" src="https://github.com/user-attachments/assets/1cdba5c0-7f22-470f-bae6-9b1e7e1a6bd7" />

##### 4. Xây dựng luồng xử lý trong Node-Red
###### Bước 1. Truy cập Node-Red và cài đặt thư viện
Truy cập địa chỉ: http://192.168.126.131:1880
Trên giao diện Nod-Red, bấm vào nút Menu -> Chọn Manage palette Chuyển sang tab Install rồi tìm kiếm và cài đặt 3 node sau:
```
- node-red-node-mysql (để kết nối MariaDB)
- node-red-contrib-influxdb (để kết nối InfluxDB)
- node-red-contrib-telegrambot (để tạo bot cảnh báo)
```

<img width="478" height="292" alt="image" src="https://github.com/user-attachments/assets/323cc131-add7-4ad4-ab58-5d5b22595a1c" />

###### Bước 2. Tạo Bot Telegram qua @BotFather để gửi cảnh báo về telegram
Mở Telegram trên máy tính hoặc điện thoại, tìm @BotFather (tài khoản có tích xanh):

Gửi tin nhắn cho @BotFather: /start
Gửi lệnh /newbot để yêu cầu tạo một bot mới.
Đặt tên cho Bot (Name): Nhập tên hiển thị bất kỳ 
Đặt tên người dùng (Username): Nhập tên viết liền, không dấu và bắt buộc phải kết thúc bằng chữ bot (Ví dụ: anh_nguyet_bot). Tên username này phải là duy nhất trên toàn hệ thống Telegram.
Sau khi đặt username thành công, BotFather sẽ gửi một đoạn tin nhắn chúc mừng kèm theo mã HTTP API Token. Coppy đoạn mã Token này lại để sau này nhập vào Node-Red

<img width="236" height="221" alt="image" src="https://github.com/user-attachments/assets/3cd1f02d-9ab3-4eee-bb04-e15d2b4dc794" />

###### Bước 3. Thiết kế và cấu hình luồng Flow để lấy dữ liệu
Trên giao diện nodered, cột bên trái chứa các node, tại đây ta kéo thả các node sau ra màn hình workspace để tạo luồng flow để lấy dữ liệu
- Node inject:
  
<img width="473" height="433" alt="image" src="https://github.com/user-attachments/assets/eba34dc7-8614-4a1e-a44d-5da8361c592c" />

- Node http request:

<img width="478" height="451" alt="image" src="https://github.com/user-attachments/assets/53e086e1-5b4e-4b38-9a54-0e1bc444e860" />

- Node function:

<img width="474" height="450" alt="image" src="https://github.com/user-attachments/assets/6bf370d4-bca1-4ca3-8d4d-3419e89135a9" />

<img width="476" height="441" alt="image" src="https://github.com/user-attachments/assets/0c344880-fd93-48d6-aa73-8549484c3c62" />

###### Bước 4. Cấu hình và lưu trữ đầu ra.
- Lưu trữ giá trị tức thời vào MariaDB:
  - Kéo node mysql ra, nối đầu ra số 1 với node Function.
  - Nháy đúp chuột vào node mysql, bấm edit như hình.
  - Điền đầy đủ bấm App -> Done

<img width="475" height="450" alt="image" src="https://github.com/user-attachments/assets/58f7bedd-83bd-4b06-8f07-83433c8ce84f" />

- Lưu giá trị tức thời vào ÌnluxDB:
  - Kéo node Influxdb out ra, nối với đầu ra số 2 của node Function.
  - Tương tự mysql, bấm edit như hình
  - Điền đầy đủ bấm Add -> Done.
 
<img width="481" height="451" alt="image" src="https://github.com/user-attachments/assets/cfc90cbc-3194-4472-813f-bf1d27ade23f" />

<img width="475" height="439" alt="image" src="https://github.com/user-attachments/assets/8bb2e5ae-ba05-4e65-8241-1d89ca90be20" />

- Gửi cảnh báo Telegram.
  - Kéo node telegram sender ra, nối vào đầu số 3 của node function
  - Nháy đúp vào node, edit như ảnh
  - Token: Paste mã token Bot Telegram vào ( Lấy từ BotFather trên Telegram )
  - nhấn Add -> Done

<img width="286" height="428" alt="image" src="https://github.com/user-attachments/assets/46de3416-a4d1-4cbf-879c-dadaa1999d94" />

- Tạo Group gửi cảnh báo thời tiết qua telegram
  - Mở telegram tạo New Group -> Thêm bot vào group -> Thêm tài khoản có id như yêu cầu bài vào group
  - Cấp quyền cho bot để luôn gửi cảnh báo vào nhóm:
    - Trong Group Telegram đã tạo và add bot -> Chọn Manage -> Administrators -> Administrator và chọn con Bot

###### Bước 5. Deploy và kiểm tra
- Sau khi đã cấu hình xong workflow, bấm nút Deploy màu đỏ ở góc trên cùng bên phải để lưu và chạy toàn bộ cấu hình.
  
<img width="479" height="182" alt="image" src="https://github.com/user-attachments/assets/62a329f4-10cd-477b-b661-c5dc13e16395" />

- Sau khi Deploy, truy cập web để xem kết quả:

<img width="479" height="233" alt="image" src="https://github.com/user-attachments/assets/fee56b36-2174-41c9-92fe-7d0429f1dcbf" />

<img width="479" height="441" alt="image" src="https://github.com/user-attachments/assets/363df7a4-aa45-421c-bc21-63bb4cb2ba61" />

##### 5. Sử dụng Grafana để vẽ biểu đồ dữ liệu lịch sử thời tiết.
###### Bước 1. Truy cập Grafana và cấu hình nguồn dữ liệu (Data Source)
- Đăng nhập bằng tài khoản admin
- Sau khi đăng nhập vào Gradân, ở tab bên trái màn hình chọn Connections -> Data sources -> Add data source

<img width="478" height="290" alt="image" src="https://github.com/user-attachments/assets/95b74d32-8351-4bd0-8613-296c82571f23" />

- Sau khi chọn Add data source -> influxdb và cấu hình nó.
  - Name: Giữ nguyên
  - Query language: InfluxQL
  - URL: http://ìnluxdb:8088 ( Grafana sẽ gọi qua influxdb thông qua mạng nội bộ của docker
  - Database: app-monitor
  - Save & test
 
###### Bước 2. Tạo biểu đồ.
- Nhấn chữ New -> New dashboard -> chọn dấu + lớn bên phải -> Configure visualization
- Sau đó đặt tên cho biểu đồ ở phần title

<img width="479" height="369" alt="image" src="https://github.com/user-attachments/assets/a68cf5c9-df2d-4427-aad2-cea33d629b07" />

###### Bước 3. Lấy link Iframe và nhúng vào website
- Trên góc phải của biểu đồ chọn dấu ba chấm -> Chọn Share -> Chọn share internally -> copy link
 - Bỏ chọn ô Lock time range để dữ liệu không bị khoá mà luôn biến động theo thời gian
   
<img width="481" height="431" alt="image" src="https://github.com/user-attachments/assets/6500e72c-678c-420b-a949-28bfbd14214b" />

- Cho đoạn link copy được vào file index.html và kết quả
- Trên trang web đã hiển thị được thông số thời tiết, cứ 10s sẽ cập nhật dữ liệu mới và hiển thị được cả dạng biểu đồ. 

<img width="959" height="486" alt="image" src="https://github.com/user-attachments/assets/31310b40-2ec7-4273-b75f-9e5147c00836" />

##### 6. Xuất tất cả các containẻ ra file nén
- Bước 1. Xuất tất cả các container ra file nén
  - Để đóng gói toàn bộ môi trường thành file nén.tar, chạy lệnh
    ```
    
#### Bước 4. Cấu hình chi tiết InfluxDB Data Source
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


Dữ liệu của 3 biểu đồ thời tiết đã được cập nhật liên tục.

<img width="958" height="481" alt="image" src="https://github.com/user-attachments/assets/82f74b33-8fb0-4de1-a3e5-0b8c5b0bb9ce" />

#### Cảnh báo 
Ảnh node-red

<img width="960" height="540" alt="image" src="https://github.com/user-attachments/assets/23b070fa-c65d-41fa-a8cb-881d70f04526" />

Ảnh biểu đồ.

<img width="960" height="540" alt="image" src="https://github.com/user-attachments/assets/8deb40a1-dc27-4a12-9653-3aa4e0e8168c" />

ảnh trên web

<img width="960" height="484" alt="image" src="https://github.com/user-attachments/assets/d2fca796-b564-4b9e-8605-cf0d711d0e1b" />

biểu đồ và cảnh báo chung 1 web



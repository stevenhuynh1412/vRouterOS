# Hướng dẫn cấu hình High Availability trong vRouterOS

## Giới thiệu

**High Availability (HA)** trong **vRouterOS** đảm bảo tính sẵn sàng của hệ thống mạng bằng cách sử dụng **VRRP (Virtual Router Redundancy Protocol)** để cung cấp cơ chế dự phòng active/backup cho các router. VRRP cho phép nhiều router chia sẻ một địa chỉ IP ảo, với một router hoạt động như **master** và các router khác là **backup**. Nếu router master gặp sự cố, router backup sẽ tự động tiếp quản địa chỉ IP ảo, đảm bảo liên tục dịch vụ với thời gian gián đoạn tối thiểu.

Tài liệu này hướng dẫn cách:
- Cấu hình VRRP để thiết lập dự phòng.
- Sử dụng các script kiểm tra sức khỏe (health check) và chuyển đổi trạng thái (transition).
- Cấu hình cân bằng tải (load balancing) với Virtual Server.
- Giám sát và kiểm tra trạng thái VRRP.

Các lệnh được thực thi trong **chế độ vận hành** (ký hiệu `$`) hoặc **chế độ cấu hình** (ký hiệu `#`). Các thay đổi cần được **commit** để có hiệu lực và **lưu** để duy trì sau khi khởi động lại.

## Tổng quan về VRRP

- **VRRP (Virtual Router Redundancy Protocol)**:
  - Mỗi router VRRP có một địa chỉ IP/IPv6 vật lý và chia sẻ một địa chỉ IP ảo.
  - Các router bầu chọn master dựa trên mức độ ưu tiên (**priority**). Router có ưu tiên cao nhất trở thành master và gán địa chỉ IP ảo cho giao diện của nó.
  - Các router backup có ưu tiên thấp hơn sẽ tiếp quản nếu master không gửi gói tin keepalive.
  - Gói tin keepalive VRRP sử dụng multicast và bị giới hạn trong cùng một phân đoạn tầng liên kết dữ liệu (datalink layer segment).
- **Nhóm VRRP (VRRP Group)**: Nhiều nhóm VRRP (còn gọi là virtual routers) có thể được cấu hình, mỗi nhóm được xác định bởi một **VRID (Virtual Router Identifier)**.

## Các lệnh cấu hình VRRP

### 1. Cấu hình nhóm VRRP
- **Mô tả**: Tạo một nhóm VRRP với địa chỉ IP ảo, giao diện, và mức độ ưu tiên.
- **Lệnh**:
  ```
  # set high-availability vrrp group <group-name> address <ip-address/prefix> interface <interface>
  # set high-availability vrrp group <group-name> vrid <vrid>
  # set high-availability vrrp group <group-name> priority <value>
  ```
  - `<group-name>`: Tên nhóm VRRP (ví dụ: `LAN_HA`).
  - `<ip-address/prefix>`: Địa chỉ IP ảo (IPv4 hoặc IPv6, không thể trộn lẫn trong cùng nhóm).
  - `<interface>`: Giao diện áp dụng VRRP (ví dụ: `eth1`).
  - `<vrid>`: ID của virtual router (1-255).
  - `<value>`: Mức độ ưu tiên (1-254, mặc định 100, cao hơn là master).
- **Ví dụ**:
  ```
  $ configure
  # set high-availability vrrp group LAN_HA address 192.168.1.1/24 interface eth1
  # set high-availability vrrp group LAN_HA vrid 10
  # set high-availability vrrp group LAN_HA priority 150
  # commit
  # save
  Saving configuration to '/config/config.boot'...
  Done
  ```
  - Tạo nhóm VRRP `LAN_HA` trên `eth1` với địa chỉ IP ảo `192.168.1.1/24`, VRID 10, và ưu tiên 150.

### 2. Cấu hình script kiểm tra sức khỏe (Health Check Script)
- **Mô tả**: Sử dụng script để kiểm tra trạng thái của router. Nếu script trả về mã lỗi khác 0, VRRP sẽ chuyển sang trạng thái FAULT.
- **Lệnh**:
  ```
  # set high-availability vrrp group <group-name> health-check script <path-to-script>
  # set high-availability vrrp group <group-name> health-check interval <seconds>
  ```
  - `<path-to-script>`: Đường dẫn đến script (ví dụ: `/config/scripts/vrrp-check.sh`).
  - `<seconds>`: Khoảng thời gian kiểm tra (ví dụ: 60 giây).
- **Ví dụ**:
  ```
  # set high-availability vrrp group LAN_HA health-check script /config/scripts/vrrp-check.sh
  # set high-availability vrrp group LAN_HA health-check interval 60
  # commit
  # save
  ```
  - Chạy script `/config/scripts/vrrp-check.sh` mỗi 60 giây. Nếu script thất bại, nhóm `LAN_HA` sẽ chuyển sang trạng thái FAULT.
- **Lưu ý**: Không thay đổi cấu hình VRRP trong script kiểm tra để tránh xung đột.

### 3. Cấu hình script chuyển đổi trạng thái (Transition Script)
- **Mô tả**: Chạy script khi trạng thái VRRP thay đổi (master, backup, hoặc fault).
- **Lệnh**:
  ```
  # set high-availability vrrp group <group-name> transition-script master <path-to-script>
  # set high-availability vrrp group <group-name> transition-script backup <path-to-script>
  # set high-availability vrrp group <group-name> transition-script fault <path-to-script>
  ```
- **Ví dụ**:
  ```
  # set high-availability vrrp group LAN_HA transition-script master /config/scripts/vrrp-master.sh
  # set high-availability vrrp group LAN_HA transition-script backup /config/scripts/vrrp-backup.sh
  # commit
  # save
  ```
  - Chạy script `/config/scripts/vrrp-master.sh` khi nhóm `LAN_HA` trở thành master, và `/config/scripts/vrrp-backup.sh` khi trở thành backup.

### 4. Cấu hình Virtual Server (Cân bằng tải)
- **Mô tả**: Cân bằng tải lưu lượng đến một địa chỉ IP ảo giữa nhiều máy chủ thực (real servers).
- **Lệnh**:
  ```
  # set high-availability virtual-server <virtual-ip> real-server <real-ip> health-check script <path-to-script>
  # set high-availability virtual-server <virtual-ip> protocol <protocol> port <port>
  ```
- **Ví dụ**:
  ```
  # set high-availability virtual-server 203.0.113.1 real-server 192.0.2.11 health-check script /config/scripts/check-real-server-first.sh
  # set high-availability virtual-server 203.0.113.1 real-server 192.0.2.12 health-check script /config/scripts/check-real-server-second.sh
  # set high-availability virtual-server 203.0.113.1 protocol tcp
  # set high-availability virtual-server 203.0.113.1 port 8280
  # commit
  # save
  ```
  - Cân bằng tải TCP đến `203.0.113.1:8280` giữa hai máy chủ thực `192.0.2.11:80` và `192.0.2.12:80`. Máy chủ thực sẽ bị loại bỏ nếu kiểm tra sức khỏe thất bại.

### 5. Kiểm tra trạng thái VRRP
- **Mô tả**: Hiển thị trạng thái hiện tại của các nhóm VRRP.
- **Lệnh**:
  ```
  $ show vrrp
  ```
- **Ví dụ**:
  ```
  $ show vrrp
  Name       Interface   VRID   State   Last Transition
  ---------- ----------  ------ ------- ----------------
  LAN_HA     eth1        10     MASTER  2s
  ```
  - Hiển thị nhóm `LAN_HA` trên `eth1` đang là master với VRID 10.

### 6. Xóa cấu hình VRRP
- **Mô tả**: Xóa nhóm VRRP hoặc một phần cấu hình.
- **Lệnh**:
  ```
  # delete high-availability vrrp group <group-name>
  ```
- **Ví dụ**:
  ```
  $ configure
  # delete high-availability vrrp group LAN_HA
  # commit
  # save
  ```
  - Xóa nhóm VRRP `LAN_HA`. Nhóm bị xóa sẽ không tham gia vào quá trình VRRP.

### 7. So sánh và hủy bỏ thay đổi
- **So sánh cấu hình**:
  ```
  # compare
  ```
  - Hiển thị sự khác biệt giữa cấu hình làm việc và cấu hình đang chạy.
  - Ví dụ:
    ```
    # compare
    + high-availability vrrp group LAN_HA address 192.168.1.1/24 interface eth1
    ```
- **Hủy bỏ thay đổi**:
  ```
  # discard
  ```
  - Hủy các thay đổi chưa commit.

## Ví dụ thực tế: Cấu hình VRRP cho dự phòng
1. Vào chế độ cấu hình:
   ```
   $ configure
   ```
2. Cấu hình nhóm VRRP trên router 1 (Master):
   ```
   # set interfaces ethernet eth1 address 192.168.1.2/24
   # set high-availability vrrp group LAN_HA address 192.168.1.1/24 interface eth1
   # set high-availability vrrp group LAN_HA vrid 10
   # set high-availability vrrp group LAN_HA priority 150
   # set high-availability vrrp group LAN_HA health-check script /config/scripts/vrrp-check.sh
   # set high-availability vrrp group LAN_HA health-check interval 60
   ```
3. Cấu hình nhóm VRRP trên router 2 (Backup):
   ```
   # set interfaces ethernet eth1 address 192.168.1.3/24
   # set high-availability vrrp group LAN_HA address 192.168.1.1/24 interface eth1
   # set high-availability vrrp group LAN_HA vrid 10
   # set high-availability vrrp group LAN_HA priority 100
   # set high-availability vrrp group LAN_HA health-check script /config/scripts/vrrp-check.sh
   # set high-availability vrrp group LAN_HA health-check interval 60
   ```
4. Áp dụng và lưu trên cả hai router:
   ```
   # commit
   # save
   Saving configuration to '/config/config.boot'...
   Done
   # exit
   ```
5. Kiểm tra trạng thái:
   ```
   $ show vrrp
   Name       Interface   VRID   State   Last Transition
   ---------- ----------  ------ ------- ----------------
   LAN_HA     eth1        10     MASTER  2s
   ```
   - Trên router 1 (ưu tiên 150) sẽ hiển thị trạng thái MASTER.
   - Trên router 2 (ưu tiên 100) sẽ hiển thị trạng thái BACKUP.

## Lưu ý quan trọng
- **Địa chỉ IP ảo**: Địa chỉ IP ảo phải giống nhau trên tất cả các router trong nhóm VRRP. Không trộn IPv4 và IPv6 trong cùng nhóm (sử dụng tùy chọn `excluded-address` nếu cần).
- **Sao lưu cấu hình**: Luôn sao lưu trước khi thay đổi:
  ```
  $ save /config/backup-20250907.conf
  ```
- **Kiểm tra lỗi**: Nếu `commit` thất bại, đọc thông báo lỗi (ví dụ: VRID trùng lặp, giao diện không tồn tại).
- **Multicast**: Đảm bảo switch hỗ trợ multicast để gửi gói tin keepalive VRRP.
- **Script an toàn**: Không thay đổi cấu hình VRRP trong health-check hoặc transition script để tránh vòng lặp hoặc lỗi.
## Kết luận
Cấu hình High Availability trong vRouterOS sử dụng VRRP cung cấp khả năng dự phòng mạnh mẽ, đảm bảo tính liên tục của mạng. Sử dụng các lệnh `set`, `delete`, `commit`, `show`, và `save` để cấu hình và quản lý VRRP hiệu quả. Thực hành các ví dụ trên để làm quen với quy trình cấu hình và giám sát HA.
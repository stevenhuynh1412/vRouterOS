# Hướng dẫn cấu hình SNMP trong vRouterOS

## Giới thiệu

**Simple Network Management Protocol (SNMP)** trong **vRouterOS** cho phép giám sát và quản lý thiết bị mạng từ các hệ thống quản lý mạng (NMS) như Zabbix, Nagios, hoặc SolarWinds. SNMP cung cấp thông tin về trạng thái thiết bị (CPU, bộ nhớ, giao diện, v.v.) thông qua các **Management Information Base (MIB)** và hỗ trợ các phiên bản SNMPv1, SNMPv2c, và SNMPv3.

Tài liệu này hướng dẫn cách:
- Kích hoạt và cấu hình dịch vụ SNMP.
- Cấu hình cộng đồng (community) cho SNMPv1/v2c hoặc tài khoản người dùng cho SNMPv3.
- Áp dụng các chính sách bảo mật như giới hạn nguồn truy cập.
- Giám sát và kiểm tra trạng thái SNMP.

Các lệnh được thực thi trong **chế độ vận hành** (ký hiệu `$`) hoặc **chế độ cấu hình** (ký hiệu `#`). Các thay đổi cần được **commit** để có hiệu lực và **lưu** để duy trì sau khi khởi động lại.

## Tổng quan về SNMP

- **SNMPv1/v2c**: Sử dụng chuỗi cộng đồng (community string) để xác thực, đơn giản nhưng kém bảo mật.
- **SNMPv3**: Hỗ trợ xác thực người dùng, mã hóa, và kiểm soát truy cập an toàn hơn.
- **Các thành phần chính**:
  - **Community**: Chuỗi xác thực cho SNMPv1/v2c.
  - **User**: Tài khoản người dùng cho SNMPv3 với xác thực và mã hóa.
  - **MIB**: Cơ sở dữ liệu chứa thông tin trạng thái thiết bị (như `IF-MIB` cho giao diện, `UCD-SNMP-MIB` cho CPU/bộ nhớ).
  - **Trap**: Thông báo sự kiện gửi đến NMS khi xảy ra sự kiện (như giao diện down).

## Các lệnh cấu hình SNMP

### 1. Kích hoạt dịch vụ SNMP
- **Mô tả**: Kích hoạt dịch vụ SNMP trên vRouterOS.
- **Lệnh**:
  ```
  # set service snmp
  ```
- **Ví dụ**:
  ```
  $ configure
  # set service snmp
  # commit
  # save
  Saving configuration to '/config/config.boot'...
  Done
  ```
  - Kích hoạt dịch vụ SNMP.

### 2. Cấu hình SNMPv1/v2c với Community
- **Mô tả**: Cấu hình chuỗi cộng đồng và giới hạn nguồn truy cập.
- **Lệnh**:
  ```
  # set service snmp community <community-name> authorization <ro|rw>
  # set service snmp community <community-name> network <ip-address/prefix>
  ```
  - `<community-name>`: Tên chuỗi cộng đồng (ví dụ: `public`).
  - `<ro|rw>`: Quyền truy cập (read-only hoặc read-write).
  - `<ip-address/prefix>`: Mạng được phép truy cập SNMP.
- **Ví dụ**:
  ```
  # set service snmp community public authorization ro
  # set service snmp community public network 192.168.1.0/24
  # commit
  # save
  ```
  - Cấu hình cộng đồng `public` với quyền read-only, chỉ cho phép truy cập từ mạng `192.168.1.0/24`.

### 3. Cấu hình SNMPv3 với User
- **Mô tả**: Cấu hình tài khoản người dùng SNMPv3 với xác thực và mã hóa.
- **Lệnh**:
  ```
  # set service snmp v3 user <username> auth plaintext-password <password>
  # set service snmp v3 user <username> auth type <md5|sha>
  # set service snmp v3 user <username> privacy plaintext-password <password>
  # set service snmp v3 user <username> privacy type <aes|des>
  ```
  - `<username>`: Tên người dùng SNMPv3.
  - `<password>`: Mật khẩu cho xác thực hoặc mã hóa.
  - `<md5|sha>`: Loại xác thực.
  - `<aes|des>`: Loại mã hóa.
- **Ví dụ**:
  ```
  # set service snmp v3 user snmpuser auth plaintext-password MyAuthPass123
  # set service snmp v3 user snmpuser auth type sha
  # set service snmp v3 user snmpuser privacy plaintext-password MyPrivPass456
  # set service snmp v3 user snmpuser privacy type aes
  # commit
  # save
  ```
  - Cấu hình người dùng `snmpuser` với xác thực SHA và mã hóa AES.

### 4. Cấu hình SNMP Trap
- **Mô tả**: Cấu hình vRouterOS gửi thông báo trap đến NMS khi xảy ra sự kiện.
- **Lệnh**:
  ```
  # set service snmp trap-target <ip-address> community <community-name>
  # set service snmp trap-target <ip-address> port <port>
  ```
- **Ví dụ**:
  ```
  # set service snmp trap-target 192.168.1.100 community public
  # set service snmp trap-target 192.168.1.100 port 162
  # commit
  # save
  ```
  - Gửi trap đến NMS tại `192.168.1.100` qua cổng 162 với cộng đồng `public`.

### 5. Cấu hình thông tin hệ thống SNMP
- **Mô tả**: Cung cấp thông tin hệ thống như vị trí, liên hệ, hoặc mô tả.
- **Lệnh**:
  ```
  # set service snmp contact <contact-info>
  # set service snmp location <location-info>
  # set service snmp description <description>
  ```
- **Ví dụ**:
  ```
  # set service snmp contact "admin@example.com"
  # set service snmp location "Data Center Hanoi"
  # set service snmp description "Core Router vRouterOS"
  # commit
  # save
  ```
  - Cấu hình thông tin liên hệ, vị trí, và mô tả cho thiết bị.

### 6. Kiểm tra cấu hình SNMP
- **Mô tả**: Xem cấu hình SNMP hiện tại.
- **Lệnh**:
  - Trong chế độ cấu hình:
    ```
    # show service snmp
    ```
  - Trong chế độ vận hành:
    ```
    $ show configuration commands | match snmp
    ```
- **Ví dụ**:
  ```
  # show service snmp
  community public {
      authorization ro
      network 192.168.1.0/24
  }
  v3 {
      user snmpuser {
          auth {
              plaintext-password MyAuthPass123
              type sha
          }
          privacy {
              plaintext-password MyPrivPass456
              type aes
          }
      }
  }
  trap-target 192.168.1.100 {
      community public
      port 162
  }
  contact "admin@example.com"
  location "Data Center Hanoi"
  description "Core Router vRouterOS"
  ```

### 7. Kiểm tra trạng thái SNMP
- **Mô tả**: Kiểm tra trạng thái hoạt động của dịch vụ SNMP.
- **Lệnh**:
  ```
  $ show snmp
  $ show snmp community
  $ show snmp v3
  ```
- **Ví dụ**:
  ```
  $ show snmp community
  Community: public
  Authorization: read-only
  Allowed Networks: 192.168.1.0/24
  ```
  ```
  $ show snmp v3
  User: snmpuser
  Auth Type: sha
  Privacy Type: aes
  ```

### 8. Xóa cấu hình SNMP
- **Mô tả**: Xóa cấu hình SNMP hoặc một phần của nó.
- **Lệnh**:
  ```
  # delete service snmp community <community-name>
  # delete service snmp v3 user <username>
  # delete service snmp trap-target <ip-address>
  ```
- **Ví dụ**:
  ```
  $ configure
  # delete service snmp community public
  # commit
  # save
  ```
  - Xóa cộng đồng `public`.
  ```
  # delete service snmp
  # commit
  # save
  ```
  - Xóa toàn bộ cấu hình SNMP.

### 9. So sánh và hủy bỏ thay đổi
- **So sánh cấu hình**:
  ```
  # compare
  ```
  - Hiển thị sự khác biệt giữa cấu hình làm việc và cấu hình đang chạy.
  - Ví dụ:
    ```
    # compare
    + service snmp community public authorization ro
    + service snmp community public network 192.168.1.0/24
    ```
- **Hủy bỏ thay đổi**:
  ```
  # discard
  ```
  - Hủy các thay đổi chưa commit.

## Ví dụ thực tế: Cấu hình SNMPv2c và Trap
1. Vào chế độ cấu hình:
   ```
   $ configure
   ```
2. Kích hoạt SNMP và cấu hình cộng đồng:
   ```
   # set service snmp
   # set service snmp community public authorization ro
   # set service snmp community public network 192.168.1.0/24
   ```
3. Cấu hình thông tin hệ thống:
   ```
   # set service snmp contact "admin@example.com"
   # set service snmp location "Data Center Hanoi"
   ```
4. Cấu hình trap:
   ```
   # set service snmp trap-target 192.168.1.100 community public
   # set service snmp trap-target 192.168.1.100 port 162
   ```
5. Kiểm tra cấu hình:
   ```
   # show service snmp
   community public {
       authorization ro
       network 192.168.1.0/24
   }
   trap-target 192.168.1.100 {
       community public
       port 162
   }
   contact "admin@example.com"
   location "Data Center Hanoi"
   ```
6. Áp dụng và lưu:
   ```
   # commit
   # save
   Saving configuration to '/config/config.boot'...
   Done
   # exit
   ```
7. Kiểm tra trạng thái:
   ```
   $ show snmp community
   Community: public
   Authorization: read-only
   Allowed Networks: 192.168.1.0/24
   ```
   ```
   $ show snmp trap-target
   Trap Target: 192.168.1.100
   Community: public
   Port: 162
   ```

## Lưu ý quan trọng
- **Bảo mật**: Sử dụng SNMPv3 với xác thực và mã hóa để tăng cường bảo mật, đặc biệt khi truyền qua mạng không an toàn.
- **Giới hạn truy cập**: Luôn cấu hình `network` để giới hạn nguồn truy cập SNMP:
  ```
  # set service snmp community public network 192.168.1.0/24
  ```
- **Sao lưu cấu hình**: Luôn sao lưu trước khi thay đổi:
  ```
  $ save /config/backup-20250907.conf
  ```
- **Kiểm tra lỗi**: Nếu `commit` thất bại, đọc thông báo lỗi (ví dụ: cộng đồng trùng lặp, mật khẩu yếu).
- **Tài nguyên hệ thống**: SNMP có thể tăng tải CPU khi giám sát nhiều OID. Theo dõi bằng:
  ```
  $ show system cpu
  ```
  
## Kết luận
Cấu hình SNMP trong vRouterOS cho phép giám sát thiết bị mạng hiệu quả thông qua SNMPv1/v2c hoặc SNMPv3. Sử dụng các lệnh `set`, `delete`, `commit`, `show`, và `save` để cấu hình và quản lý SNMP. Thực hành các ví dụ trên để làm quen với quy trình và đảm bảo giám sát mạng an toàn và hiệu quả.
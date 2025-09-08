# Hướng dẫn cấu hình SSH trong vRouterOS

## Giới thiệu

Dịch vụ **SSH (Secure Shell)** trong **vRouterOS** cho phép quản trị viên truy cập từ xa vào thiết bị một cách an toàn để thực hiện cấu hình, giám sát, hoặc khắc phục sự cố. SSH sử dụng mã hóa để bảo vệ dữ liệu truyền tải và hỗ trợ xác thực qua mật khẩu hoặc khóa công khai (public key).

Tài liệu này hướng dẫn cách:
- Kích hoạt và cấu hình dịch vụ SSH.
- Quản lý cổng, giới hạn truy cập, và các tham số bảo mật.
- Cấu hình xác thực khóa công khai.
- Kiểm tra và giám sát trạng thái SSH.

Các lệnh được thực thi trong **chế độ vận hành** (ký hiệu `$`) hoặc **chế độ cấu hình** (ký hiệu `#`). Các thay đổi cần được **commit** để có hiệu lực và **lưu** để duy trì sau khi khởi động lại.

## Tổng quan về SSH

- **Chức năng SSH**: Cung cấp giao diện dòng lệnh an toàn (CLI) để quản lý vRouterOS từ xa.
- **Phiên bản hỗ trợ**: vRouterOS sử dụng OpenSSH, hỗ trợ SSHv2 (an toàn hơn SSHv1, đã bị vô hiệu hóa theo mặc định).
- **Tính năng chính**:
  - Xác thực qua mật khẩu hoặc khóa công khai.
  - Giới hạn truy cập dựa trên địa chỉ IP/mạng.
  - Tùy chỉnh cổng SSH để tránh các cuộc tấn công quét cổng (port scanning).
- **Mặc định**: Dịch vụ SSH được kích hoạt trên cổng 22 với xác thực mật khẩu, nhưng có thể được tùy chỉnh.

## Các lệnh cấu hình SSH

### 1. Kích hoạt dịch vụ SSH
- **Mô tả**: Kích hoạt dịch vụ SSH trên vRouterOS.
- **Lệnh**:
  ```
  # set service ssh
  ```
- **Ví dụ**:
  ```
  $ configure
  # set service ssh
  # commit
  # save
  Saving configuration to '/config/config.boot'...
  Done
  ```
  - Kích hoạt dịch vụ SSH với cấu hình mặc định (cổng 22, xác thực mật khẩu).

### 2. Cấu hình cổng SSH
- **Mô tả**: Thay đổi cổng mặc định (22) để tăng cường bảo mật.
- **Lệnh**:
  ```
  # set service ssh port <port-number>
  ```
  - `<port-number>`: Số cổng từ 1 đến 65535 (khuyến nghị tránh các cổng phổ biến).
- **Ví dụ**:
  ```
  # set service ssh port 2222
  # commit
  # save
  ```
  - Đặt cổng SSH thành 2222.

### 3. Giới hạn truy cập SSH
- **Mô tả**: Chỉ cho phép các địa chỉ IP hoặc mạng cụ thể truy cập SSH.
- **Lệnh**:
  ```
  # set service ssh access-control allow address <ip-address>
  # set service ssh access-control allow network <ip-address/prefix>
  ```
- **Ví dụ**:
  ```
  # set service ssh access-control allow network 192.168.1.0/24
  # commit
  # save
  ```
  - Chỉ cho phép truy cập SSH từ mạng `192.168.1.0/24`.

### 4. Cấu hình xác thực khóa công khai
- **Mô tả**: Cho phép xác thực SSH bằng khóa công khai thay vì mật khẩu.
- **Lệnh**:
  ```
  # set service ssh authorized-key <key-name> key <public-key>
  # set service ssh authorized-key <key-name> user <username>
  ```
  - `<key-name>`: Tên định danh cho khóa.
  - `<public-key>`: Chuỗi khóa công khai (thường là SSH-RSA hoặc ECDSA).
  - `<username>`: Tài khoản người dùng được phép sử dụng khóa.
- **Ví dụ**:
  ```
  # set service ssh authorized-key admin-key key "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC..."
  # set service ssh authorized-key admin-key user admin
  # commit
  # save
  ```
  - Thêm khóa công khai cho người dùng `admin`.

### 5. Vô hiệu hóa xác thực mật khẩu
- **Mô tả**: Tắt xác thực mật khẩu để chỉ cho phép xác thực bằng khóa công khai.
- **Lệnh**:
  ```
  # set service ssh disable-password-authentication
  ```
- **Ví dụ**:
  ```
  # set service ssh disable-password-authentication
  # commit
  # save
  ```
  - Vô hiệu hóa xác thực mật khẩu, yêu cầu sử dụng khóa công khai.

### 6. Kiểm tra cấu hình SSH
- **Mô tả**: Xem cấu hình SSH hiện tại.
- **Lệnh**:
  - Trong chế độ cấu hình:
    ```
    # show service ssh
    ```
  - Trong chế độ vận hành:
    ```
    $ show configuration commands | match ssh
    ```
- **Ví dụ**:
  ```
  # show service ssh
  port 2222
  access-control {
      allow {
          network 192.168.1.0/24
      }
  }
  authorized-key admin-key {
      key ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC...
      user admin
  }
  disable-password-authentication
  ```

### 7. Kiểm tra trạng thái SSH
- **Mô tả**: Kiểm tra trạng thái dịch vụ SSH và các kết nối hiện tại.
- **Lệnh**:
  ```
  $ show system ssh
  ```
- **Ví dụ**:
  ```
  $ show system ssh
  State: enabled
  Port: 2222
  Active Connections:
      User: admin  From: 192.168.1.100  Time: 2025-09-08 12:00:00
  ```
  - Hiển thị trạng thái dịch vụ SSH và thông tin kết nối.

### 8. Khởi động lại dịch vụ SSH
- **Mô tả**: Khởi động lại dịch vụ SSH để áp dụng thay đổi mà không cần khởi động lại hệ thống.
- **Lệnh**:
  ```
  $ restart ssh
  ```
- **Ví dụ**:
  ```
  $ restart ssh
  Restarting SSH service...
  Done
  ```
  - Khởi động lại dịch vụ SSH.

### 9. Xóa cấu hình SSH
- **Mô tả**: Xóa cấu hình SSH hoặc một phần của nó.
- **Lệnh**:
  ```
  # delete service ssh
  # delete service ssh port
  # delete service ssh access-control
  # delete service ssh authorized-key <key-name>
  ```
- **Ví dụ**:
  ```
  $ configure
  # delete service ssh authorized-key admin-key
  # commit
  # save
  ```
  - Xóa khóa công khai `admin-key`.
  ```
  # delete service ssh
  # commit
  # save
  ```
  - Tắt dịch vụ SSH hoàn toàn.

### 10. So sánh và hủy bỏ thay đổi
- **So sánh cấu hình**:
  ```
  # compare
  ```
  - Hiển thị sự khác biệt giữa cấu hình làm việc và cấu hình đang chạy.
  - Ví dụ:
    ```
    # compare
    + service ssh port 2222
    + service ssh access-control allow network 192.168.1.0/24
    ```
- **Hủy bỏ thay đổi**:
  ```
  # discard
  ```
  - Hủy các thay đổi chưa commit.

## Ví dụ thực tế: Cấu hình SSH với xác thực khóa công khai
1. Vào chế độ cấu hình:
   ```
   $ configure
   ```
2. Kích hoạt SSH và cấu hình cổng:
   ```
   # set service ssh port 2222
   ```
3. Giới hạn truy cập:
   ```
   # set service ssh access-control allow network 192.168.1.0/24
   ```
4. Thêm khóa công khai:
   ```
   # set service ssh authorized-key admin-key key "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC..."
   # set service ssh authorized-key admin-key user admin
   ```
5. Vô hiệu hóa xác thực mật khẩu:
   ```
   # set service ssh disable-password-authentication
   ```
6. Kiểm tra cấu hình:
   ```
   # show service ssh
   port 2222
   access-control {
       allow {
           network 192.168.1.0/24
       }
   }
   authorized-key admin-key {
       key ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC...
       user admin
   }
   disable-password-authentication
   ```
7. Áp dụng và lưu:
   ```
   # commit
   # save
   Saving configuration to '/config/config.boot'...
   Done
   # exit
   ```
8. Kiểm tra trạng thái:
   ```
   $ show system ssh
   State: enabled
   Port: 2222
   Active Connections: None
   ```
9. Kết nối thử từ máy client:
   ```
   ssh -p 2222 admin@192.168.1.1
   ```

## Lưu ý quan trọng
- **Bảo mật cổng**: Thay đổi cổng mặc định (22) để giảm nguy cơ tấn công quét cổng:
  ```
  # set service ssh port 2222
  ```
- **Khóa công khai**: Đảm bảo khóa công khai được thêm đúng định dạng (RSA, ECDSA, v.v.) và liên kết với người dùng hợp lệ.
- **Giới hạn truy cập**: Luôn cấu hình `access-control` để chỉ cho phép các mạng đáng tin cậy:
  ```
  # set service ssh access-control allow network 192.168.1.0/24
  ```
- **Sao lưu cấu hình**: Luôn sao lưu trước khi thay đổi:
  ```
  $ save /config/backup-20250908.conf
  ```
- **Kiểm tra lỗi**: Nếu `commit` thất bại, đọc thông báo lỗi (ví dụ: cổng trùng lặp, khóa công khai không hợp lệ).
- **Tài nguyên hệ thống**: Theo dõi tải CPU khi có nhiều kết nối SSH:
  ```
  $ show system cpu
  ```

## Kết luận
Cấu hình SSH trong vRouterOS cho phép quản trị viên truy cập từ xa an toàn với các tùy chọn bảo mật như xác thực khóa công khai và giới hạn truy cập. Sử dụng các lệnh `set`, `delete`, `commit`, `show`, và `save` để cấu hình và quản lý SSH hiệu quả. Thực hành các ví dụ trên để làm quen với quy trình và đảm bảo truy cập an toàn vào thiết bị.
# Hướng dẫn cấu hình Large Community List Policy trong vRouterOS

## Giới thiệu

**Large Community List Policy** trong **vRouterOS** là một cơ chế dùng để lọc và thao tác các tuyến đường BGP dựa trên thuộc tính cộng đồng lớn (large community attribute). Large Community List cho phép quản trị viên xác định các tuyến đường được phép hoặc bị từ chối dựa trên các giá trị cộng đồng lớn, đồng thời hỗ trợ sửa đổi thuộc tính này để điều khiển chính sách định tuyến, đặc biệt trong các mạng BGP phức tạp yêu cầu khả năng gắn nhãn chi tiết hơn so với cộng đồng chuẩn hoặc mở rộng.

Tài liệu này hướng dẫn cách:
- Tạo và cấu hình Large Community List với các quy tắc để lọc hoặc sửa đổi cộng đồng lớn.
- Áp dụng Large Community List vào giao thức BGP thông qua Route Map.
- Kiểm tra và giám sát cấu hình Large Community List.

Các lệnh được thực thi trong **chế độ vận hành** (ký hiệu `$`) hoặc **chế độ cấu hình** (ký hiệu `#`). Các thay đổi cần được **commit** để có hiệu lực và **lưu** để duy trì sau khi khởi động lại.

## Tổng quan về Large Community List Policy

- **Large Community List**: Một tập hợp các quy tắc (rules) xác định các giá trị cộng đồng lớn BGP được phép (`permit`) hoặc bị từ chối (`deny`) sử dụng biểu thức chính quy (regular expression) hoặc giá trị cụ thể.
- **Thuộc tính Large Community trong BGP**:
  - Là một nhãn 96-bit (so với 32-bit của cộng đồng chuẩn và 64-bit của cộng đồng mở rộng), gồm ba trường: Global Administrator (thường là AS), Local Data Part 1, và Local Data Part 2 (ví dụ: `65001:100:200`).
  - Thường được sử dụng để gắn nhãn chi tiết hơn trong các chính sách định tuyến phức tạp.
- **Ứng dụng**:
  - Lọc tuyến đường BGP trong các mạng lớn hoặc liên AS.
  - Sửa đổi thuộc tính cộng đồng lớn để điều chỉnh chính sách định tuyến, chẳng hạn như ưu tiên tuyến đường hoặc kiểm soát luồng lưu lượng.
  - Hỗ trợ các kịch bản như inter-domain routing hoặc chính sách định tuyến trong các nhà cung cấp dịch vụ.
- **Biểu thức chính quy (Regular Expression)**:
  - `.*`: Khớp bất kỳ chuỗi cộng đồng lớn.
  - `65001:.*:.*`: Khớp cộng đồng lớn với Global Administrator là AS 65001.
  - `[0-9]+:[0-9]+:[0-9]+`: Khớp định dạng cộng đồng lớn (AS:Data1:Data2).

## Các lệnh cấu hình Large Community List

### 1. Tạo Large Community List
- **Mô tả**: Tạo một Large Community List với các quy tắc để cho phép hoặc từ chối các giá trị cộng đồng lớn.
- **Lệnh**:
  ```
  # set policy large-community-list <list-name> rule <rule-number> action <permit|deny>
  # set policy large-community-list <list-name> rule <rule-number> regex <expression>
  ```
  - `<list-name>`: Tên của Large Community List (ví dụ: `BGP_LARGECOMMUNITY_FILTER`).
  - `<rule-number>`: Số thứ tự quy tắc (1-65535).
  - `<expression>`: Biểu thức chính quy hoặc giá trị cộng đồng lớn cụ thể (ví dụ: `65001:100:200`).
- **Ví dụ**:
  ```
  $ configure
  # set policy large-community-list BGP_LARGECOMMUNITY_FILTER rule 10 action permit
  # set policy large-community-list BGP_LARGECOMMUNITY_FILTER rule 10 regex "65001:100:200"
  # set policy large-community-list BGP_LARGECOMMUNITY_FILTER rule 20 action deny
  # set policy large-community-list BGP_LARGECOMMUNITY_FILTER rule 20 regex "65002:.*:300"
  # commit
  # save
  Saving configuration to '/config/config.boot'...
  Done
  ```
  - Tạo Large Community List `BGP_LARGECOMMUNITY_FILTER`:
    - Quy tắc 10: Cho phép tuyến đường với cộng đồng lớn `65001:100:200`.
    - Quy tắc 20: Từ chối tuyến đường với cộng đồng lớn có Global Administrator là `65002` và Local Data Part 2 là `300`.

### 2. Áp dụng Large Community List vào Route Map
- **Mô tả**: Sử dụng Large Community List trong Route Map để lọc hoặc sửa đổi tuyến đường BGP.
- **Lệnh**:
  ```
  # set policy route-map <map-name> rule <rule-number> match large-community large-community-list <list-name>
  # set policy route-map <map-name> rule <rule-number> action <permit|deny>
  # set policy route-map <map-name> rule <rule-number> set large-community <large-community-value>
  ```
- **Ví dụ**:
  ```
  # set policy route-map BGP_CONTROL rule 10 match large-community large-community-list BGP_LARGECOMMUNITY_FILTER
  # set policy route-map BGP_CONTROL rule 10 action permit
  # set policy route-map BGP_CONTROL rule 10 set large-community "65001:200:300"
  # set policy route-map BGP_CONTROL rule 20 action deny
  # commit
  # save
  ```
  - Tạo Route Map `BGP_CONTROL`:
    - Quy tắc 10: Cho phép tuyến đường khớp với Large Community List `BGP_LARGECOMMUNITY_FILTER` và đặt cộng đồng lớn mới là `65001:200:300`.
    - Quy tắc 20: Từ chối các tuyến đường còn lại.

### 3. Áp dụng Route Map vào BGP
- **Mô tả**: Gắn Route Map vào giao thức BGP để áp dụng chính sách lọc hoặc sửa đổi.
- **Lệnh**:
  ```
  # set protocols bgp <asn> neighbor <neighbor-ip> route-map <import|export> <map-name>
  ```
  - `<asn>`: Số AS của BGP.
  - `<neighbor-ip>`: Địa chỉ IP của láng giềng BGP.
  - `<import|export>`: Áp dụng Route Map cho tuyến đường nhận vào (`import`) hoặc gửi đi (`export`).
- **Ví dụ**:
  ```
  # set protocols bgp 65001 neighbor 192.168.1.2 route-map import BGP_CONTROL
  # commit
  # save
  ```
  - Áp dụng Route Map `BGP_CONTROL` để lọc các tuyến đường nhận từ láng giềng BGP `192.168.1.2`.

### 4. Kiểm tra cấu hình Large Community List
- **Mô tả**: Xem cấu hình Large Community List hiện tại.
- **Lệnh**:
  - Trong chế độ cấu hình:
    ```
    # show policy large-community-list
    ```
  - Trong chế độ vận hành:
    ```
    $ show configuration commands | match large-community-list
    ```
- **Ví dụ**:
  ```
  # show policy large-community-list
  large-community-list BGP_LARGECOMMUNITY_FILTER {
      rule 10 {
          action permit
          regex "65001:100:200"
      }
      rule 20 {
          action deny
          regex "65002:.*:300"
      }
  }
  ```

### 5. Kiểm tra trạng thái Large Community List
- **Mô tả**: Kiểm tra trạng thái áp dụng của Large Community List trong giao thức BGP thông qua Route Map.
- **Lệnh**:
  ```
  $ show ip bgp neighbors <neighbor-ip>
  ```
- **Ví dụ**:
  ```
  $ show ip bgp neighbors 192.168.1.2
  BGP neighbor is 192.168.1.2, remote AS 65002
  Route-map: import BGP_CONTROL
  ```
  - Xác nhận Route Map `BGP_CONTROL` (liên kết với Large Community List) được áp dụng cho láng giềng BGP.

### 6. Xóa Large Community List
- **Mô tả**: Xóa Large Community List hoặc một quy tắc cụ thể.
- **Lệnh**:
  ```
  # delete policy large-community-list <list-name> rule <rule-number>
  # delete policy large-community-list <list-name>
  ```
- **Ví dụ**:
  ```
  $ configure
  # delete policy large-community-list BGP_LARGECOMMUNITY_FILTER rule 10
  # commit
  # save
  ```
  - Xóa quy tắc 10 trong Large Community List `BGP_LARGECOMMUNITY_FILTER`.
  ```
  # delete policy large-community-list BGP_LARGECOMMUNITY_FILTER
  # commit
  # save
  ```
  - Xóa toàn bộ Large Community List `BGP_LARGECOMMUNITY_FILTER`.

### 7. So sánh và hủy bỏ thay đổi
- **So sánh cấu hình**:
  ```
  # compare
  ```
  - Hiển thị sự khác biệt giữa cấu hình làm việc và cấu hình đang chạy.
  - Ví dụ:
    ```
    # compare
    + policy large-community-list BGP_LARGECOMMUNITY_FILTER rule 10 action permit
    + policy large-community-list BGP_LARGECOMMUNITY_FILTER rule 10 regex "65001:100:200"
    ```
- **Hủy bỏ thay đổi**:
  ```
  # discard
  ```
  - Hủy các thay đổi chưa commit.

## Ví dụ thực tế: Cấu hình Large Community List cho BGP
1. Vào chế độ cấu hình:
   ```
   $ configure
   ```
2. Tạo Large Community List:
   ```
   # set policy large-community-list BGP_LARGECOMMUNITY_FILTER rule 10 action permit
   # set policy large-community-list BGP_LARGECOMMUNITY_FILTER rule 10 regex "65001:100:200"
   # set policy large-community-list BGP_LARGECOMMUNITY_FILTER rule 20 action deny
   # set policy large-community-list BGP_LARGECOMMUNITY_FILTER rule 20 regex "65002:.*:300"
   ```
3. Tạo Route Map sử dụng Large Community List:
   ```
   # set policy route-map BGP_CONTROL rule 10 match large-community large-community-list BGP_LARGECOMMUNITY_FILTER
   # set policy route-map BGP_CONTROL rule 10 action permit
   # set policy route-map BGP_CONTROL rule 10 set large-community "65001:200:300"
   # set policy route-map BGP_CONTROL rule 20 action deny
   ```
4. Áp dụng Route Map vào BGP:
   ```
   # set protocols bgp 65001 neighbor 192.168.1.2 route-map import BGP_CONTROL
   ```
5. Kiểm tra cấu hình:
   ```
   # show policy large-community-list
   large-community-list BGP_LARGECOMMUNITY_FILTER {
       rule 10 {
           action permit
           regex "65001:100:200"
       }
       rule 20 {
           action deny
           regex "65002:.*:300"
       }
   }
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
   $ show ip bgp neighbors 192.168.1.2
   BGP neighbor is 192.168.1.2, remote AS 65002
   Route-map: import BGP_CONTROL
   ```

## Lưu ý quan trọng
- **Thứ tự quy tắc**: Quy tắc được xử lý theo số thứ tự (`rule-number`). Quy tắc có số nhỏ hơn được ưu tiên.
- **Hành động mặc định**: Nếu không có quy tắc nào khớp, tuyến đường được phép (`permit`) trừ khi có quy tắc `deny` cuối cùng.
- **Biểu thức chính quy**: Đảm bảo biểu thức chính quy được viết đúng để tránh lỗi lọc. Sử dụng các công cụ như `regex101.com` để kiểm tra.
- **Sao lưu cấu hình**: Luôn sao lưu trước khi thay đổi:
  ```
  $ save /config/backup-20250908.conf
  ```
- **Kiểm tra lỗi**: Nếu `commit` thất bại, đọc thông báo lỗi (ví dụ: biểu thức chính quy không hợp lệ, Route Map không tồn tại).
- **Tích hợp với Route Map**: Large Community List thường được sử dụng trong Route Map. Đảm bảo Route Map được cấu hình và áp dụng đúng.
- **Tài nguyên hệ thống**: Large Community List phức tạp có thể tăng tải CPU. Theo dõi bằng:
  ```
  $ show system cpu
  ```

## Kết luận
Large Community List Policy trong vRouterOS cung cấp khả năng kiểm soát chi tiết các tuyến đường BGP trong các mạng phức tạp, đặc biệt khi cần gắn nhãn chi tiết hơn so với cộng đồng chuẩn hoặc mở rộng. Sử dụng các lệnh `set`, `delete`, `commit`, `show`, và `save` để cấu hình và quản lý Large Community List hiệu quả. Thực hành các ví dụ trên để làm quen với quy trình và đảm bảo kiểm soát tuyến đường BGP một cách chính xác.
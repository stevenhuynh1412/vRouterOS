# Hướng dẫn cấu hình BGP trong vRouterOS

## Giới thiệu

**Border Gateway Protocol (BGP)** là giao thức định tuyến động được sử dụng để trao đổi thông tin định tuyến giữa các hệ thống tự trị (Autonomous Systems - AS) trong mạng Internet hoặc mạng doanh nghiệp lớn. Trong **vRouterOS**, BGP được triển khai dựa trên FRRouting (FRR), cung cấp khả năng cấu hình linh hoạt cho các kịch bản định tuyến phức tạp, bao gồm iBGP (internal BGP) và eBGP (external BGP).

Tài liệu này hướng dẫn cách:
- Cấu hình BGP cơ bản, bao gồm thiết lập AS và láng giềng (neighbor).
- Áp dụng các chính sách định tuyến như Prefix List, Route Map, hoặc Community List.
- Kiểm tra và giám sát trạng thái BGP.

Các lệnh được thực thi trong **chế độ vận hành** (ký hiệu `$`) hoặc **chế độ cấu hình** (ký hiệu `#`). Các thay đổi cần được **commit** để có hiệu lực và **lưu** để duy trì sau khi khởi động lại.

## Tổng quan về BGP

- **BGP**: Giao thức định tuyến lớp 4 (Layer 4) sử dụng TCP (cổng 179) để thiết lập kết nối giữa các láng giềng BGP. BGP hỗ trợ cả IPv4 và IPv6.
- **Loại BGP**:
  - **eBGP**: Định tuyến giữa các AS khác nhau (ví dụ: giữa ISP và doanh nghiệp).
  - **iBGP**: Định tuyến trong cùng một AS (ví dụ: giữa các router trong mạng nội bộ).
- **Thuộc tính BGP**:
  - AS Path: Danh sách các AS mà tuyến đường đi qua.
  - Community: Nhãn để gắn chính sách định tuyến.
  - Local Preference, MED (Multi-Exit Discriminator): Điều chỉnh ưu tiên tuyến đường.
- **Ứng dụng**: BGP được sử dụng trong các kịch bản như định tuyến Internet, kết nối đa nhà cung cấp (multi-homing), hoặc triển khai VPN/MPLS.

## Các lệnh cấu hình BGP

### 1. Kích hoạt BGP và thiết lập AS
- **Mô tả**: Kích hoạt BGP và chỉ định số AS (Autonomous System) cho router.
- **Lệnh**:
  ```
  # set protocols bgp <asn>
  ```
  - `<asn>`: Số AS (1-4294967295).
- **Ví dụ**:
  ```
  $ configure
  # set protocols bgp 65001
  # commit
  # save
  Saving configuration to '/config/config.boot'...
  Done
  ```
  - Kích hoạt BGP với AS 65001.

### 2. Cấu hình láng giềng BGP
- **Mô tả**: Thiết lập láng giềng BGP (neighbor) để trao đổi thông tin định tuyến.
- **Lệnh**:
  ```
  # set protocols bgp <asn> neighbor <neighbor-ip> remote-as <remote-asn>
  ```
  - `<neighbor-ip>`: Địa chỉ IP của láng giềng BGP.
  - `<remote-asn>`: Số AS của láng giềng.
- **Ví dụ**:
  ```
  # set protocols bgp 65001 neighbor 192.168.1.2 remote-as 65002
  # commit
  # save
  ```
  - Thiết lập láng giềng BGP tại `192.168.1.2` với AS 65002.

### 3. Cấu hình mạng quảng bá (Network Advertisement)
- **Mô tả**: Quảng bá một mạng (prefix) đến các láng giềng BGP.
- **Lệnh**:
  ```
  # set protocols bgp <asn> network <network/prefix>
  ```
- **Ví dụ**:
  ```
  # set protocols bgp 65001 network 192.168.10.0/24
  # commit
  # save
  ```
  - Quảng bá mạng `192.168.10.0/24` đến các láng giềng BGP.

### 4. Áp dụng chính sách định tuyến
- **Mô tả**: Sử dụng Route Map, Prefix List, hoặc Community List để lọc hoặc sửa đổi tuyến đường.
- **Lệnh** (ví dụ với Route Map):
  ```
  # set protocols bgp <asn> neighbor <neighbor-ip> route-map <import|export> <map-name>
  ```
- **Ví dụ**:
  ```
  # set policy route-map BGP_CONTROL rule 10 action permit
  # set policy route-map BGP_CONTROL rule 10 match ip address prefix-list BGP_FILTER
  # set protocols bgp 65001 neighbor 192.168.1.2 route-map import BGP_CONTROL
  # commit
  # save
  ```
  - Áp dụng Route Map `BGP_CONTROL` để lọc các tuyến đường nhận từ láng giềng `192.168.1.2`.

### 5. Cấu hình iBGP
- **Mô tả**: Thiết lập BGP trong cùng một AS, thường yêu cầu chỉ định `update-source` và `next-hop-self`.
- **Lệnh**:
  ```
  # set protocols bgp <asn> neighbor <neighbor-ip> remote-as <asn>
  # set protocols bgp <asn> neighbor <neighbor-ip> update-source <interface>
  # set protocols bgp <asn> neighbor <neighbor-ip> next-hop-self
  ```
- **Ví dụ**:
  ```
  # set protocols bgp 65001 neighbor 192.168.2.2 remote-as 65001
  # set protocols bgp 65001 neighbor 192.168.2.2 update-source lo
  # set protocols bgp 65001 neighbor 192.168.2.2 next-hop-self
  # commit
  # save
  ```
  - Cấu hình iBGP với láng giềng `192.168.2.2` trong AS 65001, sử dụng giao diện loopback (`lo`) làm nguồn cập nhật.

### 6. Kích hoạt IPv6 BGP
- **Mô tả**: Cấu hình BGP cho địa chỉ IPv6.
- **Lệnh**:
  ```
  # set protocols bgp <asn> neighbor <neighbor-ipv6> address-family ipv6-unicast
  # set protocols bgp <asn> network <ipv6-network/prefix>
  ```
- **Ví dụ**:
  ```
  # set protocols bgp 65001 neighbor 2001:db8::2 address-family ipv6-unicast
  # set protocols bgp 65001 network 2001:db8:1::/48
  # commit
  # save
  ```
  - Kích hoạt BGP IPv6 cho láng giềng `2001:db8::2` và quảng bá mạng `2001:db8:1::/48`.

### 7. Kiểm tra trạng thái BGP
- **Mô tả**: Hiển thị thông tin về trạng thái BGP, láng giềng, và bảng định tuyến.
- **Lệnh**:
  ```
  $ show ip bgp
  $ show ip bgp neighbors
  $ show ip bgp summary
  ```
- **Ví dụ**:
  ```
  $ show ip bgp summary
  BGP router identifier 192.168.1.1, local AS number 65001
  Neighbor        V    AS   MsgRcvd MsgSent  TblVer  InQ OutQ Up/Down  State/PfxRcd
  192.168.1.2     4    65002  100     100      0       0   0    00:30:00 10
  ```
  - Hiển thị trạng thái láng giềng BGP, số lượng tuyến đường nhận được (`PfxRcd`).

### 8. Xóa cấu hình BGP
- **Mô tả**: Xóa cấu hình BGP hoặc một phần của nó.
- **Lệnh**:
  ```
  # delete protocols bgp <asn> neighbor <neighbor-ip>
  # delete protocols bgp <asn>
  ```
- **Ví dụ**:
  ```
  $ configure
  # delete protocols bgp 65001 neighbor 192.168.1.2
  # commit
  # save
  ```
  - Xóa láng giềng BGP `192.168.1.2`.
  ```
  # delete protocols bgp 65001
  # commit
  # save
  ```
  - Xóa toàn bộ cấu hình BGP cho AS 65001.

### 9. So sánh và hủy bỏ thay đổi
- **So sánh cấu hình**:
  ```
  # compare
  ```
  - Hiển thị sự khác biệt giữa cấu hình làm việc và cấu hình đang chạy.
  - Ví dụ:
    ```
    # compare
    + protocols bgp 65001 neighbor 192.168.1.2 remote-as 65002
    ```
- **Hủy bỏ thay đổi**:
  ```
  # discard
  ```
  - Hủy các thay đổi chưa commit.

## Ví dụ thực tế: Cấu hình BGP cơ bản (eBGP)
1. Vào chế độ cấu hình:
   ```
   $ configure
   ```
2. Kích hoạt BGP và thiết lập AS:
   ```
   # set protocols bgp 65001
   ```
3. Cấu hình láng giềng BGP:
   ```
   # set protocols bgp 65001 neighbor 192.168.1.2 remote-as 65002
   ```
4. Quảng bá mạng:
   ```
   # set protocols bgp 65001 network 192.168.10.0/24
   ```
5. Áp dụng Route Map (nếu cần):
   ```
   # set policy route-map BGP_CONTROL rule 10 action permit
   # set policy route-map BGP_CONTROL rule 10 match ip address prefix-list BGP_FILTER
   # set protocols bgp 65001 neighbor 192.168.1.2 route-map import BGP_CONTROL
   ```
6. Kiểm tra cấu hình:
   ```
   # show protocols bgp
   bgp 65001 {
       neighbor 192.168.1.2 {
           remote-as 65002
           route-map {
               import BGP_CONTROL
           }
       }
       network 192.168.10.0/24
   }
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
   $ show ip bgp summary
   BGP router identifier 192.168.1.1, local AS number 65001
   Neighbor        V    AS   MsgRcvd MsgSent  TblVer  InQ OutQ Up/Down  State/PfxRcd
   192.168.1.2     4    65002  100     100      0       0   0    00:30:00 10
   ```

## Lưu ý quan trọng
- **Kiểm tra kết nối**: Đảm bảo kết nối TCP cổng 179 giữa các láng giềng BGP không bị chặn bởi tường lửa.
- **Chính sách định tuyến**: Sử dụng Prefix List, Route Map, hoặc Community List để kiểm soát tuyến đường và tránh quảng bá không mong muốn.
- **Sao lưu cấu hình**: Luôn sao lưu trước khi thay đổi:
  ```
  $ save /config/backup-20250908.conf
  ```
- **Kiểm tra lỗi**: Nếu `commit` thất bại, đọc thông báo lỗi (ví dụ: láng giềng không hợp lệ, mạng không tồn tại).
- **Hiệu suất hệ thống**: BGP với bảng định tuyến lớn có thể tiêu tốn tài nguyên. Theo dõi bằng:
  ```
  $ show system cpu
  ```
  
## Kết luận
Cấu hình BGP trong vRouterOS cho phép quản trị viên triển khai định tuyến động hiệu quả trong các mạng phức tạp. Sử dụng các lệnh `set`, `delete`, `commit`, `show`, và `save` để cấu hình và quản lý BGP. Thực hành các ví dụ trên để làm quen với quy trình và đảm bảo định tuyến BGP hoạt động ổn định và an toàn.
# Hướng dẫn cài đặt vRouterOS

## Giới thiệu
**vRouterOS** là một hệ điều hành router ảo hóa, được phát triển dựa trên dự án DANOS (Disaggregated Network Operating System). Hệ điều hành này cung cấp các tính năng định tuyến, firewall, và quản lý băng thông chuyên nghiệp cho môi trường ảo hóa hoặc phần cứng x86.

## Yêu cầu hệ thống
- **RAM**: Tối thiểu 2 GB (Khuyến nghị 4 GB cho môi trường production).
- **Ổ cứng**: Tối thiểu 4 GB dung lượng trống (Khuyến nghị 8 GB).
- **CPU**: Hỗ trợ Supplemental Streaming SIMD Extensions 3 (SSSE3). Kiểm tra bằng lệnh: `grep ssse3 /proc/cpuinfo`.
- **Hypervisor**: VirtualBox (phiên bản 6.1 trở lên), VMware, hoặc KVM.
- **File ISO vRouterOS**: Tải từ https://res.fwcloud.co/vRouter-base-amd64-2025.09.06.iso


## Các bước cài đặt trên VirtualBox

### 1. Tạo máy ảo
- Mở VirtualBox Manager và nhấp **New**.
- Đặt tên máy ảo (ví dụ: `vRouterOS`).
- Chọn **Linux** → **Debian (64-bit)**.
- Cấu hình:
  - **RAM**: 4096 MB.
  - **Ổ cứng**: Tạo đĩa ảo mới (VDI), kích thước cố định 8 GB.

### 2. Cấu hình phần cứng ảo
- **CPU**: Chỉ định 4 vCPU (tối thiểu 2 vCPU).
- **Network**:
  - **Adapter 1**: Chọn **Bridged Adapter** → **Adapter Type**: `Paravirtualized Network (virtio-net)`.
  - **Adapter 2-4**: Chọn **Internal Network** → Đặt tên `intnet` → **Adapter Type**: `virtio-net`.

### 3. Gắn file ISO và khởi động
- Chọn máy ảo → **Settings** → **Storage** → **Add ISO** (chọn file vRouterOS ISO).
- Khởi động máy ảo. Hệ thống sẽ boot vào shell.

### 4. Thiết lập cấu hình ban đầu
- Đăng nhập với thông tin mặc định:
  - **Username**: `theo thông tin khi cài đặt`.
  - **Password**: `theo thông tin khi cài đặt`.
- Kích hoạt DHCP và SSH bằng các lệnh:
  ```bash
  config
  set interfaces dataplane dp0p0s3 address dhcp
  set service ssh
  commit
  exit
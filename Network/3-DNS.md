# Tìm hiểu systemd-resolved
systemd-resolved là một dịch vụ quản lý DNS và tên miền trên Linux, đi kèm với systemd, đặc biệt phổ biến trên Debian/Ubuntu mới. Nó thay thế việc chỉnh trực tiếp /etc/resolv.conf và cung cấp nhiều tính năng hơn so với cách truyền thống.
## Chức năng chính systemd-resolved
1. Phân giải tên miền
2. Cache DNS
3. Quản lí nhiều nguồn DNS : Có thể nhận DNS từ NetworkManager , DHCP client, cấu hình tĩnh. Systemd-resolved kết hợp các nguồn này một cách thông minh
4. Hỗ trợ DNS theo interface riêng (wifi, card mạng)

## File cấu hình
+ File: /etc/systemd/resolved.conf
```
[Resolve]
DNS=8.8.8.8 8.8.4.4
FallbackDNS=1.1.1.1
Cache=yes
```
+ File : /etc/resolv.conf
Trên Ubuntu mới, thường là symbolic link tới:
`/run/systemd/resolve/stub-resolv.conf`
Stub resolver mặc định là 127.0.0.53, systemd-resolved sẽ lắng nghe ở đây và forward DNS request.

## Kiểm tra trạng thái và cache
```
# Kiểm tra trạng thái dịch vụ
systemctl status systemd-resolved

# Kiểm tra DNS hiện tại của các interface
resolvectl status

# Kiểm tra cache
resolvectl query example.com
```

# Tìm hiểu về stub resolver
Stub resolver là “trung gian nhẹ” trên client, nhận request từ ứng dụng → chuyển tiếp tới DNS server thực. Nó không làm phân giải hoàn chỉnh. Khi dùng systemd-resolved, stub resolver được “nhúng” ở địa chỉ 127.0.0.53, chịu trách nhiệm cache, forwarding, và phân giải tên miền theo interface.

Luồng request như sau:
ví dụ trên debian/ubuntu mới:
`/etc/resolv.conf → 127.0.0.53`
Đây chính là stub resolver của systemd-resolved.
Luồng :
`ứng dụng → stub resolver (127.0.0.53) → systemd-resolved → server DNS thực (VD: 8.8.8.8) → cache → trả về ứng dụng`

Lợi ích:
Stub resolver luôn dùng cùng 1 địa chỉ IP nội bộ (127.0.0.53) cho mọi ứng dụng.
systemd-resolved quản lý cache, fallback DNS, và DNS theo interface.

# Tìm hiểu về resolvectl 
resolvectl là công cụ dòng lệnh (CLI) để tương tác với systemd-resolved trên các hệ thống Linux dùng systemd, ví dụ Ubuntu mới hoặc Debian mới. Nó giúp kiểm tra, cấu hình, và quản lý DNS runtime mà không cần chỉnh trực tiếp file /etc/resolv.conf hay khởi động lại dịch vụ.
1. Các chức năng 
```
# Xem DNS trên tất cả interface
resolvectl status

# Cấu hình DNS tạm thời cho interface ens33
resolvectl dns ens33 8.8.8.8 8.8.4.4

# Thêm domain search
resolvectl domain ens33 example.com

# Xóa cache DNS
resolvectl flush-caches

# Truy vấn google.com
resolvectl query google.com
```
Khi nào dùng resolvectl: 
+ Khi muốn thay đổi DNS tạm thời mà không sửa /etc/resolv.conf.
+ Khi cần kiểm tra cache hoặc trạng thái DNS theo interface.
+ Khi hệ thống dùng systemd-resolved, bạn không nên chỉnh resolv.conf trực tiếp nữa.

Tóm lại:
resolvectl là “bảng điều khiển CLI” của systemd-resolved, giúp bạn: Kiểm tra trạng thái, Thay đổi DNS runtime, Quản lý cache, Query DNS

# Quá trình phân giải DNS như sau
1. Ứng dụng gọi DNS
Khi một ứng dụng (ví dụ curl hay ping) cần phân giải tên miền example.com, nó gọi resolver chuẩn.
Trên Debian/Ubuntu hiện đại, resolver này đọc /etc/resolv.conf để biết DNS server nào để hỏi.
Nếu dùng systemd-resolved, /etc/resolv.conf thường là symbolic link tới /run/systemd/resolve/stub-resolv.conf (hoặc /run/systemd/resolve/resolv.conf), trong đó thường có dòng:
`nameserver 127.0.0.53`
Đây chính là stub resolver địa phương của systemd-resolved.
2. Stub resolver
Stub resolver là một “trạm trung gian” trên localhost (127.0.0.53), nó không trực tiếp hỏi Internet, mà gửi truy vấn đến systemd-resolved (chạy như một daemon hệ thống).
Ưu điểm: ứng dụng không cần biết DNS server thật sự, systemd-resolved quản lý caching, DNSSEC, split-DNS, multi-interface DNS.
3. systemd-resolved xử lý truy vấn
systemd-resolved nhận truy vấn từ stub resolver.
Nó kiểm tra cache trước, nếu đã có IP domain thì trả về luôn.
Nếu chưa có, systemd-resolved sẽ gửi truy vấn đến DNS server cấu hình trong /etc/systemd/resolved.conf hoặc các DNS server lấy từ network manager / interface (VD: DHCP).
4. Kết quả trả về
systemd-resolved nhận IP từ upstream DNS server.
Nó cập nhật cache và trả kết quả cho stub resolver.
Stub resolver trả IP về ứng dụng.

Tóm tắt luồng: App -> /etc/resolv.conf -> 127.0.0.53 (stub resolver) 
   -> systemd-resolved (daemon) -> upstream DNS server 
   -> trả IP về app
   
# Lưu í
Systemd-resolved và resolvectl chỉ bật mặc định trên ubuntu/debian và RHEL 9+ trở lên
Còn RHEL 7/8 thì network manager sẽ chỉ định thẳng DNS server luôn
Thứ tự ưu tiên khi lấy DNS
2. Systemd-resolved lấy DNS từ 3 nguồn theo thứ tự ưu tiên:
Ưu tiên	Nguồn DNS	Ví dụ bạn đang cấu hình
1	DNS của interface từ Netplan (NetworkManager/systemd-networkd)	8.8.8.8 từ Netplan
2	DNS cấu hình trong /etc/systemd/resolved.conf	192.168.1.10
3	Fallback DNS của resolved	Ví dụ: 1.1.1.1



# Đổi DNS server trên ubuntu/debian
Sua ip nameserver trong netplan hoac DNS= trong /etc/systemd/resolved.conf
```
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - 192.168.174.12/24
      gateway4: 192.168.174.2
      nameservers:
        addresses:
          - 192.168.174.141   # Windows DNS Server
        search:
          - mylab.local
```
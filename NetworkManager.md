Trong NetworkManager, trạng thái managed và unmanaged dùng để chỉ NetworkManager có đang quản lý một interface mạng hay không. Giải thích chi tiết:

1. Managed

Nếu một interface là managed, nghĩa là NetworkManager quản lý và có thể cấu hình, bật/tắt, gán IP cho interface đó.

Thường thấy trạng thái này trong lệnh:

`nmcli device status`


Ví dụ:
```
DEVICE  TYPE      STATE      CONNECTION
ens33   ethernet  connected  ens33
```
ens33 là managed, NM biết interface này, biết connection gắn vào, có thể bật/tắt, thay đổi IP.
Điểm quan trọng:
Khi interface managed, bạn có thể dùng nmcli, nmtui để cấu hình.
Khi boot, NM sẽ tự động apply các connection cho interface này.

2. Unmanaged
Nếu một interface là unmanaged, nghĩa là NetworkManager không quản lý interface đó.
Lý do phổ biến:
Interface được khai báo tĩnh trong /etc/network/interfaces (Debian/Ubuntu) → NM mặc định bỏ qua.
Bạn cấu hình NM để bỏ qua một interface bằng /etc/NetworkManager/NetworkManager.conf:
```
[keyfile]
unmanaged-devices=interface-name:ens36
```
Interface là virtual device mà NM không hỗ trợ.
Khi unmanaged, nếu bạn chạy:
`nmcli device status`

Bạn sẽ thấy:
```
DEVICE  TYPE      STATE      CONNECTION
ens36   ethernet  unmanaged  --
```
Không thể dùng nmcli con up ens36, NM sẽ báo lỗi unknown connection.

Bạn phải dùng công cụ khác (ip addr, ifconfig, systemd-networkd, v.v.) để cấu hình interface.

3. Cách chuyển trạng thái từ unmanaged -> managed
Bước 1: Xác định interface đang unmanaged.
Bước 2: Kiểm tra NetworkManager.conf → xóa unmanaged-devices nếu có.
Bước 3: Kiểm tra /etc/network/interfaces (Debian/Ubuntu) → xóa cấu hình tĩnh nếu muốn NM quản lý.
Bước 4: Khởi động lại NetworkManager.
Bước 5: Kiểm tra lại nmcli device status.
Bước 6: Tạo connection nếu chưa có → lên mạng.

Bước 1 kiểm tra 
`nmcli device status`
Ví dụ bạn thấy:
```
DEVICE  TYPE      STATE      CONNECTION
ens36   ethernet  unmanaged  --
```
Điều này nghĩa là NM không quản lý ens36. 
Bước 2: Kiểm tra file cấu hình NetworkManager
Debian/Ubuntu

Mở file:
sudo nano /etc/NetworkManager/NetworkManager.conf
Kiểm tra xem có mục [keyfile] và unmanaged-devices không:
```
[keyfile]
unmanaged-devices=interface-name:ens36
```
Nếu có ens36, xóa tên interface đó hoặc comment dòng.
CentOS/RHEL
File tương tự: /etc/NetworkManager/NetworkManager.conf
Cấu hình cũng giống trên.

Bước 3: Kiểm tra /etc/network/interfaces (chỉ Debian/Ubuntu)

Nếu interface được cấu hình tĩnh ở đây, NM sẽ bỏ qua:
cat /etc/network/interfaces
Ví dụ:
```
auto ens36
iface ens36 inet static
    address 192.168.1.50
    netmask 255.255.255.0
```
Nếu thấy interface ở đây → NM sẽ unmanaged.
Có 2 cách:
Xóa cấu hình ở đây → NM sẽ quản lý.
Giữ cấu hình static nhưng sử dụng nmcli tạo connection tương ứng (khuyến nghị cách 1 nếu muốn NM quản lý toàn bộ).

Bước 4: Khởi động lại NetworkManager
`sudo systemctl restart NetworkManager`
Hoặc trên CentOS/RHEL:
`sudo systemctl restart NetworkManager.service`

Bước 5: Kiểm tra lại trạng thái interface
`nmcli device status`

Nếu mọi thứ ổn:
```
DEVICE  TYPE      STATE      CONNECTION
ens36   ethernet  disconnected  --
```
STATE không còn là unmanaged nữa → NM đã quản lý interface.

Bước 6: Tạo và bật connection nếu cần
Nếu chưa có connection, tạo mới:
```
sudo nmcli con add type ethernet ifname ens36 con-name ens36
sudo nmcli con up ens36
```

Hoặc nếu muốn IP tĩnh:
```
sudo nmcli con add type ethernet ifname ens36 con-name ens36 ip4 192.168.1.50/24 gw4 192.168.1.1
sudo nmcli con up ens36
```

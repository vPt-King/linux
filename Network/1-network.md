# Networking trong môi trường RHEL

## network-scripts (legacy)
Dir: /etc/sysconfig/network-scripts/
Chứa các file có dạng: 
ifcfg-eth0
ifcfg-ens33
ifcfg-lo
Đây là cách cấu hình mạng truyền thống, từ thời CentOS 6 → CentOS 7.

Đặc điểm:
+ Cấu hình bằng file text “ifcfg”
+ Dễ sửa, rõ ràng
+ Khi sửa file cần restart service:
systemctl restart network
=> Không tự động xử lý nhiều thứ như kết nối wifi, bond, team…
network-scripts đã bị bỏ (deprecated).
Họ muốn mọi người chuyển sang NetworkManager.

### Đổi ip tĩnh trong network script
```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
sửa lại thành như sau:
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="none"            # đổi từ dhcp thành none
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="76a17a7b-0cc4-4044-a0be-3efd2f5a44c9"
DEVICE="ens33"
ONBOOT="yes"

IPADDR=192.168.50.10        # địa chỉ IP tĩnh bạn muốn đặt
NETMASK=255.255.255.0       # subnet mask
GATEWAY=192.168.50.1        # default gateway
DNS1=8.8.8.8                # DNS chính
DNS2=1.1.1.1                # DNS phụ

lưu lại và : sudo systemctl restart network
```


## NetworkManager và nmcli là gì (Cách mới)
NetworkManager là dịch vụ quản lý mạng hiện đại, mạnh hơn, linh hoạt hơn.

Lệnh CLI để làm việc với NetworkManager là: 
`nmcli`
`nmtui`
Đặc điểm:
+ Quản lý mọi thứ: Ethernet, Wi-Fi, Bonding, VLAN, Bridge…
+ Tự động phát hiện card mạng mới
+ Không cần sửa file thủ công (nhưng vẫn có thể sửa)
+ Thay config dùng:
`nmcli con modify`
+ Restart không làm mất kết nối

### Các command chính làm việc với nmcli
1. Xem tất cả device (card mạng)
`nmcli device status`

Xem chi tiết 1 device
`nmcli device show ens33`

Xem danh sách kết nối (connection profiles)
`nmcli connection show`

2. Bật / tắt NetworkManager
```
sudo systemctl restart NetworkManager
sudo systemctl stop NetworkManager
sudo systemctl start NetworkManager
```

3. Enable / Disable card mạng
Tắt card:
`nmcli device disconnect ens33`

Bật card:
`nmcli device connect ens33`

4. Xóa connection
`nmcli connection delete ens33`

Hoặc xoá bằng tên profile:
`nmcli connection delete "Wired connection 1"`

5. Tạo Bridge
```
nmcli con add type bridge con-name br0 ifname br0
nmcli con add type bridge-slave con-name br0-port1 ifname ens33 master br0
```
6. Tạo Bonding
```
nmcli con add type bond ifname bond0 mode active-backup
nmcli con add type bond-slave ifname ens33 master bond0
nmcli con add type bond-slave ifname ens34 master bond0
```

7. Restart lại 1 connection
nmcli con down ens33
nmcli con up ens33

8. Bật chế độ auto connect
nmcli con mod ens33 connection.autoconnect yes

9. Disable IPv6 (nếu không cần)
nmcli con mod ens33 ipv6.method disabled

10. Xem log NetworkManager
journalctl -u NetworkManager -f

11. Debug network
```
nmcli device status

nmcli -p con show ens33 (Xem profile applied)
```


## quan hệ giữa network-scripts và NetworkManager
NetworkManager vẫn đọc và ghi các file ifcfg trong network-scripts.

Nói cách khác:
network-scripts chứa file cấu hình mạng dạng ifcfg
và NetworkManager (nmcli/nmtui) dùng chính những file đó để hoạt động.

Cụ thể:
Khi bạn sửa file ifcfg → NetworkManager đọc lại → áp dụng
Khi bạn dùng nmcli để chỉnh → NM chỉnh ngược lại file ifcfg
Ví dụ:

Nếu bạn chạy:
`nmcli con mod ens33 ipv4.addresses 192.168.1.10/24`

Nó sẽ sửa file:
/etc/sysconfig/network-scripts/ifcfg-ens33
Thêm dòng:
IPADDR=192.168.1.10
PREFIX=24

⚠️ Nhưng có 1 lưu ý quan trọng!
🔥 Trên RHEL8/CentOS8/Oracle 8 trở lên

network-scripts không còn được bật mặc định.
Nếu bạn chạy:
`systemctl status network`
Có thể bạn thấy nó disabled hoặc not found.
Đường mạng chủ yếu do NetworkManager xử lý.

# Networking trong môi trường Debian
Mặc định hiện nay (Ubuntu 18+):
Netplan (yaml config)
systemd-networkd hoặc NetworkManager (tùy môi trường)

Netplan = công cụ cấu hình
systemd-networkd hoặc NetworkManager = công cụ thực thi

## Mối quan hệ giữa Netplan và NetworkManager/systemd-networkd

Netplan là lớp cấu hình (frontend)
File nằm tại: /etc/netplan/*.yaml

Vi du:
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

Bạn sửa file YAML → rồi apply: 
`sudo netplan apply`
Trong file YAML chỉ cần khai báo:
```
network:
  renderer: NetworkManager
```
hoặc:
```
network:
  renderer: networkd
```
Sau đó Netplan sẽ chuyển cấu hình sang backend
Backend 1: systemd-networkd (server, cloud)
Backend 2: NetworkManager (desktop, laptop)

## kiểm tra service backend nào đang chạy

`systemctl is-active NetworkManager`
`systemctl is-active systemd-networkd`
`networkctl status`
`nmcli device status (Nếu lệnh này trả về danh sách card và trạng thái → NetworkManager đang chạy)` 



# Add thêm card mạng vào máy ảo
Kiểm tra phần cứng đã nhận chưa
`ip link`

Kiểm tra Network Manager đã nhận card mạng mới chưa:
`nmcli connection show`
`nmcli device status`

Nếu nmcli device status vào bị thấy disconbection:
`sudo nmcli con add type ethernet ifname ens36 con-name ens36`
`nmcli con up ens36`

# Troubleshoot
## Add thêm card mạng nhưng nmcli không nhận
dùng `ip link` -> Vẫn hiển thị card mạng ens36
Kiểm tra network manager
`nmcli device status`
+ Nếu ens36 xuất hiện nhưng trạng thái “unmanaged” => file cấu hình cũ hoặc plugin keyfile không quản lý interface này. 
Fix: 
```
sudo nmcli device set ens36 managed yes
sudo systemctl restart NetworkManager
```
+ Nếu ens36 chưa xuất hiện: NetworkManager chưa load lại danh sách NIC.
run:
`sudo systemctl reload NetworkManager`
`sudo systemctl restart NetworkManager`
Nếu vẫn không thấy ENS36 → NIC ảo chưa được VMware “Connect”.

+ ens36 xuất hiện trong nmcli device status nhưng không có connection

`sudo nmcli con add type ethernet ifname ens36 con-name ens36`
`nmcli con up ens36`
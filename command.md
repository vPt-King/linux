# Các command hay sử dụng

## Tìm kiếm file
Tìm file theo tên (chính xác)
`find /path/to/search -name "ten_file"`

Tìm file không phân biệt hoa/thường
`find / -iname "abc.txt"`

Tìm thư mục theo tên
`find / -type d -name "myfolder"`

Tìm file rồi thực thi lệnh (xóa, copy…). Ví dụ: Xóa tất cả file .log
`find /var/log -name "*.log" -delete`




## compress, decompress

Gom file dạng tar
`tar -cvf tenfile.tar /path/to/folder_or_file`

Nén dạng .tar.gz (gzip)
`tar -czvf tenfile.tar.gz /path/to/folder_or_file`

Nén dạng .tar.bz2 (bzip2 – nén mạnh hơn)
`tar -cjvf tenfile.tar.bz2 /path/to/folder_or_file`

Giải nén .tar
`tar -xvf tenfile.tar`

Giải nén .tar.gz
`tar -xzvf tenfile.tar.gz`

Giải nén .tar.bz2
`tar -xjvf tenfile.tar.bz2`

Giải nén vào thư mục chỉ định:
`tar -xvf tenfile.tar -C /thu_muc_dich`

Nén zip 1 thư mục:
`zip -r tenfile.zip /path/to/folder`

Nén zip nhiều file:
`zip tenfile.zip file1 file2 file3`

Giải nén file zip:
`unzip tenfile.zip`

Giải nén vào thư mục chỉ định:
`unzip tenfile.zip -d /thu_muc_dich`


## Networking

### Show up down status
`ip link show ens33`

Up card mạng:
`sudo ip link set ens33 up`

### Với OS cũ dòng RHEL có thể dùng sysconfig
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

### Dùng network manager cho RHEL
Oracle Linux (giống RHEL/CentOS) dùng NetworkManager
1. Xem trạng thái
`nmcli device status`

Nếu ens33 là disconnected hoặc unavailable, thì kết nối lại:
`nmcli con up ens33`

Xem tất cả device (card mạng)
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

### Với dòng debian 
Vào trong file netplan cấu hình

kiểm tra service backend nào đang chạy

`systemctl is-active NetworkManager`
`systemctl is-active systemd-networkd`
`networkctl status`
`nmcli device status (Nếu lệnh này trả về danh sách card và trạng thái → NetworkManager đang chạy)` 
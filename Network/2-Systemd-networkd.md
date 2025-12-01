systemd-networkd là trình quản lý mạng (network manager) của hệ thống systemd. 
được dùng mặc định trong ubuntu
Nó dùng cấu hình dạng .network, .netdev, .link nằm trong:

/etc/systemd/network/        # Admin config
/run/systemd/network/        # Generated
/usr/lib/systemd/network/    # Package default

3 loại file:
```
Loại file	Mục đích
.network	Thiết lập IP, gateway, DNS, routing
.netdev	Tạo interface ảo: bond, bridge, vlan, vxlan...
.link	Đặt tên interface, thay đổi MAC, MTU...
```

Cách kiểm tra trạng thái:
```
systemctl status systemd-networkd
networkctl
```
networkctl là CLI để xem trạng thái interface:
`networkctl status eth0`

Trên ubuntu thì netplan quản lí cấu hình IP, còn systemd-resolved quản lí backend

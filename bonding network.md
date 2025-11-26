# Boding (Nic Teaming)
Bonding trong Linux là kỹ thuật gộp nhiều card mạng (NIC) lại thành một interface duy nhất, nhằm tăng tốc độ, dự phòng, hoặc chia tải.

Nó còn được gọi là:
+ NIC Teaming
+ Link Aggregation
+ LAG

Mục đích: 
+ Tăng tính sẵn sàng (High Availability) 
+ Tăng băng thông
Nếu bạn có 2 NIC 1Gbps → bonding có thể tạo ra kết nối 2Gbps (tuỳ mode).
+ Load balancing (chia tải)
Phân phối lưu lượng lên nhiều NIC để giảm bottleneck.

## Cách bonding hoạt động
Bạn tạo một interface ảo:
`bond0`

Sau đó thêm các NIC thật vào làm slave:
```
ens33 → slave của bond0  
ens34 → slave của bond0  
```
Ứng dụng và hệ thống chỉ thấy bond0
→ hệ thống không biết bạn có nhiều NIC phía sau.

## Các mode bonding
Linux hỗ trợ 7 mode bonding:

Mode	Tên	Tính năng	Switch cần config?
0	balance-rr	Round robin (tăng băng thông)	❗ Cần config LACP? Không, nhưng dễ lỗi
1	active-backup	HA (1 active, 1 standby)	❌ Không
2	balance-xor	Load balance theo MAC	✔️ Có
3	broadcast	Gửi ra tất cả NIC	❌ Không
4	802.3ad (LACP)	Chuẩn link aggregation	✔️ Switch phải bật LACP
5	balance-tlb	Load balance outbound	❌ Không
6	balance-alb	LB cả inbound/outbound	❌ Không

Thực tế doanh nghiệp dùng mode gì?
Mode 1 – active-backup → server HA (không cần cấu hình switch)
Mode 4 – LACP → server cần băng thông cao (2Gbps / 10Gbps / 40Gbps)
Lưu í khi cấu hình bonding là các card mạng thật ko được đặt ip, chỉ đặt ip cho bond0

## Cấu hình bonding (systemd-networkd)
File: /etc/systemd/network/bond0.netdev
```
[NetDev]
Name=bond0
Kind=bond

[Bond]
Mode=active-backup
```
Slave: /etc/systemd/network/ens33.network
```
[Match]
Name=ens33

[Network]
Bond=bond0
```

`sudo systemctl restart systemd-networkd`
## Cấu hình bonding với NetworkManager
Tạo bond:
`nmcli con add type bond ifname bond0 mode active-backup`

Add slave NIC:
```
nmcli con add type bond-slave ifname ens33 master bond0
nmcli con add type bond-slave ifname ens34 master bond0
```
Set IP:
```
nmcli con mod bond0 ipv4.method manual ipv4.addresses 192.168.1.10/24 ipv4.gateway 192.168.1.1
nmcli con up bond0
```
Kiểm tra bonding
`cat /proc/net/bonding/bond0`
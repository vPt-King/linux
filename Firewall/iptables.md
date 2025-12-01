iptables là công cụ firewall mặc định trên Linux (trước khi nftables ra đời).
Nó làm việc ở tầng Kernel (Netfilter) và cho phép:
+ Chặn / cho phép traffic
+ NAT (SNAT, DNAT)
+ Port forwarding
+ Lọc gói theo IP, port, protocol
+ Giới hạn kết nối (rate limit)

1. Các bảng mặc định trong iptables
Bảng	Chức năng
filter	Lọc gói (ALLOW/DENY) – bảng mặc định
nat	NAT địa chỉ (SNAT, DNAT, MASQUERADE)
mangle	Thay đổi header gói tin
raw	Bỏ qua connection tracking
security	Dùng trong SELinux

3. Các chain quan trọng
Bảng filter
INPUT → gói vào máy
OUTPUT → gói từ máy đi ra
FORWARD → gói đi qua máy (router)

Bảng nat
PREROUTING → sửa IP đích (DNAT)
POSTROUTING → sửa IP nguồn (SNAT / MASQUERADE)

4. Một số tag trong iptables
+ -t : chọn bảng , nết ko chọn bảng thì mặc định sẽ là filter
`iptables -t filter`

+ -A Append: Thêm rule vào cuối chain
`iptables -A INPUT ...`

+ -I Insert: Thêm rule vào đầu chain
`iptables -I INPUT ...`

+ -D delete: XÓA RULE
`iptables -D INPUT -p tcp --dport 22`

+ -P Policy: Đặt chính sách mặc định
`iptables -P INPUT DROP`

+ -p protocol: Xác định protocol
`-p tcp `
+ --dport: destination port
`--dport 80`

+ --sport: source port
`sport 443`

+ -s : source ip
`-s 192.168.10.1`

+ -d : destination ip
`-d 8.8.8.8`

+ -i : input interface
`-i eth0`

+ -o : output interface
`-o eth0`

+ -j ACCEPT → cho phép
+ -j DROP → chặn (âm thầm)
+ -j REJECT → chặn (gửi thông báo)
+ -j LOG → ghi log
+ -j DNAT → đổi IP đích
+ -j SNAT → đổi IP nguồn
+ -j MASQUERADE → NAT IP động

`iptables -A INPUT -p tcp --dport 22 -j ACCEPT`

7. Lưu iptables khởi động lại không bị mất
Ubuntu:
```
apt install iptables-persistent
iptables-save > /etc/iptables/rules.v4
```

RHEL / CentOS:
`service iptables save`


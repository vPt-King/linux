Kiểm tra bảng route
`ip route show`

Thêm route tạm thời
`sudo ip route add 10.10.10.0/24 via 192.168.1.1 dev ens33`
Gói đến mạng 10.10.10.0/24 sẽ đi qua gateway 192.168.1.1 qua interface ens33.

Xóa route:
`sudo ip route del 10.10.10.0/24`

Thêm route cố định
Cú pháp nmcli:
```
nmcli connection modify <PROFILE> +ipv4.routes "<DEST>/<PREFIX> <GATEWAY>"
nmcli connection up <PROFILE>
sudo systemctl restart NetworkManager
```

Ví dụ:

Mạng bạn muốn thêm: 10.10.10.0/24 qua gateway 192.168.1.1 trên profile ens33:
```
nmcli connection modify ens33 +ipv4.routes "10.10.10.0/24 192.168.1.1"
nmcli connection up ens33
sudo systemctl restart NetworkManager
```

Dấu + trước ipv4.routes để append route mà không xóa các route khác.

Xoa route co dinh:
Cu phap:
```
nmcli show connection <PROFILE>
nmcli connection up <PROFILE>
sudo systemctl restart NetworkManager
```

Tim dong ipv4.route , chon route can xoa:
`nmcli connection modify <PROFILE> -ipv4.routes "10.10.10.0/24 192.168.1.1"`

1. Các câu lệnh với user
```
sudo useradd thanh (Tạo thêm user Thanh)
sudo useradd -m thanh (Tạo user và home directory)
sudo useradd -m -s /bin/bash thanh (Tạo user và shell)
sudo passwd thanh (Đặt password cho user)
id thanh (Xem group user)
sudo userdel thanh (Xóa user thanh)
sudo userdel -r thanh (Xóa home + mail)
```

2. Câu lệnh trung cấp user
```
sudo usermod -s /bin/zsh thanh (đổi shell mặc định user)
sudo usermod -d /data/thanh -m thanh (Đổi home directory)
sudo usermod -l thanh2 thanh (đổi tên user ? có đổi cả home directory ko)
sudo usermod -u 2000 thanh (đổi UID)

# User
User là tài khoản dùng để đăng nhập vào hệ thống
Mỗi User gồm có: UID (UserID) , Home Directory (~/), Shell (sh, bash,...), Quyền (đọc/ghi/thực thi)

Có 3 loại user:
+ Root : (UID=0) quyền cao nhất
+ System user : (UID=1-999) User phục vụ cho dịch vụ (nginx, mysql…)
+ Normal User : (UID >= 1000) User bạn tạo để sử dụng

# Group
Group = nhóm user để quản lý quyền dễ hơn.
Ví dụ:
+ Nhóm admins chứa các admin.
+ Nhóm docker cho phép chạy Docker.
+ Nhóm dev chứa các lập trình viên.

Mỗi group có:
+ GID (Group ID)
+ Danh sách user thuộc group

Một user có thể:
+ Thuộc 1 primary group
+ Và nhiều secondary groups

## Primary Group
Nhóm mặc định của User
+ Nếu tạo 1 user là `thanh` thì Linux sẽ tự tạo group `thanh`
+ Khi User tạo file mới thì nhóm mặc định là User

## Secondary Group
Các nhóm mà user được thêm thêm để có thêm quyền.

Ví dụ bạn muốn user `thanh` có quyền chạy Docker:
`sudo usermod -aG docker thanh`
→ docker là secondary group.

# Kiểm tra thông tin user và group
`id thanh`

Xem danh sách group
`getent group`

# Các file quan trọng chứa User&Group
```
File	            Ý nghĩa
/etc/passwd	        Danh sách user + UID + shell
/etc/shadow     	Password hash của user
/etc/group	        Danh sách group + GID
/etc/gshadow	    Mật khẩu group
```

Ví dụ một dòng trong /etc/passwd:
`thanh:x:1001:1001::/home/thanh:/bin/bash`

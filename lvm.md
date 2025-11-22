# Why using LVM
+  Phân vùng động: Với LVM, bạn có thể dễ dàng thay đổi kích thước của phân vùng (tăng hoặc giảm) mà không cần phải khởi động lại hệ thống hoặc xóa dữ liệu.
Ví dụ: Khi không gian đĩa của một phân vùng hết, bạn có thể mở rộng nó bằng cách thêm dung lượng từ Volume Group.
+  Gộp nhiều ổ đĩa: LVM cho phép bạn gộp nhiều ổ đĩa vật lý (Physical Volumes) thành một nhóm (Volume Group), sau đó chia nhóm này thành các phân vùng logic (Logical Volumes). Điều này rất hữu ích nếu bạn cần một phân vùng lớn hơn kích thước của một ổ đĩa đơn lẻ.

# Tìm hiểu lí thuyết
## Physical volume
PV: là cấp cơ bản nhất trong hệ thống lưu trữ LVM. Nó đại diện cho các thiết bị lưu trữ vật lý, như ổ cứng, phân vùng ổ cứng, hoặc ổ đĩa ảo, được chuẩn bị để sử dụng trong LVM. Một hoặc nhiều PV có thể được gộp lại thành một Volume Group (VG), từ đó tạo ra các Logical Volumes (LV).
Thiết bị làm PV:
Một ổ đĩa vật lý hoàn chỉnh (ví dụ: /dev/sdb).
Một phân vùng trên ổ đĩa (ví dụ: /dev/sda1).
Một thiết bị ảo (ví dụ: ổ đĩa được cung cấp bởi RAID hoặc iSCSI).
Để hiển thị thông tin các PV trong linux dùng câu lệnh:

`pvdisplay`


Ý nghĩa các thông tin của Volume group khi chạy lệnh vgdisplay:
- PV Name: Tên Physical volume
- VG Name: Tên volume group sử dụng
- PV Size: Dung lượng tổng của PV
- Allocatable: Trạng thái có cấp dung lượng cho PV 
- PE size: Physical extent PE là dung lượng nhỏ nhất mà PV có thể phân chia, mặc định trong LVM là 4MB
- Total PE: Tổng PE hiện tại
- Free PE: PE còn trống
- Allocated PE: PE đã dung
- PV UUID: uuid của PV

## Volume Group
Volume Group (VG) trong LVM (Logical Volume Manager) là lớp trung gian giữa Physical Volumes (PV) và Logical Volumes (LV). Nó là một tập hợp các Physical Volumes gộp lại để tạo thành một không gian lưu trữ logic duy nhất, từ đó bạn có thể tạo ra các Logical Volumes để sử dụng.
VG đóng vai trò như một "bể lưu trữ" (storage pool), kết hợp nhiều Physical Volumes (PV) thành một không gian logic lớn hơn.

Không gian này sau đó được chia nhỏ thành các Logical Volumes (LV) mà hệ điều hành và người dùng có thể sử dụng như các phân vùng ổ đĩa truyền thống.
Physical Extents (PE): VG chia toàn bộ dung lượng thành các khối nhỏ gọi là PE (Physical Extents). PE là đơn vị nhỏ nhất mà VG sử dụng để quản lý không gian.
Để liệt kê volume group ta dùng câu lệnh:

`vgdisplay`
+ VG name: Tên Volume group
+ Format: lvm2
+ Max LV: số lượng logical volume max
+ Cur LV: Số lượng logical volume hiện tại
+ Max PV: số lượng physical volume max
+ Cur PV: Số lượng physical volume hiện tại
+ Act PV: Số lượng physical volume đang hoạt động
+ VG size: Dung lượng volume group
+ PE size: physical extent dung lượng nhỏ nhất mà VG sẽ chia nhỏ PV
+ Total PE: Tổng PE hiện tại
+ Alloc PE: Tổng số PE đã cấp
+ Free PE: Tổng PE còn trống


## Logical Volume
Logical Volume (LV) trong LVM (Logical Volume Manager) là lớp trên cùng, được tạo ra từ không gian của Volume Group (VG), và hoạt động như một phân vùng lưu trữ mà người dùng hoặc hệ điều hành có thể sử dụng để lưu dữ liệu.
+ LV đóng vai trò tương tự như một phân vùng đĩa truyền thống (như /dev/sda1), nhưng có tính linh hoạt cao hơn.
+ Bạn có thể tăng hoặc giảm kích thước LV mà không cần xóa hoặc định dạng lại.
+ LV có thể được định dạng với hệ thống tệp (như ext4, xfs) và gắn vào hệ thống dưới dạng thư mục (ví dụ: /home, /var).
+ LV cũng có thể được sử dụng làm không gian swap.
Để hiển thị thông tin của logical volume ta dùng câu lệnh:
`lvdisplay`

LV Path: đường dẫn đến logical volume
LV name: Tên lv
VG name: Tên vg thuộc về
LV size: dung lượng của lv

# LVM Device
## Bản chất LVM
Linux dùng device mapper (dm-X) làm thiết bị thật, và symbolic links (symlink) được tạo ra để cung cấp tên “ổn định – dễ đọc” cho người dùng.
Hai thứ này cùng tồn tại

## Device mapper (/dev/dm-X)
LVM hoạt động dựa trên Device Mapper — một lớp trong kernel quản lý các thiết bị ảo:

Khi bạn tạo một LV, kernel tạo ra một device như:

{
    /dev/dm-0
    /dev/dm-1
    /dev/dm-2
}
Đây mới là thiết bị thật mà kernel quản lý.
Nhưng tên /dev/dm-0 không ổn định – mỗi lần boot có thể đổi, nên không dùng trực tiếp.

## Để dễ dùng, LVM tạo ra SYMLINK ỔN ĐỊNH
Vì /dev/dm-X không ổn định nên LVM tạo ra các liên kết tượng trưng (symbolic link) có tên theo VG/LV, ví dụ:

/dev/vg01/lvdata

/dev/ubuntu-vg/root

Đây là link ổn định → dùng cho:

mount

fstab

script

backup

Bạn nên dùng dạng này.

## Các symlink được đặt ở đâu?
A. Thư mục theo VG/LV:
/dev/<vg-name>/<lv-name>

Ví dụ:
/dev/ubuntu-vg/root

B. Thư mục /dev/mapper:
/dev/mapper/<vg-name>--<lv-name>

Ví dụ:
/dev/mapper/ubuntu--vg-root

Lưu ý:
Dấu - trong tên VG hoặc LV được escape thành -- để phân biệt.
Đây cũng chỉ là symlink, không phải thiết bị thật.

## Sơ đồ mối quan hệ
`/dev/ubuntu-vg/root  ----->  /dev/mapper/ubuntu--vg-root  ----->  /dev/dm-0 (device thật)`

Giải thích:

/dev/dm-0
→ Thiết bị thật (device mapper)

/dev/mapper/ubuntu--vg-root
→ symlink do device-mapper tạo ra

/dev/ubuntu-vg/root
→ symlink “đẹp – dễ đọc”, do udev/LVM tạo

## Khi mount nên dùng cái nào
Bạn nên dùng:
/dev/vgname/lvname

Hoặc:
/dev/mapper/vgname--lvname

# Thực hành LVM
## Tạo LVM
Các bước tạo LVM: Physical volume > Volume group > Logical volume

Tạo physical volume
```
parted /dev/sdb
(parted) print
(parted) mklabel gpt
(parted) unit GB
(parted) mkpart primary 0GB 20GB (dung lượng phân vùng)
(parted) print
(parted) set 1 lvm on
(parted) print
(parted) quit

Sau khi tạo xong phân vùng /dev/sdb1
pvcreate /dev/sdb1 
pvscan
vgcreate vg_data /dev/sdb1 
vgdisplay 
 + tạo logical volume (nằm trong volume group vg_mail):
lvcreate --name lv_data --size 70G vg_data
or lvcreate --name lv_data -l 100%FREE vg_data
 + định dạng lại logical volume:
mkfs.xfs (mkfs.ext4) /dev/vg_data/lv_data
 + mount thư mục cần sử dụng tới logical volume:
mount /dev/vg_data/lv_data /app
 + chỉnh file fstab để tự nhận logical volume khi boot:
vim /etc/fstab
/dev/vg_data/lv_data  /app                      xfs (ext4)      defaults        0 0
```

## Extend 1 volume group
```
    vgextend vg_test /dev/sdc1`
    lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
    df -Th ( Check ext4 hay xfs)
    resize2fs /dev/ubuntu-vg/ubuntu-lv hoac xfs_growfs /dev/mapper/rhel-root
```

## Xóa phân vùng có sẵn
`wipefs -a /dev/sdb`
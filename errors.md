--------------
E: dpkg was interrupted, you must manually run 'sudo dpkg --configure -a' to correct the problem.
Nguyen nhan: nghĩa là quá trình cài đặt trước đó bị gián đoạn (mất điện, kill process, đóng terminal…).

fix: 
+ sudo dpkg --configure -a
+ sudo apt update


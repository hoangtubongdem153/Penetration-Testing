
![Screenshot 2024-02-22 093905](https://hackmd.io/_uploads/H1kjGI4hp.png)
![Screenshot 2024-02-22 094303](https://hackmd.io/_uploads/r1Bsz8Nn6.png)
Đây là một thử thách trên về `NetSec` trên `Tryhackme`. Sau khi Start Machine , ta được cung cấp một `IP` của máy chủ: `10.10.170.39` (kết nối đường mạng của Tryhackme qua `OpenVPN`)
![Screenshot 2024-02-22 121052](https://hackmd.io/_uploads/B18uw8V3T.png)

![Screenshot 2024-02-22 121410](https://hackmd.io/_uploads/H1Y4OIVna.png)

1. *What is the highest port number being open less than 10,000?*
Thực hiện quét mạng cơ bản sử dụng `nmap` : `nmap 10.10.170.39` , ta được kết quả sau: 
![Screenshot 2024-02-22 121556](https://hackmd.io/_uploads/r1pyKI4np.png)
*Answer: 8080*

2. *There is an open port outside the common 1000 ports; it is above 10,000. What is it?*
Theo mặc định, Nmap quét 1.000 cổng phổ biến nhất cho mỗi giao thức mà nó được yêu cầu quét. Do đó ta phải điều chỉnh phạm vi cổng được quét: 
![Screenshot 2024-02-22 122541](https://hackmd.io/_uploads/Sy2ksI436.png)
Ở đây time quét đến 90s , ta có thể sử dụng min-rate flag để tăng tốc độ quét :>>
*Answer: 10021*

3. How many TCP ports are open?
Kết quả đã có ở trên!
*Answer: 6*

4. *What is the flag hidden in the HTTP server header?*
Vì ta biết có cổng `http(80)` đang mở , truy cập qua browser và get Server Header:
![Screenshot 2024-02-22 123559](https://hackmd.io/_uploads/Hkd9p8436.png)
*Answer: THM{web_server_25352}*

5. *What is the flag hidden in the SSH server header?*
Thực hiện kết nối tới máy chủ với cổng `ssh(22)` sử dụng Telnet: 
![Screenshot 2024-02-22 131516](https://hackmd.io/_uploads/HJGj8D4np.png)
Ta thu được cờ THM!
*Answer: THM{946219583339}*

6. *We have an FTP server listening on a nonstandard port. What is the version of the FTP server?*

Chúng ta sử dụng cờ `-A` để thực hiện quét phát hiện OS và version, chỉ định cổng đáng ngờ `10021`: 
`nmap  -p10021 -A  10.10.170.39 `

![Screenshot 2024-02-22 133502](https://hackmd.io/_uploads/rycBjDNh6.png)
*Answer: vsftpd 3.0.3*

7. *We learned two usernames using social engineering: eddie and quinn. What is the flag hidden in one of these two account files and accessible via FTP?*

Biết được 2 người dùng là `eddie` và `quinn`. Kết nối FTP với 2 tài khoản trên thử xem: 
![Screenshot 2024-02-22 134124](https://hackmd.io/_uploads/B1S_6D43a.png)
Ta cần tìm mật khẩu của `eddie` và `quinn`,thử sử dụng `hydra`(a very fast network logon cracker which supports many different services) để tấn công từ điển trên máy chủ ftp trên port 10021 xem sao:
`hydra -l eddie -P /usr/share/wordlists/rockyou.txt 10.10.170.39 ftp -s 10021`
Trong đó: 
* `-l` : chỉ định tên đăng nhập người dùng
* `-P` : password list được dùng để dictionary attack
* `-s` : chỉ định port service được sử dụng (ftp:10021)
Enter and...
![Screenshot 2024-02-22 135539](https://hackmd.io/_uploads/SyfKgOE3p.png)
*eddie: jordan*
Tương tự:  
![Screenshot 2024-02-22 135655](https://hackmd.io/_uploads/ByGiluN2a.png)
*quinn: andrea*
Đăng nhập ftp server với quinn ta thấy được file *ftp_flag.txt* , get it to local! 
![Screenshot 2024-02-22 140147](https://hackmd.io/_uploads/r1B3WONnT.png)
![Screenshot 2024-02-22 140221](https://hackmd.io/_uploads/BJEWMON2p.png)
*Answer: THM{321452667098}*
All done! 🫠🫠🫠




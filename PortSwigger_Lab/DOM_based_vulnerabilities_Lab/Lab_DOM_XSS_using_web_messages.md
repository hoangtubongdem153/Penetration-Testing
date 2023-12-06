### Web Academy PortSwigger
* Lab: [DOM XSS using web messages](https://portswigger.net/web-security/dom-based/controlling-the-web-message-source/lab-dom-xss-using-web-messages)
* Descript: This lab demonstrates a simple web message vulnerability.
* To solve this lab, use the exploit server to post a message to the target site that causes the print() function to be called.
* Solved and Writeup by Me ^^!  
* 
![Ảnh chụp màn hình 2023-12-06 100315](https://hackmd.io/_uploads/ry0PU5prp.png)

Đọc qua mô tả đề bài ,nhiệm vụ là ta cần gọi hàm print() trên trang web.
Access the lab, ta có giao diện gian hàng như dưới:  

![Ảnh chụp màn hình 2023-12-06 131457](https://hackmd.io/_uploads/B1GiI9pH6.png)

---
Xem qua một sản phẩm : 

![Ảnh chụp màn hình 2023-12-06 131943](https://hackmd.io/_uploads/S1Gbw5pBT.png)

---
![Ảnh chụp màn hình 2023-12-06 131615](https://hackmd.io/_uploads/SJ3ZvqpH6.png)
Ta có thể thấy trang web ở dạng chỉ đọc , ta không thấy đầu vào (input) có thể có trên web để có thể inject script.
Xem qua source trang , ta phát hiện đoạn script sau:

![Ảnh chụp màn hình 2023-12-06 131717](https://hackmd.io/_uploads/SJOSOqTSp.png)

Đoạn script sử dụng hàm addEventListener() để thêm một hàm thực thi function() khi có sự kiện message xảy ra.

*Note: Sự kiện 'message' là một sự kiện trong JavaScript được kích hoạt khi một trang web nhận được một thông điệp từ một cửa sổ khác thông qua phương thức postMessage. Đây là cách trang web có thể gửi và nhận dữ liệu giữa các cửa sổ hoặc iframe mà không cần làm mới trang.*

Giá trị của tin nhắn message (e) sẽ được xử lý và gán vào thẻ div có id='ads'. Do sử dụng innerHTML (dễ bị tấn công XSS), ta lên ý tưởng chèn script vào message được gửi đến!

Ta xây dựng mã khai thác sử dụng iframe để gửi thông điệp đến cửa sổ của trang web :  

![Ảnh chụp màn hình 2023-12-06 133157](https://hackmd.io/_uploads/r1kci5arp.png)

Giải thích sơ qua , ta xây dựng trang html , sử dụng iframe để nhúng trang web mục tiêu. Sử dụng this.contentWindow (tham chiếu đến iframe được nhúng) và phương thức postMessage() để gửi message (đã chèn script ) đến mục tiêu.
**
Hit Enter Deliver exploit to victim.... End.....

![Ảnh chụp màn hình 2023-12-06 133220](https://hackmd.io/_uploads/r1Nx65TBT.png)

Lab được giải quyết , cửa sổ in xuất hiện !

![Ảnh chụp màn hình 2023-12-06 140556](https://hackmd.io/_uploads/H1FFpcpHp.png)



 




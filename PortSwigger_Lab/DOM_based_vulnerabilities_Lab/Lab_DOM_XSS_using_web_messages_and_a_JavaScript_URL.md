### Web Academy PortSwigger
* Lab: [DOM XSS using web messages and a JavaScript URL](https://portswigger.net/web-security/dom-based/controlling-the-web-message-source/lab-dom-xss-using-web-messages-and-a-javascript-url)
* Descript: This lab demonstrates a DOM-based redirection vulnerability that is triggered by web messaging.
*  To solve this lab, construct an HTML page on the exploit server that exploits this vulnerability and calls the print() function.
* Solved and Writeup by Me ^^!
  
![Ảnh chụp màn hình 2023-12-06 192946](https://hackmd.io/_uploads/ByGM01CSp.png)

Hit Accessss, Truy cập lab , ta có giao diện như dưới:  

![Ảnh chụp màn hình 2023-12-06 193111](https://hackmd.io/_uploads/ByDV0JAHa.png)

Đọc qua mô tả đề bài , nhiệm vụ là tìm ra lổ hổng để gọi hàm `print()`. Sau khi test qua phần comment trong post , không thấy khả thi để chèn scritp.
Xem qua source trang , ta thấy đoạn script sau:  

![Ảnh chụp màn hình 2023-12-06 193201](https://hackmd.io/_uploads/Syhq0kCST.png)

Phân tích qua đoạn script , đoạn script trên thêm một `event` khi xảy ra việc nhận `message` , sẽ kích hoạt `function()` với tham số là tin nhắn `e` , sau đó kiểm tra nội dung của tin nhắn để xác định có chứa `http:` hay `https:` . Nếu điều kiện đúng sẽ sử dụng `location.href` chuyển hướng trang web!

Tiến hành xây dựng iframe để gửi message đến trang web trên: 

![Ảnh chụp màn hình 2023-12-06 194646](https://hackmd.io/_uploads/Sy_CJeCBa.png)

Ta sử dụng iframe để nhúng trang web mục tiêu , sử dụng `postMessage` để gửi thông điệp (message) qua `contentWindow` (refer to this iframe) , vì có điêù kiện kiểm tra `https:` nhưng không xét đến vị trí trong tin nhắn , ta sử dụng payload sau :  ` javascript:print()//https:`

Hit deliver exploit to victim ... 
End.....
![Ảnh chụp màn hình 2023-12-06 194709](https://hackmd.io/_uploads/ByNg-gABT.png)

Lab đã được giải quyết ,view exploit,  cửa sổ in đã xuất hiện!

![Ảnh chụp màn hình 2023-12-06 194748](https://hackmd.io/_uploads/rkvmbxCH6.png)


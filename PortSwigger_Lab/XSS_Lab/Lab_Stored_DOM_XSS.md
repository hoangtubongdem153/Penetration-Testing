### Web Academy PortSwigger
* Lab: [Lab: Stored DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-stored)
* Descript: This lab demonstrates a stored DOM vulnerability in the blog comment functionality.
* To solve this lab, exploit this vulnerability to call the alert() function.
* Solved and Writeup by Me ^^!
![Ảnh chụp màn hình 2023-11-04 102015.png](https://hackmd.io/_uploads/Bkjt_4Xma.png)

Truy cập trang Lab , giao diện trang web hiện ra như sau : 

![Ảnh chụp màn hình 2023-11-04 103520.png](https://hackmd.io/_uploads/H1Tb2VmQp.png)

Vì đề bài gợi ý vuln stored DOM trong chức năng comment ,ta nhấp vào môt bài post bất kỳ và xem : 

![Ảnh chụp màn hình 2023-11-04 103443.png](https://hackmd.io/_uploads/BJUn24mQa.png)

Đây là nội dung bài post cho ai muốn biết ^_O (Đã dịch :>)

![Ảnh chụp màn hình 2023-11-04 104122.png](https://hackmd.io/_uploads/SkDR6EQQ6.png)

Lướt xuống cuối trang , ta sẽ thấy mục comment và các comment trước đó đã được load 


![Ảnh chụp màn hình 2023-11-04 104450.png](https://hackmd.io/_uploads/SyoBA4Xm6.png)


Xem qua source trang, ta thấy có vẻ như có file xử lý việc tải comment ,

![Ảnh chụp màn hình 2023-11-04 102509.png](https://hackmd.io/_uploads/HycYREQma.png)

Hãy thử tạo một vài bình luận và xem : 

![Ảnh chụp màn hình 2023-11-04 110115.png](https://hackmd.io/_uploads/HkmdMrmXT.png)

Kết quả hiện thị , mình không hiểu tại sao thẻ đóng `</script>` bị mất , thẻ `img` vẫn  hiện thị chính xác, thẻ img vẫn nằm trong thẻ `p` , thử chèn thẻ đóng để break out ra thử :` </p><img src=x onerror="alert()" > `

![Ảnh chụp màn hình 2023-11-04 110842.png](https://hackmd.io/_uploads/r1z-NrXmT.png)

---
![Ảnh chụp màn hình 2023-11-04 110915.png](https://hackmd.io/_uploads/r1OWEHQ7T.png)

Thông báo được bật lên khi load trang, bài lab được giải quyết . Nhưng một thắc mắc là ta có thể break out dễ như vậy mà không có hạn chế nào ¯\_(ツ)_/¯ 
Xem qua file js được tìm thấy trong source lúc nãy :  

![Ảnh chụp màn hình 2023-11-04 111218.png](https://hackmd.io/_uploads/SkybSHmQT.png)

Có thể trang lab đã load không đúng nên đã không lọc nội dung của cmt được tải lên. Dù sao , hãy phân tích nó!

Mô tả ngắn gọn về chức năng , hàm `loadComments` thực hiện tạo và gửi request thông qua Object `XMLHttpRequest` (Đối tượng được sử dụng để yêu cầu dữ liệu từ máy chủ web,có thể cập nhật nội dung trang web mà không cần load trang ) 

Phương thức `Json.parse`,  được sử dụng để chuyển đổi dữ liệu phản hồi từ request dạng `json` thành mảng các object javascript , sau đó gán vào biến `comments` , làm tham số cho `function` hiển thị nội dung `displayComments()` 

Ta chú ý hàm `displayComments()` sử dụng `escapeHTML()` để xử lý chuyển đổi `<` , `>` trong chuỗi thành `&lg;`, `&gt;`

---
![Ảnh chụp màn hình 2023-11-04 111356.png](https://hackmd.io/_uploads/SJwQrSmXT.png)

Chẳng hạn như `<script>` sẽ được chuyển đổi thành `&lg;script&gt;`

Do sử dụng `replace` chỉ thay thế các ký tự xuất hiện đầu tiên. Ta có thể thêm `<>` vào đầu scrip alert ! để  bypass hàm `escapeHTML()` này và thực thi thành công đoạn scrip alert rồi ¯\_(ツ)_/¯!

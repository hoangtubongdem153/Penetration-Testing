### Web Academy PortSwigger
* Lab: [Lab: Reflected DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-reflected)
* Descript: This lab demonstrates a reflected DOM vulnerability. Reflected DOM vulnerabilities occur when the server-side application processes data from a request and echoes the data in the response. A script on the page then processes the reflected data in an unsafe way, ultimately writing it to a dangerous sink.
* To solve this lab, create an injection that calls the alert() function.
* Solved and Writeup by Tung!

![Ảnh chụp màn hình 2023-11-03 202219.png](https://hackmd.io/_uploads/SJSAEuGQp.png)

Truy cập trang Lab ,ta thấy xuất hiện trang giao diện web như trên. Dựa trên mô tả Lab , trang web có lỗ hổng Reflected-XSS dựa trên DOM ,request được xử lý bên phía máy chủ và phản hồi lại cho người dùng.
Xem qua source code , ta thấy không có đoạn script nào trên trang nên có lẽ request được xứ lý phía máy chủ:  

![Ảnh chụp màn hình 2023-11-03 202257.png](https://hackmd.io/_uploads/Hy-d8_zQT.png)

Hãy thử search ngẫu nhiên một chuỗi:
![Ảnh chụp màn hình 2023-11-03 204410.png](https://hackmd.io/_uploads/B13TF_fXT.png)

---
![Ảnh chụp màn hình 2023-11-03 204811.png](https://hackmd.io/_uploads/S14B5OfmT.png)

Inspect trang , ta thấy có vẻ như kết quả trả về được xử lý ở một nơi khác - `searchResults.js`.

![Ảnh chụp màn hình 2023-11-03 210450.png](https://hackmd.io/_uploads/H1NwCdfmT.png)

Có vẻ như đây là file xử lý phản hồi được trả về từ phía máy chủ

Mở Burpsite, search bất kỳ : `aabc1233` (nghĩ bừa :>>) ,chặn request ta thấy `aabc1233` được đưa đến xử lý ở trang `/search-result?search=aabc1233`, không phải trang search request ban đầu `/?search=aabc1233`: 

![Ảnh chụp màn hình 2023-11-03 205342.png](https://hackmd.io/_uploads/B1X9ouGQp.png)

Response được trả về ở dạng `json`, gồm kết quả `results` và chuỗi tìm kiếm `searchTerm` . Theo mô tả gợi ý , script trên trang xử lý dữ liệu trả về và đưa vào phản hồi một cách không an toàn , ta nảy sinh ý tưởng break-out chuỗi trong `searchTerm` để chèn lệnh `alert()` , thử thêm chuỗi đóng `"` vào chuỗi và xem kết quả:  

![Ảnh chụp màn hình 2023-11-03 211119.png](https://hackmd.io/_uploads/Sy2JxYMX6.png)

Có vẻ như `"` đã được thoát , ta không thấy chuỗi đóng xuất hiện trong chuỗi kết quả ,thử thêm ký tự thoát `\` (mục đích hủy bỏ việc thoát ký tự) và xem phản hồi:   

![Ảnh chụp màn hình 2023-11-03 211624.png](https://hackmd.io/_uploads/Hy7kWtM76.png)

Ta thấy không có phản hồi trên trang web , tiếp tục chèn, ta nối thêm `alert()` sau khi break-out khỏi chuỗi search, không quên ghi chú `//` dấu đóng `"` đằng sau nó :  `aabc1233\"+alert()//` 

![Ảnh chụp màn hình 2023-11-03 213213.png](https://hackmd.io/_uploads/Bkz9EKz7T.png) 

Và Lab đã được giải quyết :>>

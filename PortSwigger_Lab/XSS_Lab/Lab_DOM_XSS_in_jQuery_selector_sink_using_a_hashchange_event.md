Web Academy PortSwigger
* Lab: [Lab: DOM XSS in jQuery selector sink using a hashchange event](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event)
* Descript: This lab contains a DOM-based cross-site scripting vulnerability on the home page. It uses jQuery's $() selector function to auto-scroll to a given post, whose title is passed via the location.hash property.
* To solve the lab, deliver an exploit to the victim that calls the print() function in their browser.
* Solved and Writeup by Tung!

---
![Ảnh chụp màn hình 2023-11-01 233555.png](https://hackmd.io/_uploads/ryYcRex7a.png)

---
Đọc mô tả bài lab , ta được biết trang có lỗ hổng DOM-based XSS trên trang , sử dụng selector function của `jQuery` để tự động cuộn tới post được yêu cầu! Xem qua source , đoạn dưới ta thấy đoạn script sau: 

---
![Ảnh chụp màn hình 2023-11-01 234236.png](https://hackmd.io/_uploads/B12LlbxQa.png)

---
Giải thích sơ qua , đoạn script trên sử dụng jQuery selector `$()` để xác định các phần tử sau đó ta có thể thao tác trên chúng . Ở đây `$(window).on('hashchange', function())` được sử dụng để thao tác với cửa sổ trình duyệt (URL) khi có sự kiện phần hash (dấu thăng # và mọi thứ sau nó) của URL thay đổi. Hãy thử thêm `#abcd12345` và xem kết quả!

---
![Ảnh chụp màn hình 2023-11-01 235648.png](https://hackmd.io/_uploads/Bkv67Ze7a.png)

---
Ta có thể thấy ,không có gì thay đổi cả! Phân tích tiếp đoạn script trên , ta thấy `var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
                            if (post) post.get(0).scrollIntoView();`

---
![Ảnh chụp màn hình 2023-11-02 000601.png](https://hackmd.io/_uploads/SypcHZe76.png)

---
Đoạn code trên lựa chọn tất cả các phần tử `h2` trong các phần tử `section` có `class` là "blog-list" trên trang web, và gán vào biến `post`.

---
![Ảnh chụp màn hình 2023-11-02 001421.png](https://hackmd.io/_uploads/HJFKPWeQp.png)

---
Và nếu tồn tại `post` ,trang web sẽ thực hiện cuộn đến vị trí của phần tử đó .Hãy thử thực hiện thay đổi phần hash thành `#The Digital Fairytale`

---
![Ảnh chụp màn hình 2023-11-02 001746.png](https://hackmd.io/_uploads/rkFLOWlmp.png)

---
Trang web đã cuộn đến "The Digital Fairytale" post!
Như vậy đoạn script đã thực thi, với yêu cầu đề bài thực hiện gọi hàm `print()` , ta thấy đoạn script không kiểm tra đoạn sau # nên ta thử chèn vào img để thực hiện hàm gọi `print()`: #<img src=xyz onerror=print()>

---
![Ảnh chụp màn hình 2023-11-02 002455.png](https://hackmd.io/_uploads/B1OGjWgmp.png)

---
![Ảnh chụp màn hình 2023-11-02 010250.png](https://hackmd.io/_uploads/ryKg7GlXT.png)

Đi tới trang deliver exploit , ta submit payload và lab đã được giải quyết !

---
![Ảnh chụp màn hình 2023-11-02 005704.png](https://hackmd.io/_uploads/H1STzfl76.png)


---
Ta vẫn có thể sử dụng `iframe` để thay thế!


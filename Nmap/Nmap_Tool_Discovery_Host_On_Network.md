# Discovering Live Hosts 
* ARP từ Lớp liên kết
* ICMP từ Lớp Mạng
* TCP từ lớp vận chuyển
* UDP từ lớp vận chuyển  

![745e0412b319d324352c7b29863b74f4](https://hackmd.io/_uploads/By8toGbsp.png)

# Sử dụng ARP từ lớp liên kết: 
Mục đích: gửi các frame đến địa chỉ quảng bá (broadcast) trên phân đoạn mạng và yêu cầu máy tính có địa chỉ IP cụ thể phản hồi bằng cách cung cấp địa chỉ MAC (phần cứng) của nó.
Chỉ có thể quét ARP nếu bạn ở trên cùng mạng con với hệ thống đích. Truy vấn ARP chỉ hoạt động nếu mục tiêu nằm trên cùng mạng con với chính bạn
Cú pháp: `nmap -PR (-sn) MACHINE_IP/24`
* Sử dụng -PR để chỉ định quét ARP 
* Sử dụng -sn để bỏ qua scanning port
![f0ce4cd34b827f529255c5c73bb909d1](https://hackmd.io/_uploads/SkNn1QZip.png)
Các máy hoạt động sẽ phản hồi bằng ARP Reply.
# Sử dụng ICMP (Internet control message protocol) từ lớp mạng:
1.Cách tiếp cận đơn giản nhất nhưng nó không phải lúc nào cũng đáng tin cậy: Ping Echo (-PE)
![f0ce4cd34b827f529255c5c73bb909d1](https://hackmd.io/_uploads/S1H4-7WiT.png)
 Quá trình quét này sẽ gửi các gói tiếng vang ICMP đến mọi địa chỉ IP trên mạng con, tuy nhiên, nên nhớ rằng nhiều tường lửa chặn ICMP Echo.
 
 2.Vì các yêu cầu ICMP echo có xu hướng bị chặn nên bạn cũng có thể xem xét các yêu cầu ICMP Timestamp  hoặc ICMP Address Mask để biết liệu hệ thống có đang trực tuyến hay không:
*  ICMP Timestamp
![image](https://hackmd.io/_uploads/rkcCGXZj6.png)
*  ICMP Address Mask
![image](https://hackmd.io/_uploads/SyJxmmbsp.png)

 
# Sử dụng giao thức TCP
1.TCP SYN Ping

Chúng tôi có thể gửi một gói có cờ SYN (Đồng bộ hóa) tới cổng TCP , 80 theo mặc định và chờ phản hồi. Một cổng mở sẽ trả lời bằng SYN/ACK (Xác nhận); một cổng đóng sẽ dẫn đến RST (Đặt lại). Trong trường hợp này, chúng tôi chỉ kiểm tra xem liệu chúng tôi có nhận được bất kỳ phản hồi nào để suy ra liệu máy chủ có hoạt động hay không.
Quy trình bắt tay 3 bước trong TCP:
![image](https://hackmd.io/_uploads/SkwzVXZjT.png)
TCP SYN Ping (sử dụng với quyền root):
Người root và sudoers có thể gửi các gói TCP SYN và không cần hoàn thành bắt tay 3 bước TCP ngay cả khi cổng mở (sử dụng RST flag để kết thúc kết nối TCP). Người dùng không có đặc quyền không có lựa chọn nào khác ngoài việc hoàn thành bắt tay 3 chiều nếu cổng được mở:
![image](https://hackmd.io/_uploads/SJ1ermbsa.png)
Máy chủ hoạt động sẽ trả về phản hồi `SYN,ACK`
2.Ping TCP ACK
Bạn phải chạy Nmap với tư cách là người dùng có đặc quyền để có thể thực hiện việc này. Nếu bạn thử với tư cách là người dùng không có đặc quyền, Nmap sẽ thử bắt tay 3 bước.
![image](https://hackmd.io/_uploads/Bku3B7Zja.png)
Máy chủ hoạt động trả về phản hồi `RST`.Các hệ thống không phản hồi sẽ ngoại tuyến hoặc không thể truy cập được.
# Sử dụng giao thức UDP

Cuối cùng, chúng ta có thể sử dụng UDP để khám phá xem máy chủ có trực tuyến hay không. Ngược lại với ping TCP SYN, việc gửi gói UDP đến một cổng mở sẽ không dẫn đến bất kỳ phản hồi nào. Tuy nhiên, nếu chúng tôi gửi gói UDP đến một cổng UDP đã đóng, chúng tôi sẽ nhận được gói không thể truy cập cổng ICMP.
![image](https://hackmd.io/_uploads/Sy8kP7bop.png)
![image](https://hackmd.io/_uploads/B1ylDQZip.png)

Hành vi mặc định của Nmap là sử dụng các máy chủ trực tuyến DNS ngược . Vì tên máy chủ có thể tiết lộ nhiều điều nên đây có thể là một bước hữu ích. Tuy nhiên, nếu bạn không muốn gửi các truy vấn DNS như vậy thì bạn có thể `-n` bỏ qua bước này.

Theo mặc định, Nmap sẽ tra cứu các máy chủ trực tuyến; tuy nhiên, bạn có thể sử dụng tùy chọn `-R` để truy vấn máy chủ DNS ngay cả đối với các máy chủ ngoại tuyến.

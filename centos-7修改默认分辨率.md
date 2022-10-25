# 

centos6
http://www.cnblogs.com/lost-1987/articles/3997970.html

修改文件`/boot/grub2/grub.cfg`
```bash
/ /vmlinuz-3.10.0-123.el7.x86_64 root=UUID=881ac4e6-4a55-47b1-b864-555de7051763 ro rd.lvm.lv=centos/swap vconsole.font=latarcyrheb-sun16 rd.lvm.lv=centos/root crashkernel=auto vconsole.keymap=us rhgb quiet LANG=en_US.UTF-8
```
添加如下,???具体看下表
```bash
vga=0x???

vga=0x340
```

![image-20200730233519861](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200730233519861.png)

重启后即可生效

---
title: grub2引导多系统
---
### 配置引导多系统
#### 探测其他操作系统
想要让 grub-mkconfig 探测其他已经安装的系统并自动把他们添加到启动菜单中，安装 软件包 os-prober 并 挂载 包含其它系统的磁盘分区。然后重新运行 grub-mkconfig。
#### 不能检测的系统
[处理配置文件](https://www.gnu.org/software/grub/manual/grub/html_node/Simple-configuration.html#Simple-configuration)将额外的自定义菜单项添加到列表末尾时，可以通过编辑完成 /etc/grub.d/40_custom 或创造 /boot/grub/custom.cfg。
``` bash
menuentry "Android"{
set root='(hd0,gpt10)'
linux /bliss-x86-11.11/kernel quiet root=/dev/sda10 		androidboot.selinux=permissive acpi_sleep=s3_bios,s3_mode SRC=/bliss-x86-11.11
initrd /bliss-x86-11.11/initrd.img
}

```
重新生成配置文件：
``` bash
# grub-mkconfig -o /boot/grub/grub.cfg
```
注意：不同的系统grub配置文件的目录可能不一样，可以通过命令查找：
``` bash
# sudo find / -name grub.cfg
```

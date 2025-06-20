# kernel-debug

## 本地环境
 * ubuntu22.04
 * ```md
   apt install build-essential libncurses-dev bison flex libssl-dev libelf-dev git-email -y 
   ```
 * ```md
   apt install qemu qemu-kvm qemu-system-x86
   
   qemu-system-x86_64 --version
   QEMU emulator version 6.2.0
   ```

## 编译

### 编译kernel
* `git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git` 克隆最新内核源码


* `cd linux`


* `make mrproper`可清理所有构建文件和配置重新开始


* `make x86_64_defconfig`生成默认的.config


* `make menuconfig`可配置调试参数等


* `make -j12`多线程编译, 最终生成`./arch/x86/boot/bzImage`和`./vmlinux`镜像文件 它们的区别似乎是**vmlinux**可调试


* `cp ./arch/x86/boot/bzImage /root/linux-demo`


***
### 编译busybox
* 从该链接`https://www.busybox.net/downloads/busybox-1.37.0.tar.bz2` 获取源文件  


* 解压`tar -xjf busybox-1.37.0.tar.bz2`


* `cd busybox-1.37.0`


* `make mrproper`可清理所有构建文件和配置，回归最初的状态


* `make menuconfig`配置，这里选上**Settings-Build static binary (no shared libs)**


* `make -j12`多线程编译, 接着执行`make install`,  最终在当前目录下生成**busybox**可执行文件


* `chmod +x busybox && cp busybox /root/linux-demo/initramfs_dir/bin`


* 制作init进程文件，`vim /root/linux-demo/initramfs_dir/init` 内容如下：
```md
#!/bin/busybox sh

/bin/busybox echo "Hello Linux"

/bin/busybox sh
```
* `chmod +x /root/linux-demo/initramfs_dir/init`


* 创建Makefile文件， `vim /root/linux-demo/Makefile`
```md
initramfs:
         cd ./initramfs_dir && find . -print0 | cpio -ov --null --format=newc | gzip -9 > ../initramfs.img

run:
   qemu-system-x86_64 \
            -kernel bzImage \
            -initrd initramfs.img \
            -m 1G \
            -nographic \
            -append "earlyprink=serial,ttyS0 console=ttyS0"
```


* `cd /root/linux-demo`


* `make initramfs`


* `make run` 通过qemu进入新终端，可以看到内核启动的一些日志打印，包括启动init进程，打印了我们上面的**Hello Linux**


* 按回车进入新终端,ls命令是没有的,可执行`busybox ls`
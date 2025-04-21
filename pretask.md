# OERV_Pretask
环境：

宿主机：Fedora 40

qemu 8.2.9

openEuler 24.03 LTS

## 任务一
> 通过 QEMU 仿真 RISC-V 环境并启动 openEuler RISC-V 系统，设法输出 fastfetch 结果并截图提交

编写脚本从qemu启动openEuler：
```
qemu-system-riscv64 \
    # 不使用图形界面
    -nographic -machine virt \
    # 虚拟CPU数量
    -smp 8 \
    # 虚拟机内存大小
    -m 4G \
    # 启动文件
    -bios fw_payload_oe_uboot_2304.bin \
    # 虚拟机硬盘的映像文件和格式
    -drive file=openEuler-24.03-V1-base-qemu-testing.qcow2,format=qcow2,id=hd0 \
    -object rng-random,filename=/dev/urandom,id=rng0 \
    -device virtio-gpu \
    -device virtio-rng-pci,rng=rng0 \
    -device virtio-blk-pci,drive=hd0 \
    -device virtio-net-pci,netdev=usernet \
    # 宿主机10000端口映射到虚拟机22端口，用于ssh连接
    -netdev user,id=usernet,hostfwd=tcp::10000-:22
```
repo源中没有fastfetch，从github下载fastfetch源码：
```
git clone https://github.com/fastfetch-cli/fastfetch.git
```
编译安装
```
cd fastfetch
mkdir build
cd build
cmake ..
make
sudo make install
```
运行fastfetch，结果如图：

![fastfetch](https://github.com/JoeyLYL/oerv_pretask/blob/main/images/fastfetch.png)

## 任务二
> 在 openEuler RISC-V 系统上通过 obs 命令行工具 osc，从源代码构建 RISC-V 版本的 rpm 包，比如 pcre2。

安装osc
```
dnf install osc build
```
在[OBS网页](https://build.tarsier-infra.isrc.ac.cn/)注册账号

编辑~/.oscrc修改配置文件
```
[general]
apiurl = https://build.tarsier-infra.isrc.ac.cn/
no_verify = 1
[https://build.tarsier-infra.isrc.ac.cn/]
user=userName
pass=passWord
```
新建本地空目录
```
osc checkout home:joeylyl
A    home:joeylyl

cd home:joeylyl
```
将需要修改软件包的相关配置文件（_service）下载到本地
```
osc co openEuler:24.03:SP1:Everything  pcre2
A    openEuler:24.03:SP1:Everything
A    openEuler:24.03:SP1:Everything/pcre2
_service: | Elapsed Time: 0:00:00                                              
A    openEuler:24.03:SP1:Everything/pcre2/_service
At revision 14.
```
将软件包远程代码同步到本地
```
cd openEuler:24.03:SP1:Everything/pcre2
osc up -S
```
删除_service文件，重命名源文件，删除从左边开始到最后一个:之间的内容，然后将重命名后的源文件添加到OBS暂存中
```
rm -f _service;for file in `ls | grep -v .osc`;do new_file=${file##*:};mv $file $new_file;done
osc addremove *
```
开始构建
```
osc build standard_riscv64  riscv64
```
构建完成如图所示：
![build_pcre2](https://github.com/JoeyLYL/oerv_pretask/blob/main/build_pcre2.png)

## 任务三
> 尝试使用 qemu user & nspawn 或者 docker 加速完成任务二

首先了解下qemu-user和nspawn
- 前面两个任务使用qemu-system模拟openEuler RISC-V内核，模拟的是RISCV64系统环境，这种模式类似虚拟机，需要模拟内核态，性能相对较弱；而qemu-usr只模拟用户态程序，使用的是宿主机内核，速度更快
- systemd-nspawn是一种轻量级容器工具，用于在隔离的环境中运行命令或完整的操作系统
使用qemu-user和systemd-nspawn可以在宿主机使用osc构建软件包，加速编译过程

安装所需工具（在Fedora宿主机中）
```
sudo dnf install qemu-user-static
sudo dnf install qemu-user-binfmt
sudo dnf install osc
```
在宿主机中使用osc工具同任务二一样拉取pcre2，然后开始构建，使用nspawn构建在osc build后加--vm-type=nspawn选项即可
```
osc build standard_riscv64 riscv64 --vm-type=nspawn
```
结果如图，可以看出：


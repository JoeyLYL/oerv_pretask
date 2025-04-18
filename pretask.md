# OERV_Pretask
环境：

Fedora 40

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
    #虚拟机硬盘的映像文件和格式
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


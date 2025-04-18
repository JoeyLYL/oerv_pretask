# OERV_Pretask
环境：
Fedora 40
qemu 8.2.9
openEuler 24.03 LTS
## 任务一
> 通过 QEMU 仿真 RISC-V 环境并启动 openEuler RISC-V 系统，设法输出 fastfetch 结果并截图提交
编写脚本从qemu启动openEuler
```

下载fastfetch源码
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

<div align="center">

#  Installation Guide to Real-Time Linux
<img width="400" src="/folder.png">
<img width="350" src="/RT.png">
</div>

This repository introduces 2 methods to build a real-time environment on your Linux OS. The kernel files in the figure above are not offered here since they are too large to upload in GitHub. You can contact Jia Xinyu (xinyu.jia@u.nus.edu) to obtain them. Aso, the installation of PCAN Driver in a Xenomai kernel is introduced below.

1. `PREEMPT_RT` - Ubuntu 16.04 / 18.04 / 20.04 LTS
2. `Xenomai 3.1` - Ubuntu 16.04 / 18.04 LTS

If you are new to real-time Linux, please read this [article](https://www.cnblogs.com/wsg1100/p/12822346.html) first. 

## 1. Installing PREEMPT RT patch
You can refer to these tutorials to configure and compile a kernel ([link_1](https://www.jianshu.com/p/b74b05d26cf9) / [link_2](https://blog.csdn.net/shenyage/article/details/102099198) / [link_3](https://blog.csdn.net/weixin_43455581/article/details/103899362)). Or you can directly install kernels below if your controller uses **Intel** or **AMD** processors, e.g. PC104/PCM-3365 or Intel NUC. These real-time kernels have only 2 differences with the normal kernel:

```
**Preemption Model**: Preemptible Kernel (Low-Latency Desktop) -> **Fully Preemptible Kernel (RT)**

**Timer Frequency**: 250 Hz -> **1000 Hz** 
```

* For Ubuntu 16.04 run:
```
sudo dpkg -i linux-image-4.13.13-ashwin-rt5_1.0_amd64.deb
sudo dpkg -i linux-headers-4.13.13-ashwin-rt5_1.0_amd64.deb
```
* For Ubuntu 18.04 run: (recommended)
```
sudo dpkg -i linux-image-5.4.69-rt39_5.4.69-rt39-1_amd64.deb
sudo dpkg -i linux-headers-5.4.69-rt39_5.4.69-rt39-1_amd64.deb
```
* For Ubuntu 20.04 run:
```
sudo dpkg -i linux-image-5.11.0-jiaxy-2021128-rt7_2021_amd64.deb
sudo dpkg -i linux-headers-5.11.0-jiaxy-2021128-rt7_2021_amd64.deb
```

**OTHERS**: If your controller uses **ARM** processors, e.g. NVIDIA Jeston Nano / TX / Xavier which requires to install real-time kernels via NVIDIA SDK Manager, please refer to these links ([link_1](https://zhuanlan.zhihu.com/p/158825325) / [link_2](https://forums.developer.nvidia.com/t/preempt-rt-patches-for-jetson-nano/72941/15) / [link_3](https://orenbell.com/?p=436)).

## 2. Installing Xenomai 3.1
You can refer to these tutorials to configure and compile a Xenoami kernel ([link_1](https://xenomai.org/documentation/xenomai-3/pdf/README.INSTALL.pdf) / [link_2](https://blog.csdn.net/Alanber14919/article/details/60327162) / [link_3](https://blog.csdn.net/pupil_wjj/article/details/105856926)). Or you can directly install the kernel below if your controller is **PC104/PCM3365**.

**Step 1)** Installing *Cobalt* kernel

Copy *Cobalt* kernel that has been configured and compiled:
```
tar -xvf modules.tar
tar -xvf linux-sources.tar
sudo mv 4.19.89-sumantra-xenomai3-qdped-25th-march /lib/modules 
```
Replace `source` and `build` with `linux-4.19.89` from linux-sources.tar:
```
cd /lib/modules/4.19.89-sumantra-xenomai3-qdped-25th-march
sudo rm -r source build
ln -s ~/Downloads/linux-4.19.89 source
ln -s ~/Downloads/linux-4.19.89 build
```
Install *Cobalt* kernel:
```
cd /linux-sources
sudo make install -j3
```

**Step 2)** Installing Xenomai libraries

Make sure the library's version is **3.1**:
```
unzip xenomai.zip && cd xenomai
sudo ./scripts/bootstrap
mkdir build && cd build
sudo ../configure --with-core=cobalt --enable-smp --enable-pshared
sudo make install -j3
```

**Step 3)** Configuring

Create a configuration file:
```
cd /etc/ld.so.conf.d/
sudo touch xenomai.conf 
sudo sh -c "echo '/usr/xenomai/lib' >> xenomai.conf"
sudo ldconfig
```

Update `.bashrc` in normal and root mode to specify the compile path:
```
echo "export PATH=$PATH:/usr/xenomai/bin:/usr/xenomai/lib:/usr/xenomai/include" >> ~/.bashrc

sudo su 
echo "export PATH=$PATH:/usr/xenomai/bin:/usr/xenomai/lib:/usr/xenomai/include" >> ~/.bashrc
exit
```

Create Xenomai groups:
```
sudo addgroup xenomai
sudo addgroup root xenomai
sudo usermod -a -G xenomai `whoami`

XenoGID=`cat /etc/group | sed -nr "s/xenomai:.:([0-9]+):.*/\1/p"`
sudo sh -c "echo $XenoGID > /sys/module/xeno_nucleus/parameters/xenomai_gid"
```

**Step 4)** (Optional) Install **[PCAN Driver](https://www.peak-system.com/fileadmin/media/linux/files/PCAN-Driver-Linux_UserMan_eng.pdf)**

Unzip and open the folder
```
unzip peak-linux-driver-8.11.0.zip && cd peak-linux-driver-8.11.0
```

Build and install
```
make xeno
sudo make install
```

Configure Tx/Rx size and speed (1Mb/s). Note to uncomment the line "options ..."
```
gedit /etc/modprobe.d/pcan.conf 
```
```
options pcan type=pci txqsize=1000 rxqsize=1000 bitrate=1000000
```

Check if PCAN driver is successfully loaded:
```
cat /proc/pcan
```

Sometimes chardev (PCAN) is effected by netdev (SocketCAN). You can remove netdev module `peak_pci` when using `pcan`.
```
sudo modprobe -r peak_pci
sudo modprobe pcan
```

## 3. Startup Menu Configuration
Edit `grub` file. Click this [link](https://blog.csdn.net/xin_yu_xin/article/details/19546613) if you are interested in grub parameters.
```
sudo gedit /etc/default/grub
```

Set the real-time kernel as a default kernel to boot. `1` denotes the second item *Advanced options for Ubuntu*; `0` denotes the first item at submenu which is usually your real-time kernel. Adjust these numbers according to what displays at your startup pages.
```
##### For PREEMPT_RT
GRUB_DEFAULT="1>0"

##### For Xenomai
GRUB_DEFAULT="1>Ubuntu, with Linux 4.19.89-sumantra-xenomai3-qdped-25th-march"
```
Set the startup page to disappear after 10 seconds: 
```
GRUB_TIMEOUT_STYLE=menu（default=hidden)
GRUB_TIMEOUT=10（default=0）
```

Update grub file
```
sudo update-grub
```

Reboot and check if the real-time kernel is loaded
```
reboot
uname -r
```

## 4. Latency Test
* For PREEMPT_RT install test tools first and then run (5 threads, 80 priority, infinite loop):
```
sudo apt-get install rt-tests
sudo cyclictest -t 5 -p 80 -n
```
* For Xenomai run:
```
sudo /usr/xenomai/bin/latency
```

## 5. Tips
**A)** Error `BUG in low_init(): [main] ABI mismatch: required r18, provided r17` in Xenomai latency test.

This is due to different versions of Xenomai's kernel and library ([lnik](https://blog.csdn.net/pupil_wjj/article/details/105856926)). Check and install consistent versions:
```
cat /proc/xenomai/version
sudo /usr/xenomai/sbin/version
```

**B)** Clean up old kernels

Check your current kernel and display all kernels you have:
```
uname -r
sudo dpkg --get-selections |grep linux
```

Delete old kernels, such as `3.0.0-12`:
```
sudo apt-get purge linux-headers-3.0.0-12 linux-image-3.0.0-12-generic
sudo update-grub
```

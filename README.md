<div align="center">

#  Installation Guide to Real-Time Linux
<img width="400" src="/folder.png">
</div>

This repository presents 2 methods to build a real-time environment on your linux OS. The knernel files in the above figure are not offered since they are too large to upload in GitHub. You can request Jia Xinyu (xinyu.jia@u.nus.edu) for these knernel files.

1. `PREEMPT RT` - Ubuntu 16.04/18.04/20.04 LTS
2. `Xenomai` - Ubuntu 16.04/18.04 LTS.

## 1. Installing PREEMPT RT patch
You can refer to these tutorials to configure a kernel and then compile ([link_1](https://www.jianshu.com/p/b74b05d26cf9) / [link_2](https://blog.csdn.net/shenyage/article/details/102099198) / [link_3](https://blog.csdn.net/weixin_43455581/article/details/103899362)). Or you can directly install patched kernels below if your controllers use **Intel/AMD** processors e.g. PC104/PCM3365 or Intel NUC. The real-time kernels only have 2 differences with normal kernels:
* **Preemption Model**: Preemptible Kernel (Low-Latency Desktop) -> **Fully Preemptible Kernel (RT)**
* **`Timer Frequency**: 250 Hz -> **1000 Hz** 

**1)** For Ubuntu 16.04 run:
```
sudo dpkg -i linux-image-4.13.13-ashwin-rt5_1.0_amd64.deb
sudo dpkg -i linux-headers-4.13.13-ashwin-rt5_1.0_amd64.deb
```
**2)** For Ubuntu 18.04 run: (recommended)
```
sudo dpkg -i linux-image-5.4.69-rt39_5.4.69-rt39-1_amd64.deb
sudo dpkg -i linux-headers-5.4.69-rt39_5.4.69-rt39-1_amd64.deb
```
**3)** For Ubuntu 20.04 run:
```
sudo dpkg -i linux-image-5.11.0-jiaxy-2021128-rt7_2021_amd64.deb
sudo dpkg -i linux-headers-5.11.0-jiaxy-2021128-rt7_2021_amd64.deb
```

**OTHERS**: If your controllers use **ARM** processors, e.g. NVIDIA Jeston Nano/TX/Xavier which requires to install via NVIDIA SDK Manager, please refer to these links ([link_1](https://zhuanlan.zhihu.com/p/158825325) / [link_2](https://forums.developer.nvidia.com/t/preempt-rt-patches-for-jetson-nano/72941/15) / [link_3](https://orenbell.com/?p=436)).

## 2. Installing Xenomai kernel



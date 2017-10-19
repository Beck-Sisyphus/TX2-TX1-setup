# 1. flash the TX2
## 1.1 download jetpack L4T 3.0

Provided by TA: JetPack-L4T-3.0-linux.run

put it in a new folder. Run:

`chmod +x JetPack-L4T-3.0-linux.run`

`./JetPack-L4T-3.0-linux.run`


Warning:
* do not run the script in sudo mode when installing host components
* do not install OpenCV4Tegra V2.4 for TX2, we will use OpenCV3
* for TX2 you may need to flash it tiwce, one just flash the OS, the other install cuda
* if the tx1/tx2 cannot boot the GUI, but shut down after finish loading BIOS, try to change a more powerful power source

After flash, the system can be launch.

username: nvidia
password: nvidia

# 2. Install basic tools

connect to lab WiFi: indigo or indigo5G,install arp-scan and
run: `sudo arp-scan --interface=eth0 192.168.1.0/24 | grep NVIDIA'
find your TX2 device.

ssh to your TX2 and run:

`sudo apt-get install git`

`mkdir setup`

`git clone https://github.com/gaowenliang/TX2-TX1-setup.git`

`sudo ./install.sh`

`sudo reboot`

to install USB device and reboot.

run one by one:

`sudo apt-get install terminator -y`


`sudo apt-get install cmake -y`

`sudo apt-get install vim -y`

`sudo apt-get install htop`

# 3. Install ROS
follow http://wiki.ros.org/Installation/Ubuntu

# 4. Eigen manuly
## install Eigen stable release V3.3.3

* http://eigen.tuxfamily.org/index.php?title=Main_Page

* it will fail 6 ctest cases out of 798
* or if you build BTL, fail 13 ctest cases out of 832

host:

`scp ~/Downloads/eigen-eigen-67e894c6cd8f.tar.bz2 ubuntu@<YOUR_SSH PATH>:~/`
 

TX2:
`cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local -DCMAKE_INSTALL_PREFIX=/usr/local -DEIGEN_TEST_CXX11=ON -DEIGEN_CUDA_COMPUTE_ARCH=62 -DEIGEN_BUILD_BTL=ON`

`make check`

this step will take really a long time

# 5. ceres manly
normal install, without eigen, need pass all ctest cases

typical Ceres test bench mark is 131s for TX2

ceres-solver.org/ceres-solver-1.12.0.tar.gz

`sudo apt-get install libgoogle-glog-dev`

`sudo apt-get install libatlas-base-dev -y`

`sudo apt-get install libsuitesparse-dev -y`

`tar zxf ceres-solver-1.12.0.tar.gz`

`mkdir ceres-bin`

`cd ceres-bin`

`cmake ../ceres-solver-1.12.0 -DCMAKE_INSTALL_PREFIX=/usr/local` 

`make -j4`

`make test`

`make install`

# 6. Download buildOpencvTX2, remove libeigen3-dev, install Opencv manuly

https://github.com/jetsonhacks/buildOpenCVTX2/blob/master/buildOpenCV.sh

in opencv V3.2.0
opencv_test_cudev will fail, but acturally, it success, ctest made a wrong test compare between 2.0 and 2.
 
# 5. ros-desktop source install, remove eigen & opencv3

# ROSTX2

you could install ROS through apt, but it might conflict with OPENCV3-CUDA

1 if ROS deb init fall try

`sudo c_rehash /etc/ssl/certs`

2 to enable best performence mode use:

`~$ sudo nvpmodel -m 0`

`~$ sudo ./jecson_clock.sh`

## SDL
`sudo apt-get install libsdl-image1.2-dev 

and

sudo apt-get install libsdl-dev

# some useful scropts
monitor the CPU temperture

`watch -n 1 cat /sys/devices/virtual/thermal/thermal_zone1/temp`

check CUDA version

`nvcc -V`


# Enable /dev/ttyTHS2 (UART1) for TX2

## install dtc 
```
sudo apt-get install device-tree-compiler
```
## rewrite kernel

### enable tty THS2
```
sudo -s
cd /tmp
dtc -I dtb -O dts -o extracted.dts /boot/tegra186-quill-p3310-1000-c03-00-base.dtb
# Search for "serial@c280000" where it is a block of code and not just a single line...
# Change status = "disabled" to status = "okay";
# Build a modified version:
dtc -I dts -O dtb -o /boot/modified_tegra186-quill-p3310-1000-c03-00-base.dtb extracted.dts
cd /boot/extlinux
# edit extlinux.conf...add this line between MENU LABEL line and LINUX line:
FDT /boot/modified_tegra186-quill-p3310-1000-c03-00-base.dtb
```
### enable CAN buss

ref: http://www.jetsonhacks.com/2017/03/25/build-kernel-and-modules-nvidia-jetson-tx2/

```
$ git clone https://github.com/jetsonhacks/buildJetsonTX2Kernel.git
```

### use Manifold2 board USBs

# test Ethernet speeed

```
$ sudo apt-get install iperf
#start server
$ sudo iperf -s
#start client
$ sudo iperf -c 192.168.1.xxx -i 5
```

# install SSH keys
```
ssh-copy-id user@123.45.56.78
```

# bash tools

## devlist
devlist is 
```
aliased to `sudo arp-scan --interface=eth0 192.168.1.0/24 | grep NVIDIA'
```

## dev 
devlist is aliased to `sudo arp-scan --interface=eth0 192.168.1.0/24 | grep NVIDIA'
dev is a function
```
dev () 
{ 
    local ip="$(sudo arp-scan --interface=eth0 192.168.1.0/24 | grep NVIDIA | sed -n $1p | cut -d'	' -f 1)";
    echo "ssh into $ip";
    ssh nvidia@$ip
}
```

# max_performance.sh

```
nvidia@tegra-ubuntu:~$ cat max_performance.sh 
echo "Enabling fan for safety..."
if [ ! -w /sys/kernel/debug/tegra_fan/target_pwm ] ; then
echo "Cannot set fan -- exiting..."
fi
echo 255 > /sys/kernel/debug/tegra_fan/target_pwm

nvpmodel -m 0
./jetson_clocks.sh
```

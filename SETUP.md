
[Install info](https://forum.armbian.com/topic/32493-rockchip-rk3318-x88-pro-10-in-progress/) ([original](https://forum.armbian.com/topic/26978-csc-armbian-for-rk3318rk3328-tv-box-boards/page/26/#comment-134910))

1. Install OpenVFD
```
#install kernel headers (skip if already installed)
### INSTALL KERNEL HEADERS ####
sudo apt install armbian-config
sudo armbian-config
### TAKE THE SYSTEM NAME ####
uname -r
https://imola.armbian.com/apt/pool/main/l/
in my case is 
https://imola.armbian.com/apt/pool/main/l/linux-libc-dev-current-rockchip64/
sudo dpkg -i file_name.deb

# download the sources:
git clone https://github.com/hqnicolas/linux_openvfd

# change the directory
cd linux_openvfd/driver

# edit the Makefile
nano Makefile
# change the line
        KERNELDIR = ../../../
# to this:
        KERNELDIR = /lib/modules/6.6.32-current-rockchip64/build

# create a symlink to correct System.map in this KERNELDIR - in my case:
sudo ln -sf /boot/System.map-6.6.28-current-rockchip64 /lib/modules/6.6.32-current-rockchip64/build/System.map 

sudo ln -s /boot/System.map-6.6.28-current-rockchip64 /usr/src/linux-headers-6.6.32-current-rockchip64/System.map 
# compile the driver (in openvfd/driver):
sudo apt install make
sudo apt-get update
sudo apt-get install --reinstall build-essential
sudo apt install gcc-aarch64-linux-gnu
make -j4

# install module
sudo make modules_install

# load module
modprobe openvfd
```
2. Install OpenVFDService
```
cd linux_openvfd
make -j4 OpenVFDService
sudo cp OpenVFDService /usr/sbin/
```
3. Update dts using armbian-config
```
       openvfd {
                compatible = "open,vfd";
                dev_name = "openvfd";
                status = "okay";
        };
```
4. Create openvfd.service in /lib/systemd/system/openvfd.service
```
[Unit]
Description=OpenVFD Service
ConditionPathExists=/proc/device-tree/openvfd/

[Service]
ExecStart=/bin/sh -c '[ `cat /proc/device-tree/openvfd/compatible` = "open,vfd" ] && /sbin/modprobe openvfd; /usr/sbin/OpenVFDService'
ExecStop=/bin/kill -TERM $MAINPID
ExecStopPost=-/usr/sbin/rmmod openvfd
RemainAfterExit=yes

[Install]
WantedBy=basic.target
```
5. create VFD config in /etc/modprobe.d/openvfd.conf
```
options openvfd vfd_gpio_clk="4,0x13,0"
options openvfd vfd_gpio_dat="4,0x16,0"
options openvfd vfd_gpio_stb="4,0x12,0"
options openvfd vfd_chars="4,0,1,2,3"
options openvfd vfd_dot_bits="0,1,3,2,4,5,6"

#ID     Display type
#0x03      A display like on the A95X R2 (or the Abox A1 Max). <--

#.2     Reserved - must be 0.
#.3     flags
#.4     0 - > FD628 and compatible controllers
options openvfd vfd_display_type="0x03,0x00,0x00,0x00"
```
6. enable service and start (this will autoload the kernel module openvfd
```
sudo systemctl enable openvfd
sudo systemctl start openvfd
```
After this the system clock will successfully show up on the front LCD

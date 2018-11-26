# Nvidia + Linux

1 . Install the Graphics Drivers PPA and install the latest nvidia drivers
```
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
sudo apt install nvidia-driver-410 nvidia-prime [nvidia-settings]
```
PPA: https://launchpad.net/~graphics-drivers/+archive/ubuntu/ppa

2. Turn on Kernel Mode Setting for the nvidia driver
```
echo "options nvidia-drm modeset=1" > /lib/modprobe.d/nvidia-kms.conf
sudo update-initramfs -u
```
Reference: https://forum.manjaro.org/t/nvidia-drm-kernel-mode-setting/53756/3

Reference: https://wiki.archlinux.org/index.php/NVIDIA#DRM_kernel_mode_setting

Reference: http://us.download.nvidia.com/XFree86/Linux-x86/370.28/README/kms.html

Reference: https://devtalk.nvidia.com/default/topic/1022670/linux/official-driver-384-59-with-geforce-1050m-doesn-t-work-on-opensuse-tumbleweed-kde/post/5203910/#5203910

3. Find the PCI Bus Id of the graphics card (Ex: `06.00.00`)
```
lscpi | grep 3D
```

4. Update the Xorg configuration files:
```
# /usr/share/X11/xorg.conf.d/10-nvidia.conf
Section "Module"
    Load "modesetting"
EndSectionj

Section "Device"
    Identifier "nvidia"
    Driver "nvidia"
    BusID "PCI:6.0.0" # not an error, format is #.#.#
    Option "AllowEmptyInitialConfiguration"
EndSection
```
Reference: http://us.download.nvidia.com/XFree86/Linux-x86/358.16/README/randr14.html

Reference: https://wiki.archlinux.org/index.php/NVIDIA_Optimus#Using_nvidia

5. Update `.xinitrc`
```
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
...
exec i3
```
Reference: https://wiki.archlinux.org/index.php/NVIDIA_Optimus#Using_nvidia

6. Verify everything works using `nvidia-smi`. The second table `Processes` should show that `Xorg` is using the GPU. Also verify that `glxinfo | grep vendor` shows NVIDIA as the vendor for the OpenGL library

Reference: https://wiki.archlinux.org/index.php/NVIDIA_Optimus#Checking_3D

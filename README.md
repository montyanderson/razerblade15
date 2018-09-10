# razerblade15

Razer Blade 15 (2018)

## Recompiling Kernel for touchpad support

Download kernel tarball

``` bash
wget https://git.kernel.org/torvalds/t/linux-4.18-rc4.tar.gz
```

Extract archive

``` bash
tar xvfs linux-4.18-rc4.tar.gz
```

Enter directory

``` bash
cd linux-4.18-rc4
```

  Ubuntu-specific patches

  http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.18-rc4/

  ```
  wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.18-rc4/0001-base-packaging.patch;
  patch -f < 0001-base-packaging.patch
  ```

  ```
  wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.18-rc4/0002-UBUNTU-SAUCE-add-vmlinux.strip-to-BOOT_TARGETS1-on-p.patch;
  patch -f < 0002-UBUNTU-SAUCE-add-vmlinux.strip-to-BOOT_TARGETS1-on-p.patch
  ```

  ```
  wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.18-rc4/0003-UBUNTU-SAUCE-tools-hv-lsvmbus-add-manual-page.patch;
  patch -f < 0003-UBUNTU-SAUCE-tools-hv-lsvmbus-add-manual-page.patch
  ```

  ```
  wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.18-rc4/0004-debian-changelog.patch;
  patch -f < 0004-debian-changelog.patch
  ```

  ```
  wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.18-rc4/0005-configs-based-on-Ubuntu-4.18.0-1.2.patch;
  patch -f < 0005-configs-based-on-Ubuntu-4.18.0-1.2.patch
  ```

Apply trackpad patches

```
wget https://raw.githubusercontent.com/jbdrthapa/razerblade15/master/razerfiles/touchpad/gpiolib.patch;
patch drivers/gpio/gpiolib.c < gpiolib.patch
```

```
wget https://raw.githubusercontent.com/jbdrthapa/razerblade15/master/razerfiles/touchpad/pinctrl-intel.patch;
patch drivers/pinctrl/intel/pinctrl-intel.c < pinctrl-intel.patch
```

Install compilation dependencies

```
sudo apt-get install bison git build-essential kernel-package fakeroot libncurses5-dev libssl-dev ccache flex
```

Copy config from current kernel

```
cp /boot/config-`uname -r` .config
```

Clean source root

```
make clean
```

Compile kernel

```
make -j `getconf _NPROCESSORS_ONLN` deb-pkg LOCALVERSION=-custom
```

Go to parent directory

```
cd ../
```

Install newly built kernel

```
sudo dpkg -i *.deb
```

Reboot, hold shift down while Grub is loading, select the new kernel.


## Adding NVIDIA Driver DKMS Support to 4.18.X Kernel
As of July 2018, new Linux kernels had made some changes causing incompatibilities with Nvidia proprietary drivers.  This can be fixed easily by applying a patch to the Nvidia DKMS code to properly expose features in the DRM, which will allow the DKMS to build properly.

Download and install the Nvidia drivers from the graphics team ppa

```
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt-get update
apt-get install nvidia-390 nvidia-dkms-390
```

The DKMS portion may fail to correctly build the DKMS for your 4.18 kernel.  If it does, apply the nvidia-drm-drv.c file.
```
cp razerfiles/nvidia-drm-drv.c /usr/src/nvidia-390.67/nvidia-drm/nvidia-drm-drv.c
sudo dkms build -m nvidia -v 390.67 -k $(uname -r) -a x86_64
sudo dkms install -m nvidia -v 390.67 -k $(uname -r) -a x86_64
```

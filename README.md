# GPU Passthrough with a Radeon 7000 series card and an iGPU
Passing through a single GPU with igpu as host.

## This requires:
CPU virtualization is enabled in bios.\
a CPU with integrated grafics eg. Ryzen 7600x.\
a discrete GPU eg. RX 7900 XT\
GRUB as bootloader\
Arch installation with ZEN kernel. (im sure others work, i know Gentoo do...)\

## My system specs
    ASUS ROG B650E-F Gaming Wifi
    AMD Ryzen 5 7600X - 6 Cores (Host GPU)
    AMD Radeon RX 7900 XT (Guest VM GPU)
    32GB (6000MHz) DDR5 RAM
    Host OS: Cachyos Linux

    My host iGPU (RDNA2, on Ryzen R5 7600x ) is connected to monitor with an displayport cable on my motherboard.
    My guest GPU (Radeon RX 7900 XT) is connected to monitor with an HDMI cable (to fool it into turning on).

### Make sure your system is updated
`sudo pacman -Syu`

# Check IOMMU Grouping
If your using Cachyos or other distro that defaults to fish just simply in terminal enter
```
bash
```
And it will change it from fish to bash (you can afterwords return to fish, simply by enthering "fish".

You can use this bash script to see your IOMMU groupings
```bash
#!/bin/bash

for d in /sys/kernel/iommu_groups/*/devices/*; do
  n=${d#*/iommu_groups/*}; n=${n%%/*}
  printf 'IOMMU Group %s ' "$n"
  lspci -nns "${d##*/}"
done
```
You will get a simelar output like this
```
IOMMU Group 0 00:00.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge Root Complex [1022:14d8]
IOMMU Group 10 00:08.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge Internal GPP Bridge to Bus [C:A] [1022:14dd]
IOMMU Group 11 00:08.3 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge Internal GPP Bridge to Bus [C:A] [1022:14dd]
IOMMU Group 12 00:14.0 SMBus [0c05]: Advanced Micro Devices, Inc. [AMD] FCH SMBus Controller [1022:790b] (rev 71)
IOMMU Group 12 00:14.3 ISA bridge [0601]: Advanced Micro Devices, Inc. [AMD] FCH LPC Bridge [1022:790e] (rev 51)
IOMMU Group 13 00:18.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge Data Fabric; Function 0 [1022:14e0]
IOMMU Group 13 00:18.1 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge Data Fabric; Function 1 [1022:14e1]
IOMMU Group 13 00:18.2 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge Data Fabric; Function 2 [1022:14e2]
IOMMU Group 13 00:18.3 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge Data Fabric; Function 3 [1022:14e3]
IOMMU Group 13 00:18.4 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge Data Fabric; Function 4 [1022:14e4]
IOMMU Group 13 00:18.5 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge Data Fabric; Function 5 [1022:14e5]
IOMMU Group 13 00:18.6 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge Data Fabric; Function 6 [1022:14e6]
IOMMU Group 13 00:18.7 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge Data Fabric; Function 7 [1022:14e7]
IOMMU Group 14 01:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 10 XL Upstream Port of PCI Express Switch [1002:1478] (rev 10)
IOMMU Group 15 02:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 10 XL Downstream Port of PCI Express Switch [1002:1479] (rev 10)
IOMMU Group 16 03:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 31 [Radeon RX 7900 XT/7900 XTX/7900 GRE/7900M] [1002:744c] (rev cc)
IOMMU Group 17 03:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 31 HDMI/DP Audio [1002:ab30]
IOMMU Group 18 04:00.0 Non-Volatile memory controller [0108]: Kingston Technology Company, Inc. KC3000/FURY Renegade NVMe SSD [E18] [2646:5013] (rev 01)
IOMMU Group 19 05:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 600 Series Chipset PCIe Switch Upstream Port [1022:43f4] (rev 01)
IOMMU Group 1 00:01.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge Dummy Host Bridge [1022:14da]
IOMMU Group 20 06:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 600 Series Chipset PCIe Switch Downstream Port [1022:43f5] (rev 01)
IOMMU Group 20 07:00.0 Non-Volatile memory controller [0108]: Kingston Technology Company, Inc. KC3000/FURY Renegade NVMe SSD [E18] [2646:5013] (rev 01)
IOMMU Group 21 06:08.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 600 Series Chipset PCIe Switch Downstream Port [1022:43f5] (rev 01)
IOMMU Group 22 06:09.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 600 Series Chipset PCIe Switch Downstream Port [1022:43f5] (rev 01)
IOMMU Group 23 06:0a.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 600 Series Chipset PCIe Switch Downstream Port [1022:43f5] (rev 01)
IOMMU Group 23 0a:00.0 Ethernet controller [0200]: Intel Corporation Ethernet Controller I225-V [8086:15f3] (rev 03)
IOMMU Group 24 06:0b.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 600 Series Chipset PCIe Switch Downstream Port [1022:43f5] (rev 01)
IOMMU Group 24 0b:00.0 Network controller [0280]: MEDIATEK Corp. MT7921K (RZ608) Wi-Fi 6E 80MHz [14c3:0608]
IOMMU Group 25 06:0c.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 600 Series Chipset PCIe Switch Downstream Port [1022:43f5] (rev 01)
IOMMU Group 25 0c:00.0 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] 600 Series Chipset USB 3.2 Controller [1022:43f7] (rev 01)
IOMMU Group 26 06:0d.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 600 Series Chipset PCIe Switch Downstream Port [1022:43f5] (rev 01)
IOMMU Group 26 0d:00.0 SATA controller [0106]: Advanced Micro Devices, Inc. [AMD] 600 Series Chipset SATA Controller [1022:43f6] (rev 01)
IOMMU Group 27 0e:00.0 Non-Volatile memory controller [0108]: Kingston Technology Company, Inc. NV2 NVMe SSD [SM2267XT] (DRAM-less) [2646:5017] (rev 03)
IOMMU Group 28 0f:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Raphael [1002:164e] (rev c7)
IOMMU Group 29 0f:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Radeon High Definition Audio Controller [Rembrandt/Strix] [1002:1640]
IOMMU Group 2 00:01.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge GPP Bridge [1022:14db]
IOMMU Group 30 0f:00.2 Encryption controller [1080]: Advanced Micro Devices, Inc. [AMD] Family 19h PSP/CCP [1022:1649]
IOMMU Group 31 0f:00.3 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge USB 3.1 xHCI [1022:15b6]
IOMMU Group 32 0f:00.4 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge USB 3.1 xHCI [1022:15b7]
IOMMU Group 33 10:00.0 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge USB 2.0 xHCI [1022:15b8]
IOMMU Group 3 00:01.2 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge GPP Bridge [1022:14db]
IOMMU Group 4 00:02.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge Dummy Host Bridge [1022:14da]
IOMMU Group 5 00:02.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge GPP Bridge [1022:14db]
IOMMU Group 6 00:02.2 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge GPP Bridge [1022:14db]
IOMMU Group 7 00:03.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge Dummy Host Bridge [1022:14da]
IOMMU Group 8 00:04.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge Dummy Host Bridge [1022:14da]
IOMMU Group 9 00:08.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Raphael/Granite Ridge Dummy Host Bridge [1022:14da]
```
We are looking for our grafics card groups and IDs
```
IOMMU Group 16 03:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 31 [Radeon RX 7900 XT/7900 XTX/7900 GRE/7900M] [1002:744c] (rev cc)
IOMMU Group 17 03:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 31 HDMI/DP Audio [1002:ab30]
```
The group would be 16 and 17 (Notice that they are in different groups?\
If yours belong to the same group, you need to patch your kernel and **that is beyond the scoop of this guide**.)

And the ids in my case would be
```
1002:744c and 1002:ab30
```
We shall note this for use in our grub conf.

### Grub configuration
```
sudo nano /etc/default/grub
```
Add the following to the **GRUB_CMDLINE_LINUX_DEFAULT** line (keep the rest as is).
```
GRUB_CMDLINE_LINUX_DEFAULT='vfio_pci,ids=1002:744c,1002:ab30
```
**Remember to change the ids to that of your own hardware that you want to passthrough**

Commit the changes you made
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
**reboot** you system before continueing!

Install yay (if not already) so we can be bleeding edge.
```
sudo pacman -S yay
```

### QEMU install with yay
```
yay -S qemu-full libvirt virt-manager bridge-utils dnsmasq iptables-nft virt-viewer edk2-ovmf swtpm qemu-img guestfs-tools libosinfo tuned
```
### Install looking-glass with yay
```
yay -S looking-glass
```

### Edit qemu.conf
```
sudo nano /etc/libvirt/qemu.conf
```
change the user and group from "root" to your username in qemu.conf (user ctrl+w to search in nano)
```
user = "username"
```
```
group = "username"
```

### Enable and Start the libvirtd service
```
sudo systemctl enable libvirtd.service
```
```
sudo systemctl start libvirtd.service
```

### Enable tuned and setting it op for virtual-host.
```
sudo systemctl enable --now tuned
```

Show what profile is currentl active for tuned.
```
tuned-adm active
```

Set et profile to virtual-host
```
sudo tuned-adm profile virtual-host
```

Setup the network for qemu
```
sudo virsh net-start default
```
```
sudo virsh net-autostart default
```

Setup the user to use QEMU instead of root.

```
sudo usermod -a -G qemu,input,kvm,libvirt $(whoami)
```

# Setting up looking-glass
Edit looking-glass.conf
```
sudo nano /etc/tmpfiles.d/10-looking-glass.conf
```
And add the follwing to it
```
# Type Path               Mode UID  GID Age Argument

f /dev/shm/looking-glass 0660 yourusername kvm -
```
```
systemd-tmpfiles --create
```
Or if you already created the shm looking-glass file, you prolly need to change ownership.
```
sudo chown username:kvm /dev/shm/looking-glass
```
### for easiere windows 11 installation, might need to add iptables to qemu network conf
```
sudo nano /etc/libvirt/network.conf
```
At the of end of file add this
```
firewall_backend = "iptables"
```

Then restart the libvirtd service (or reboot system)
```
sudo systemctl restart libvirtd
```

# Now open Virtual Manager
## i am asuming that you already got a windows 11 iso file.

Create a new VM

![billede](https://github.com/user-attachments/assets/fe2144ff-2216-4ee4-b4a6-84bb44745bca)

Install from local media file

![billede](https://github.com/user-attachments/assets/f458bd76-a8c2-407f-9744-2c289926b25e)

Browse and find your windows 11 iso file

![billede](https://github.com/user-attachments/assets/b3a45dcc-0561-40b9-93be-71727ab7104b)

Give it minimum of 2gb of memory and 2 cores, but for anything usefull i recommend about half your system resources

![billede](https://github.com/user-attachments/assets/fab7ffbe-c7a1-4a52-8fc9-c221d4cb6dc0)

You need atleast 128gb of storeage for windows 11, but i do recommend 250gb+\
I also reccomend that you use "select or create custom storeage" to manage where your qcow file goes.\
but dont exceed your avalible space!

![billede](https://github.com/user-attachments/assets/8873fac8-df45-490d-92a0-c07eb11fb107)

![billede](https://github.com/user-attachments/assets/cb5ae445-7320-4b0f-8a72-b9baea2a0c66)

Give the VM a name, here i called it **win11-guide**\
Next make sure an tick of "Customize configuration before install"

![billede](https://github.com/user-attachments/assets/0ecac695-7b16-4ba8-88a8-a75e87592334)

Select "CPUs" and set the CPU topology so that you use about half your cores avalible.\
Also make sure you have TPM set "Emulated" is fine.

![billede](https://github.com/user-attachments/assets/56c0641a-1e3a-4d0e-9bec-139d43f0c758)

# Begin installation
Now simply press the "Begin installation" button and install windows 11 like you normally would\
A note is, when your get to account setup, select Coporate/School and setup for domain\
that way you get to install with a local account, instead of having to create an online MS account.

# When windows 11 installation is done






# GPU Passthrough with a Radeon 7000 series card and an iGPU
Passing through a single GPU with igpu as host.

## This requires:
CPU virtualization is enabled in bios.
a CPU with integrated grafics eg. Ryzen 7600x.
a discrete GPU eg. RX 7900 XT
Arch installation with ZEN kernel. (im sure others work, i know Gentoo do...)

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
`sudo pacman -S yay`

### QEMU install with yay
`yay -S qemu-full libvirt virt-manager bridge-utils dnsmasq iptables-nft virt-viewer edk2-ovmf swtpm qemu-img guestfs-tools libosinfo tuned`

### Enable and Start the libvirtd service
`sudo systemctl enable libvirtd.service`

`sudo systemctl start libvirtd.service`
### Enable tuned and setting it op for virtual-host.
`sudo systemctl enable --now tuned`
Show what profile is currentl active for tuned.
`tuned-adm active`
Set et profile to virtual-host
`sudo tuned-adm profile virtual-host`
Setup the network for qemu
`sudo virsh net-start default`
`sudo virsh net-autostart default`

Setup the user to use QEMU instead of root.
`sudo usermod -a -G libvirt $(whoami)`
`sudo usermod -a -G kvm$(whoami)`
`sudo usermod -a -G qemu $(whoami)`







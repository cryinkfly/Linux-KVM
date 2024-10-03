STILL IN PROGRESS!!!

# Linux-KVM
This project involves providing detailed information on the installation and configuration of KVM (Kernel-based Virtual Machine) across various Linux distributions. KVM is a popular open-source virtualization solution integrated into the Linux kernel, enabling users to run multiple virtual machines on a single host. The project covers key steps for setting up KVM on different Linux distros, including dependencies, installation of necessary packages, enabling virtualization support, configuring network bridges, and managing virtual machines through tools like libvirt, virt-manager, or command-line utilities. Each distro's specific nuances and commands are highlighted for a seamless setup experience.

---

## Show PCI identification number and [Vendor-ID:Device-ID] of the graphics card and USB controller:

    lspci -nn | grep -i amd #All AMD graphics cards are displayed!

    lspci -nn | grep -i nvidia #All NVIDIA graphics cards are displayed!

    lspci -nn | grep -i usb #All USB devices (controllers) are displayed!

For example my GPU and PCI-USB controller:

    12:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 24 [Radeon PRO W6400] [1002:7422]
    12:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 21/23 HDMI/DP Audio Controller [1002:ab28]
    06:00.0 USB controller [0c03]: ASMedia Technology Inc. ASM2142/ASM3142 USB 3.1 Host Controller [1b21:2142]

---

### Debian

    su -
    apt install bridge-utils guestfs-tools libguestfs-tools libvirt-clients libvirt-daemon ovmf qemu-system-x86 qemu-utils virt-manager virt-viewer virtinst

---

### Ubuntu

    sudo apt update
    sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager

---

### Fedora

#### Fedora Workstation

    sudo dnf install virt-install libvirt-daemon-config-network libvirt-daemon-kvm qemu-kvm virt-manager virt-viewer guestfs-tools libguestfs-tools virt-top libvirt-devel bridge-utils edk2-ovmf
    
    sudo nano /etc/default/grub

    # Enable the IOMMU feature and the [vfio-pci] kernel module on the KVM host (line 6).
    GRUB_CMDLINE_LINUX="rhgb quiet amd_iommu=on iommu=pt video=efifb:off" #For Intel CPU: intel_iommu=on
    
    sudo grub2-mkconfig -o /etc/grub2-efi.cfg

#### Fedora Atomics (Silverblue, ...)

    rpm-ostree install virt-install libvirt-daemon-config-network libvirt-daemon-kvm qemu-kvm virt-manager virt-viewer guestfs-tools libguestfs-tools virt-top bridge-utils edk2-ovmf
    
    sudo rpm-ostree kargs --append-if-missing="amd_iommu=on" --append-if-missing="iommu=pt" --append-if-missing="video=efifb:off" --append-if-missing="rd.driver.pre=vfio_pci" --reboot #For Intel CPU: intel_iommu=on

---

### openSUSE

#### openSUSE Leap & Tumbleewed

    sudo zypper install -t pattern kvm_server kvm_tools
    sudo zypper install libvirt libvirt-client libvirt-daemon virt-manager virt-install virt-viewer qemu qemu-kvm qemu-ovmf-x86_64 qemu-tools

#### openSUSE Micro OS - Aeon, Kalpa & Baldur

    sudo transactional-update pkg install -t pattern kvm_server kvm_tools
    sudo transactional-update -c pkg install libvirt libvirt-client libvirt-daemon virt-manager virt-install virt-viewer qemu qemu-kvm qemu-ovmf-x86_64 qemu-tools

---

## 2. Configuration of /etc/default/grub

### 2.1 Open the GRUB configuration file for editing:

The GRUB configuration file defines how your kernel is loaded, and it's sometimes necessary to modify it for enabling virtualization features or specific kernel modules.

    sudo nano /etc/default/grub

    # OR

    su -
    nano /etc/default/grub

    # OR
    
    su -c 'nano /etc/default/grub' # openSUSE Micro OS ...

### 2.2 Modify the GRUB_CMDLINE_LINUX_DEFAULT line to include any virtualization options you may need. For example, to enable IOMMU for better hardware virtualization support:

Enable the IOMMU feature and the [vfio-pci] kernel module on the KVM host (line 6).

    for AMD CPU, set [amd_iommu=on iommu=pt video=efifb:off]
    for INTEL CPU, set [intel_iommu=on iommu=pt video=efifb:off]

*Note 1: The "video=efifb:off" option should only be added if your system is configured to automatically load the graphical environment! If you want to switch to the graphical environment via the terminal after booting, you may no longer see the terminal.*

*Note 2: In addition, the option causes problems with some NVIDIA graphics cards!*

*Note 3: Basically, the "amd_iommu=on" or "intel_iommu=on" option would also suffice, but you get better performance in the guest VM with the "iommu=pt" option and with the "video=efifb:off" option will prevent the driver from stealing the GPU.*

![237171775-a91e4c93-92e3-4397-88df-6e68d10eee01](https://github.com/user-attachments/assets/1d1400af-709f-4f11-98e0-a85aacd19e3f)

### 2.3 Update GRUB to apply the changes:

    sudo update-grub  # For Debian/Ubuntu
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # For CentOS/RHEL/Fedora/openSUSE Leap & TW

    # openSUSE Micro OS ...
    su -c 'nano /etc/default/grub'
    

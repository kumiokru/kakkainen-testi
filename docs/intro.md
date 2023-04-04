# Testing Terraform and Ansible 

## Preface
This is more or less a personal journal to document the test of Terraform and Ansible to do different things with them. Errors and such can be pointed out to author if you happen to know how to get in contact but for the moment contact details will be left out due to having a piss poor repo name for this and while being heavily modified. 😂

To test Terraform you need a some ways to interact with either a public cloud provider (think for example AWS, GCP or Azure) or if you're being a cheapskate like I am, you could virtualize private Proxmox "cloud". 

Terraform has support for Proxmox and the former can be interacted with Terraform to spin up new virtual machines for example. This is all done using nested virtualization (TODO: insert yo dawg meme here) and then manage said virtual machines with Ansible.

You could also do this on your local computer (e.g. virtualize proxmox or similar directly on your workstation using virtual box or something like it)... 

## Virtualization prerequisites
You have to have virtualization enabled. Easiest way to check this on is to check CPU flags with following smallish bash script that checks if you have virtualization enabled. Works at least with AMD cpus and requires bash version >4 that has support for associative arrays:

``` 
# this should return something if CPU has support for virtualization
LC_ALL=C lscpu | grep Virtualization

# this should return something if kernel has support (all arch linux kernels should have support)
zgrep CONFIG_KVM /proc/config.gz

# this should output info about kernel modules
lsmod | grep kvm
``` 

## Installing Proxmox VE as virtualized server
Most of this info was gathered from the following links:
* [libvirt](https://wiki.archlinux.org/title/libvirt)
* [qemu](https://wiki.archlinux.org/title/QEMU)
* [kvm](https://wiki.archlinux.org/title/KVM)

As I'm running headless Arch linux on my server the first things to install are of course `libvirt`, `qemu-base` and `qemu-virtiofsd` with also `virt-install` to ease the installation process of a VM and `openbsd-netcat` for virt-viewer to work through network.

so therefore:
``` 
pacman -S libvirt qemu-base qemu-virtiofsd virt-install openbsd-netcat
``` 

### Download the image
Download the image of proxmox along with the SHA256SUMS and signature
```
curl -O http://download.proxmox.com/iso/proxmox-ve_7.4-1.iso
curl -O http://download.proxmox.com/iso/SHA256SUMS.asc
curl -O http://download.proxmox.com/iso/SHA256SUMS
```

### something something
``` 
sudo systemctl start libvirtd.service
```

Testing that you get connection to qemu
```
virsh -c qemu:///system

virsh -c qemu:///session
```
Which seems to be the case. 

### something something more...
At this point I seem to have old packages here and there so, update everything
```
pacman -Syu
``` 
Make network for the virtual machine by adding bridge interface to the host machine. Leverage the fact that QEMU default settings allow any user to attach domain (virtual machine) to virbr0 network by default at least in Arch. Your mileage may wary and check the `/etc/qemu/bridge.conf` file for settings or create a new rule.

However, create a new interface:
```
sudo ip link add virbr0 type bridge
sudo ip link set dev virbr0 up

# sanity check, seems ok
bridge link

# set ip address for virbr0 interface
sudo ip address add dev virbr0 192.168.100.1/24
``` 


Install the virtual machine (TODO: make it non interactive if possible, looks like not possible)

```
virt-install  \
  --name proxmox_test \
  --memory 4096             \
  --vcpus=2                 \
  --cpu host-passthrough,cache.mode=passthrough \
  --cdrom $HOME/vms/proxmox-ve_7.4-1.iso \
  --disk size=20,format=qcow2  \
  --network bridge=virbr0   \
  --virt-type kvm           \
  --osinfo detect=on
```

This leaves the domain (virtual machine) running ready to accept connections

Due to my firewall settings, only traffic from internal network is accepted so connect using virt-viewer from another machine like so using ssh as the underlying:
```
virt-viewer --connect qemu+ssh://<user_name_here>@192.168.0.1/session proxmox_test

``` 

Which fires up a graphical user interface to install the guest hypervisor (proxmox) from the ISO. Do it. Do it NAO!


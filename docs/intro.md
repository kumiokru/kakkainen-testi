# Testing Terraform and Ansible 

## Preface
This is more or less a personal journal to document the test of Terraform and Ansible to do different things with them. Errors and such can be pointed out to author if you happen to know how to get in contact but for the moment contact details will be left out due to having a piss poor repo name for this. 😂

To test Terraform you need a some ways to interact with either a public cloud provider (think for example AWS, GCP or Azure) or if you're being a cheapskate like I am (yet still somehow own an internet facing linux server), you could virtualize for example a private proxmox cloud and then magically handle that proxmox with Terraform as it seems to have support for this and then to spin up virtual machines (using nested virtualization (TODO: insert yo dawg meme here)) and then manage said virtual machines with Ansible.

You could also do this on your local computer (e.g. virtualize proxmox or similar directly on your workstation using virtual box or something like it)... 

## Virtualization prerequisites
You have to have virtualization enabled. Easiest way to check this on is to check CPU flags with following smallish bash script that checks if you have virtualization enabled. Works at least with AMD cpus and requires bash version >4 that has support for associative arrays:

TODO: please check this shit, it's not finished and I ran out of interest in this as it's only prerequisite and is only small portion of this whole ordeal.
``` 
_cpu() {

}

_cpu_vendor=$(grep "vendor_id" /proc/cpuinfo | uniq | awk '{print $NF}')



declare -a cpuArray
if [[ _cpu_vendor ~ "AMD" ]]; then
    _cpu="amd"
    _cpu_flags="svm"
    cpuArray=
elif
    _cpu="intel"
    _cpu_flags="vmx"
    
    grep svm /proc/cpuinfo
    cat /sys/module/kvm_amd/parameters/nested
fi 
``` 

## Installing Proxmox VE as virtualized server
As I'm running Arch linux on my server the first things to install are of course 

### Enable 

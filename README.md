# vgpu_unlock_pve
Simple repo to unlock vGPU feature on consumer grade GPU on Proxmox VE machines.

Tested on :
* Proxmox 7.3-3
* 64GB RAM 4x16GB HyperX Fury Beast DDR4 3200MHz
* AMD Ryzen 9 3900X
* Asus Prime X370-Pro
* Kernel 5.15.64-1-pve
* NVIDIA Quadro P400 2GB
* NVIDIA GTX 1080 8GB
* NVIDIA GTX 1070 8GB
* NVIDIA vGPU driver 510.85.03


## Patch Files
This files are stored in this repo just for backup purposes. Prefer use the original files from the [polloloco repo](https://gitlab.com/polloloco/vgpu-proxmox/-/tree/master)

|Patch File|vGPU version|Driver Version|
|:----------:|:-------------:|:------:|
|525.60.12.patch|15.0|NVIDIA-GRID-Linux-KVM-525.60.12-525.60.13-527.41.zip|
|510.108.03.patch|14.3|NVIDIA-GRID-Linux-KVM-510.108.03-513.91.zip|
|510.85.03.patch|14.2|NVIDIA-GRID-Linux-KVM-510.85.03-510.85.02-513.46.zip|
|510.73.06_unlock.patch|14.1|NVIDIA-GRID-Linux-KVM-510.73.06-510.73.08-512.78.zip|

## Compatibility


<details>
<summary>GPU compatibillity matrix:</summary>

|GPU|tested|Driver tested|
|:----------:|:-------------:|:------:|
|Nvidia Quadro P400|working|14.2|
|Nvidia GTX 1060 3gb|working|14.3|
|Nvidia GTX 1070 8gb|working|14.2|
|Nvidia GTX 1080 8gb|working|14.2|
|Nvidia GTX 1050|should work|Untested|
|Nvidia GTX 1050 ti|should work|Untested|
|Nvidia GTX 1060 6GB|should work|Untested|
|Nvidia GTX 1080 ti 11GB|should work|Untested|
|Nvidia GTX 2060 6GB|should work|Untested|
|Nvidia GTX 2060 12GB|special GPU more research needed|Untested|
|Nvidia GTX 2070 8GB|should work|Untested|
|Nvidia GTX 2080 8GB|should work|Untested|
|Nvidia GTX 2080 ti 11GB|should work|Untested|
|Nvidia GTX 1660 6GB|should work|Untested|
|Nvidia GTX 1660 ti 6GB|should work|Untested|
|Nvidia GTX 1650 4GB|should work|Untested|
|Nvidia GTX 1650 6GB|should work|Untested|
|Nvidia GTX 1660 super 6GB|should work|Untested|
|Nvidia GTX 3050 8GB|should work|Untested|
|Nvidia GTX 3060 12GB|should work|Untested|
|Nvidia GTX 3070 8GB|should work|Untested|
|Nvidia GTX 3080 10GB|should work|Untested|
|Nvidia GTX 3090 24GB|should work|Untested|
|Nvidia GTX 3060 ti 8GB|should work|Untested|
|Nvidia GTX 3070 ti 8GB|should work|Untested|
|Nvidia GTX 3080 ti 12GB|should work|Untested|
|Nvidia GTX 3090 ti 24GB|should work|Untested|
</details>


## Upgrade ?

### Uninstall previous driver
If you have already installed the script, you need to uninstall it before upgrading to the new version.
```bash
nvidia-uninstall
```
Then, you can install the new version of the script.
### Get the last version of vgpu-unlock-rs

I asume your vgpu-unlock-rs folder is in /opt
```bash
cd /opt/vgpu_unlock_rs && git pull
```

# Install
I asume you are root or sudoer and you have a working internet connection.
## Prerequisites
You will need an updated installation of Proxmox VE
```bash	
apt update && apt upgrade
```
For post-installation, I recommand the eXtremeSHOK.com script [xshok-proxmox](https://github.com/extremeshok/xshok-proxmox)

It will enable all the features of we will need later.
reboot after the installation of the script 

### Kernel compiling tools
```bash
apt install -y git build-essential dkms pve-headers mdevctl
```	
<details>
    <summary>Details about packages</summary>
    
    - git: to clone the repo
    - build-essential: to compile the driver
    - dkms: to install the driver
    - pve-headers: to compile the driver
    - mdevctl: to create the vGPU
</details>

### Rust compiler
We need the rust compiler to compile the **vgpu_unlock-rs** software.
```bash	
curl https://sh.rustup.rs -sSf | sh -s -- -y
source $HOME/.cargo/env
```

## Cloning repository

In your home directory, clone the repo
```bash
git clone https://gitlab.com/polloloco/vgpu-proxmox.git
```
This is the repo containing the patch file

In the /opt directory, clone the repo
```bash
git clone https://github.com/mbilker/vgpu_unlock-rs.gi
```

## Compile the driver
```bash
cd vgpu_unlock-rs/
cargo build --release
```

## Create the vGPU folder and files
```bash
mkdir /etc/vgpu_unlock
touch /etc/vgpu_unlock/profile_override.toml
mkdir /etc/systemd/system/{nvidia-vgpud.service.d,nvidia-vgpu-mgr.service.d}
echo -e "[Service]\nEnvironment=LD_PRELOAD=/opt/vgpu_unlock-rs/target/release/libvgpu_unlock_rs.so" > /etc/systemd/system/nvidia-vgpud.service.d/vgpu_unlock.conf
echo -e "[Service]\nEnvironment=LD_PRELOAD=/opt/vgpu_unlock-rs/target/release/libvgpu_unlock_rs.so" > /etc/systemd/system/nvidia-vgpu-mgr.service.d/vgpu_unlock.conf
```	

## You need now to download the vGPU driver from [Nvidia portal](https://www.nvidia.com/en-us/drivers/vgpu-software-driver/)
You need to create an account on Nvidia portal to download the driver.

From the Nvidia portal, download the latest supported driver for linux KVM

Copy the run file from the host folder to your proxmox with scp or filezilla

```bash	
scp NVIDIA-Linux-x86_64-525.60.12-vgpu-kvm.run root@pve:/root/
```	

## Patchig driver
```bash
chmod +x NVIDIA-Linux-x86_64-525.60.12-vgpu-kvm.run
./NVIDIA-Linux-x86_64-525.60.12-vgpu-kvm.run --apply-patch ~/vgpu-proxmox/525.60.12.patch
```
The output should be something like this:
```bash
Self-extractible archive "NVIDIA-Linux-x86_64-525.60.12-vgpu-kvm-custom.run" successfully created.
```	

## Install the driver
```bash
./NVIDIA-Linux-x86_64-525.60.12-vgpu-kvm-custom.run --dkms
```
After the installation reboot the server

# Post-installation
## Check GPU status
```bash
nvidia-smi
```	
You should see something like this:
```bash
Sun Dec  4 14:01:07 2022       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.85.03    Driver Version: 510.85.03    CUDA Version: N/A      |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Quadro P400         On   | 00000000:0C:00.0 Off |                  N/A |
| 34%   41C    P8    N/A /  N/A |   1025MiB /  2048MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```	

## Check instances available
```bash
mdevctl types
```
<details>
    <summary>The output should be something like this:</summary>

```bash
0000:0c:00.0
  nvidia-156
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-2B
    Description: num_heads=4, frl_config=45, framebuffer=2048M, max_resolution=5120x2880, max_instance=12
  nvidia-215
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-2B4
    Description: num_heads=4, frl_config=45, framebuffer=2048M, max_resolution=5120x2880, max_instance=12
  nvidia-241
    Available instances: 23
    Device API: vfio-pci
    Name: GRID P40-1B4
    Description: num_heads=4, frl_config=45, framebuffer=1024M, max_resolution=5120x2880, max_instance=24
  nvidia-283
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-4C
    Description: num_heads=1, frl_config=60, framebuffer=4096M, max_resolution=4096x2160, max_instance=6
  nvidia-284
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-6C
    Description: num_heads=1, frl_config=60, framebuffer=6144M, max_resolution=4096x2160, max_instance=4
  nvidia-285
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-8C
    Description: num_heads=1, frl_config=60, framebuffer=8192M, max_resolution=4096x2160, max_instance=3
  nvidia-286
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-12C
    Description: num_heads=1, frl_config=60, framebuffer=12288M, max_resolution=4096x2160, max_instance=2
  nvidia-287
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-24C
    Description: num_heads=1, frl_config=60, framebuffer=24576M, max_resolution=4096x2160, max_instance=1
  nvidia-46
    Available instances: 23
    Device API: vfio-pci
    Name: GRID P40-1Q
    Description: num_heads=4, frl_config=60, framebuffer=1024M, max_resolution=5120x2880, max_instance=24
  nvidia-47
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-2Q
    Description: num_heads=4, frl_config=60, framebuffer=2048M, max_resolution=7680x4320, max_instance=12
  nvidia-48
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-3Q
    Description: num_heads=4, frl_config=60, framebuffer=3072M, max_resolution=7680x4320, max_instance=8
  nvidia-49
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-4Q
    Description: num_heads=4, frl_config=60, framebuffer=4096M, max_resolution=7680x4320, max_instance=6
  nvidia-50
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-6Q
    Description: num_heads=4, frl_config=60, framebuffer=6144M, max_resolution=7680x4320, max_instance=4
  nvidia-51
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-8Q
    Description: num_heads=4, frl_config=60, framebuffer=8192M, max_resolution=7680x4320, max_instance=3
  nvidia-52
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-12Q
    Description: num_heads=4, frl_config=60, framebuffer=12288M, max_resolution=7680x4320, max_instance=2
  nvidia-53
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-24Q
    Description: num_heads=4, frl_config=60, framebuffer=24576M, max_resolution=7680x4320, max_instance=1
  nvidia-54
    Available instances: 23
    Device API: vfio-pci
    Name: GRID P40-1A
    Description: num_heads=1, frl_config=60, framebuffer=1024M, max_resolution=1280x1024, max_instance=24
  nvidia-55
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-2A
    Description: num_heads=1, frl_config=60, framebuffer=2048M, max_resolution=1280x1024, max_instance=12
  nvidia-56
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-3A
    Description: num_heads=1, frl_config=60, framebuffer=3072M, max_resolution=1280x1024, max_instance=8
  nvidia-57
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-4A
    Description: num_heads=1, frl_config=60, framebuffer=4096M, max_resolution=1280x1024, max_instance=6
  nvidia-58
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-6A
    Description: num_heads=1, frl_config=60, framebuffer=6144M, max_resolution=1280x1024, max_instance=4
  nvidia-59
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-8A
    Description: num_heads=1, frl_config=60, framebuffer=8192M, max_resolution=1280x1024, max_instance=3
  nvidia-60
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-12A
    Description: num_heads=1, frl_config=60, framebuffer=12288M, max_resolution=1280x1024, max_instance=2
  nvidia-61
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P40-24A
    Description: num_heads=1, frl_config=60, framebuffer=24576M, max_resolution=1280x1024, max_instance=1
  nvidia-62
    Available instances: 23
    Device API: vfio-pci
    Name: GRID P40-1B
    Description: num_heads=4, frl_config=45, framebuffer=1024M, max_resolution=5120x2880, max_instance=24
```	
</details>

## Check if the vGPU feature is enabled
```bash	
nvidia-smi vgpu
```	

The output should be something like this:    
```bash
Sun Dec  4 14:05:04 2022       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.85.03              Driver Version: 510.85.03                 |
|---------------------------------+------------------------------+------------+
| GPU  Name                       | Bus-Id                       | GPU-Util   |
|      vGPU ID     Name           | VM ID     VM Name            | vGPU-Util  |
|=================================+==============================+============|
|   0  Quadro P400                | 00000000:0C:00.0             |   0%       |
+---------------------------------+------------------------------+------------+
```

if you get an error like this:    
```bash
No supported devices in vGPU mode
```	
Clean up your installation and try again 

## Attach the vGPU to a VM
To add a vGPU to a VM, you need to add an argument to the VM configuration file.    
You can do like this
```bash
echo "args: -uuid 00000000-0000-0000-0000-00000000<VM_ID>" >> /etc/pve/qemu-server/<VM_ID>.conf
```
In a real situation with a vm with id 100, the command would be like this:    
```bash
echo "args: -uuid 00000000-0000-0000-0000-00000000100" >> /etc/pve/qemu-server/100.conf
```

## Adding a vGPU to a VM from WebUI

Go to the Hardware tab and click on the Add button.
Select PCI Device and click on the Add button.
- Select your GPU in Device 
- Select the profile you want to use in MDev Type
- Uncheck All functions 
- Don't forget to Uncheck primary GPU before you had installed the driver
- Check PCI-Express box and you can click on the OK button

## Sources
- https://pve.proxmox.com/wiki/PCI_Passthrough
- https://pve.proxmox.com/wiki/PCI_Passthrough#vGPU
- https://gitlab.com/polloloco/vgpu-proxmox
- https://www.reddit.com/r/homelab/comments/9x0q0m/proxmox_vgpu_passthrough/
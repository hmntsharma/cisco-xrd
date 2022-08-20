# Infrastructure for Cisco XRd with XRD-Tools


Exploring Cisco XRd 7.7.1 (Control-Plane) with the help of [ios-xr/xrd-tools](https://github.com/ios-xr/xrd-tools) in Ubuntu 22.04.1 running in EVE-NG

* The vRouter image is out of the scope of this document.

## Setup

+ Ubuntu 22.04.1 [16 vcpu/32GB RAM]
+ Download the Cisco XRd image from the Cisco support portal and upload(e.g. scp) it to the vm. The first public release version is 7.7.1 

    **Note:** A Cisco support contract is required to download the image from the official website, in case of any issue, please contact your Cisco account representative.

    **Note:** The docker image is inside the archive
  
+ Install [Docker](https://docs.docker.com/engine/install/ubuntu/)

```
  lab@xrdlab:~$ curl -fsSL https://get.docker.com -o get-docker.sh
  lab@xrdlab:~$ sudo sh get-docker.sh
```

+ Install Docker-Compose ```sudo apt install docker-compose```
+ Install the Cisco XRd Docker container

```
  lab@xrdlab:~/xrd-control-plane$ sudo docker load -i xrd-control-plane-container-x64.dockerv1.tgz
  a42828b8fe58: Loading layer [==================================================>]  1.179GB/1.179GB
  Loaded image: localhost/ios-xr:7.7.1
  lab@xrdlab:~/xrd-control-plane$ sudo docker images
  REPOSITORY         TAG       IMAGE ID       CREATED       SIZE
  localhost/ios-xr   7.7.1     dd8d741e50b2   3 weeks ago   1.15GB
  lab@xrdlab:~/xrd-control-plane$
```


## Clone the XRD-Tools repository

```
lab@xrdlab:~/github$ sudo git clone https://github.com/ios-xr/xrd-tools.git
Cloning into 'xrd-tools'...
remote: Enumerating objects: 69, done.
remote: Counting objects: 100% (69/69), done.
remote: Compressing objects: 100% (43/43), done.
remote: Total 69 (delta 27), reused 61 (delta 24), pack-reused 0
Receiving objects: 100% (69/69), 84.39 KiB | 3.25 MiB/s, done.
Resolving deltas: 100% (27/27), done.
lab@xrdlab:~/github$
```  

## Host-Check

As per the xrd-tools repo, first check whether the host vm is ready to run the XRd or not

```
lab@xrdlab:~/github/xrd-tools/scripts$ sudo ./host-check
==============================
Platform checks
==============================

base checks
-----------------------
PASS -- CPU architecture (x86_64)
PASS -- CPU cores (16)
PASS -- Kernel version (5.15)
PASS -- Base kernel modules
        Installed module(s): dummy, nf_tables
FAIL -- Cgroups version
        Cgroups version 2 is in use, but this is not supported by XRd.
        Please use cgroups version 1.
SKIP -- systemd mounts
        Skipped due to failed checks: Cgroups version
FAIL -- Inotify max user instances
        The kernel parameter fs.inotify.max_user_instances is set to 128 but
        should be at least 4000 (sufficient for a single instance) - the
        recommended value is 64000.
        This can be addressed by adding 'fs.inotify.max_user_instances=64000'
        to /etc/sysctl.conf or in a dedicated conf file under /etc/sysctl.d/.
        For a temporary fix, run:
          sysctl -w fs.inotify.max_user_instances=64000
PASS -- Inotify max user watches
        249593 - this is expected to be sufficient for 62 XRd instance(s).
INFO -- Core pattern (core files managed by the host)
PASS -- ASLR (full randomization)
INFO -- Linux Security Modules
        AppArmor is enabled. XRd is currently unable to run with the
        default docker profile, but can be run with
        '--security-opt apparmor=unconfined' or equivalent.

xrd-control-plane checks
-----------------------
PASS -- RAM
        Available RAM is 30.6 GiB.
        This is estimated to be sufficient for 15 XRd instance(s), although memory
        usage depends on the running configuration.
        Note that any swap that may be available is not included.

xrd-vrouter checks
-----------------------
FAIL -- CPU extensions
        Missing CPU extension(s): sse4_1, sse4_2, ssse3
        Please install the missing extension(s).
PASS -- RAM
        Available RAM is 30.6 GiB.
        This is estimated to be sufficient for 6 XRd instance(s), although memory
        usage depends on the running configuration.
        Note that any swap that may be available is not included.
FAIL -- Hugepages
        Hugepages are not enabled. These are required for XRd to function correctly.
        To enable hugepages, see the instructions at:
        https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt.
PASS -- Interface kernel driver (vfio-pci loaded)
FAIL -- IOMMU
        The kernel module vfio-pci cannot be used, as IOMMU is not enabled.
        IOMMU is recommended for security when using the vfio-pci kernel driver.
PASS -- Shared memory pages max size (17179869184.0 GiB)

==============================
Extra checks
==============================

docker checks
-----------------------
PASS -- Docker client (version 20.10.17)
PASS -- Docker daemon (running, version 20.10.17)
PASS -- Docker supports d_type

xr-compose checks
-----------------------
FAIL -- docker-compose
        Docker Compose not found (checked with 'docker-compose --version').
        Launching XRd topologies with xr-compose requires docker-compose.
        See installation instructions at https://docs.docker.com/compose/install/.
PASS -- PyYAML (installed)
FAIL -- Bridge iptables
        For xr-compose to be able to use Docker bridges, bridge IP tables must
        be disabled. Note that there may be security considerations associated
        with doing so.
        Bridge IP tables can be disabled by setting the kernel parameters
        net.bridge.bridge-nf-call-iptables and net.bridge.bridge-nf-call-ip6tables
        to 0. These can be modified by adding 'net.bridge.bridge-nf-call-iptables=0'
        and 'net.bridge.bridge-nf-call-ip6tables=0' to /etc/sysctl.conf or in a
        dedicated conf file under /etc/sysctl.d/.
        For a temporary fix, run:
          sysctl -w net.bridge.bridge-nf-call-iptables=0
          sysctl -w net.bridge.bridge-nf-call-ip6tables=0

==================================================================
!! Host NOT set up correctly for any XR platforms !!
------------------------------------------------------------------
Extra checks passed: docker
Extra checks failed: xr-compose
==================================================================
lab@xrdlab:~/github/xrd-tools/scripts$
```

### Fix as per the above output

+ Cgroups to v1, thanks to this [post](https://discuss.linuxcontainers.org/t/error-the-image-used-by-this-instance-requires-a-cgroupv1-host-system-when-using-clustering/13885)
  + Update the file ```/etc/default/grub``` with ```GRUB_CMDLINE_LINUX_DEFAULT="systemd.unified_cgroup_hierarchy=false"```
  + Update grub with ```sudo update-grub```

```
    lab@xrdlab:~$ sudo update-grub
    Sourcing file `/etc/default/grub'
    Sourcing file `/etc/default/grub.d/init-select.cfg'
    Generating grub configuration file ...
    Found linux image: /boot/vmlinuz-5.15.0-46-generic
    Found initrd image: /boot/initrd.img-5.15.0-46-generic
    Warning: os-prober will not be executed to detect other bootable partitions.
    Systems on them will not be added to the GRUB boot configuration.
    Check GRUB_DISABLE_OS_PROBER documentation entry.
    done
    lab@xrdlab:~$
```

  + Reboot the vm ```sudo reboot now```

+ Increasing the max user instance ```echo 'fs.inotify.max_user_instances=249593' >> /etc/sysctl.conf``` 
+ Disable Bridge iptables, thanks to this [post](https://gist.github.com/adamelliotfields/aa9cfa2aacbb767dcf09860b112ee2ed), the change is persitent

```
  echo 'br_netfilter' >> /etc/modules

  echo 'net.bridge.bridge-nf-call-iptables=1' >> /etc/sysctl.conf
  echo 'net.bridge.bridge-nf-call-ip6tables=1' >> /etc/sysctl.conf
```

+ To update the missing CPU extensions, stop the vm and then add ```-cpu qemu64,+ssse3,+sse4.1,+sse4.2``` in QEMU Custom options of the image in EVE-NG, but it is not applicable for the control-plane image.
  
  
## Host-Check again

```
lab@xrdlab:~/github/xrd-tools/scripts$ sudo ./host-check
==============================
Platform checks
==============================

base checks
-----------------------
PASS -- CPU architecture (x86_64)
PASS -- CPU cores (16)
PASS -- Kernel version (5.15)
PASS -- Base kernel modules
        Installed module(s): dummy, nf_tables
PASS -- Cgroups version (v1)
PASS -- systemd mounts
        /sys/fs/cgroup and /sys/fs/cgroup/systemd mounted correctly.
PASS -- Inotify max user instances
        249593 - this is expected to be sufficient for 62 XRd instance(s).
PASS -- Inotify max user watches
        249593 - this is expected to be sufficient for 62 XRd instance(s).
INFO -- Core pattern (core files managed by the host)
PASS -- ASLR (full randomization)
INFO -- Linux Security Modules
        AppArmor is enabled. XRd is currently unable to run with the
        default docker profile, but can be run with
        '--security-opt apparmor=unconfined' or equivalent.

xrd-control-plane checks
-----------------------
PASS -- RAM
        Available RAM is 30.7 GiB.
        This is estimated to be sufficient for 15 XRd instance(s), although memory
        usage depends on the running configuration.
        Note that any swap that may be available is not included.

xrd-vrouter checks
-----------------------
PASS -- CPU extensions (sse4_1, sse4_2, ssse3)
PASS -- RAM
        Available RAM is 30.7 GiB.
        This is estimated to be sufficient for 6 XRd instance(s), although memory
        usage depends on the running configuration.
        Note that any swap that may be available is not included.
FAIL -- Hugepages
        Hugepages are not enabled. These are required for XRd to function correctly.
        To enable hugepages, see the instructions at:
        https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt.
PASS -- Interface kernel driver (vfio-pci loaded)
FAIL -- IOMMU
        The kernel module vfio-pci cannot be used, as IOMMU is not enabled.
        IOMMU is recommended for security when using the vfio-pci kernel driver.
PASS -- Shared memory pages max size (17179869184.0 GiB)

==============================
Extra checks
==============================

docker checks
-----------------------
PASS -- Docker client (version 20.10.17)
PASS -- Docker daemon (running, version 20.10.17)
PASS -- Docker supports d_type

xr-compose checks
-----------------------
PASS -- docker-compose (version 1.29.2)
PASS -- PyYAML (installed)
PASS -- Bridge iptables (disabled)

==================================================================
XR platforms supported: xrd-control-plane
XR platforms NOT supported: xrd-vrouter
------------------------------------------------------------------
Extra checks passed: docker, xr-compose
==================================================================
lab@xrdlab:~/github/xrd-tools/scripts$
```

The checks passed and the xrd-control-plane platform is now supported

## Launch XRd

Run the launch-xrd script as below to run a container from the installed image

```
lab@xrdlab:~/github/xrd-tools/scripts$ sudo ./launch-xrd localhost/ios-xr:7.7.1
systemd 230 running in system mode. (+PAM +AUDIT +SELINUX +IMA -APPARMOR +SMACK +SYSVINIT +UTMP -LIBCRYPTSETUP -GCRYPT -GNUTLS +ACL +XZ -LZ4 -SECCOMP +BLKID -ELFUTILS +KMOD -IDN)
Detected virtualization docker.
Detected architecture x86-64.

Welcome to Cisco XR (Base Distro SELinux and CGL) 9.0.0.26!

Set hostname to <2382f3358d2c>.
Initializing machine ID from random generator.
[  OK  ] Listening on Journal Socket.
[  OK  ] Created slice User and Session Slice.
[  OK  ] Reached target Paths.
[  OK  ] Reached target Swap.
[  OK  ] Reached target Remote File Systems.
[  OK  ] Created slice System Slice.
         Starting Remount Root and Kernel File Systems...
         Mounting Huge Pages File System...
[  OK  ] Reached target Slices.
         Mounting FUSE Control File System...
         Mounting Temporary Directory...
[  OK  ] Listening on Journal Socket (/dev/log).
[  OK  ] Listening on Syslog Socket.
         Starting Journal Service...
[  OK  ] Mounted Huge Pages File System.
[  OK  ] Mounted FUSE Control File System.
[  OK  ] Mounted Temporary Directory.
[  OK  ] Started Remount Root and Kernel File Systems.
         Starting Rebuild Hardware Database...
         Starting Load/Save Random Seed...
         Starting Create System Users...
         Starting Copy selected logs to var/log/old directories...
         Starting Monitoring of LVM2 mirrors, snapshots etc. using dmeventd or progress polling...
[  OK  ] Started Load/Save Random Seed.
[  OK  ] Started Create System Users.
[  OK  ] Started Journal Service.
         Starting Flush Journal to Persistent Storage...
[  OK  ] Started Flush Journal to Persistent Storage.
[  OK  ] Started Copy selected logs to var/log/old directories.
[  OK  ] Started Monitoring of LVM2 mirrors, snapshots etc. using dmeventd or progress polling.
[  OK  ] Reached target Local File Systems (Pre).
         Mounting /var/volatile...
         Mounting /mnt...
[  OK  ] Mounted /var/volatile.
[  OK  ] Mounted /mnt.
[  OK  ] Reached target Local File Systems.
         Starting Rebuild Journal Catalog...
         Starting Rebuild Dynamic Linker Cache...
         Starting Create Volatile Files and Directories...
[  OK  ] Started Rebuild Journal Catalog.
[  OK  ] Started Create Volatile Files and Directories.
         Starting Update UTMP about System Boot/Shutdown...
[  OK  ] Started Update UTMP about System Boot/Shutdown.
[  OK  ] Started Rebuild Hardware Database.
[  OK  ] Started Rebuild Dynamic Linker Cache.
         Starting Update is Completed...
[  OK  ] Started Update is Completed.
[  OK  ] Reached target System Initialization.
[  OK  ] Started Daily Cleanup of Temporary Directories.
[  OK  ] Reached target Timers.
[  OK  ] Listening on D-Bus System Message Bus Socket.
[  OK  ] Reached target Sockets.
[  OK  ] Reached target Basic System.
         Starting Resets System Activity Logs...
[  OK  ] Started IOS-XR XRd Core Watcher.
[  OK  ] Started Periodic Command Scheduler.
[  OK  ] Started Job spooling tools.
         Starting sysklogd Kernel Logging Service...
         Starting OpenSSH Key Generation...
[  OK  ] Started Service for factory reset.
         Starting IOS-XR Setup Non-Root related tasks...
         Starting System Logging Service...
[  OK  ] Started D-Bus System Message Bus.
[  OK  ] Reached target Network.
         Starting Permit User Sessions...
         Starting Xinetd A Powerful Replacement For Inetd...
         Starting /etc/rc.local Compatibility...
[  OK  ] Started Resets System Activity Logs.
[  OK  ] Started Permit User Sessions.
[  OK  ] Started /etc/rc.local Compatibility.
[  OK  ] Started Xinetd A Powerful Replacement For Inetd.
[  OK  ] Reached target Login Prompts.
[  OK  ] Reached target Multi-User System.
         Starting Update UTMP about System Runlevel Changes...
[  OK  ] Started Update UTMP about System Runlevel Changes.
[  OK  ] Started IOS-XR Setup Non-Root related tasks.
[  OK  ] Started OpenSSH Key Generation.
         Starting IOS-XR ISO Installation...
[  OK  ] Started System Logging Service.
[  OK  ] Started sysklogd Kernel Logging Service.
[  667.401907] xrnginstall[361]: 2022 Aug 20 18:41:13.106 UTC: Setting up dumper and build info files
[  667.529228] xrnginstall[361]: 2022 Aug 20 18:41:13.232 UTC: XR Lineup:  r77x.lu%EFR-00000436820
[  667.536537] xrnginstall[361]: 2022 Aug 20 18:41:13.240 UTC: XR Version: 7.7.1
[  667.548415] xrnginstall[361]: 2022 Aug 20 18:41:13.252 UTC: Completed set up of dumper and build info files
[  667.557122] xrnginstall[361]: 2022 Aug 20 18:41:13.261 UTC: Preparing IOS-XR (first boot)
[  667.724285] xrnginstall[361]: 2022 Aug 20 18:41:13.427 UTC: Checking if rollback cleanup is required
[  667.733597] xrnginstall[361]: 2022 Aug 20 18:41:13.436 UTC: Finished rollback cleanup stage
[  667.740465] xrnginstall[361]: 2022 Aug 20 18:41:13.443 UTC: Single node: starting XR
[  667.757563] xrnginstall[361]: 2022 Aug 20 18:41:13.461 UTC: xrnginstall completed successfully
[  OK  ] Started IOS-XR ISO Installation.
         Starting IOS-XR XRd...
[  OK  ] Started Cisco Directory Services.
[  OK  ] Started IOS-XR XRd.
         Starting IOS-XR Reaperd and Process Manager...
[  OK  ] Started IOS-XR Reaperd and Process Manager.
[  OK  ] Reached target XR installation and startup.


ios con0/RP0/CPU0 is now available





Press RETURN to get started.





This product contains cryptographic features and is subject to United
States and local country laws governing import, export, transfer and
use. Delivery of Cisco cryptographic products does not imply third-party
authority to import, export, distribute or use encryption. Importers,
exporters, distributors and users are responsible for compliance with
U.S. and local country laws. By using this product you agree to comply
with applicable laws and regulations. If you are unable to comply with
U.S. and local laws, return this product immediately.

A summary of U.S. laws governing Cisco cryptographic products may be
found at:
http://www.cisco.com/wwl/export/crypto/tool/stqrg.html

If you require further assistance please contact us by sending email to
export@cisco.com.



RP/0/RP0/CPU0:Aug 20 18:41:24.932 UTC: pyztp2[252]: %INFRA-ZTP-4-EXITED : ZTP exited

!!!!!!!!!!!!!!!!!!!! NO root-system username is configured. Need to configure root-system username. !!!!!!!!!!!!!!!!!!!!

         --- Administrative User Dialog ---


  Enter root-system username: RP/0/RP0/CPU0:Aug 20 18:41:28.381 UTC: smartlicserver[266]: %LICENSE-SMART_LIC-3-COMM_FAILED : Communications failure with the Cisco Smart Software Manager (CSSM) : Communications init failure
   co
  % Entry must not be null.

  Enter root-system username: cisco
  Enter secret:
  Enter secret again:
Use the 'configure' command to modify this configuration.
User Access Verification

Username: cisco
Password:


RP/0/RP0/CPU0:ios#show platform
Sat Aug 20 18:41:45.892 UTC
Node              Type                     State                    Config state
--------------------------------------------------------------------------------
0/RP0/CPU0        XRd-CP-C-01(Active)      IOS XR RUN               NSHUT
RP/0/RP0/CPU0:ios#show version
RP/0/RP0/CPU0:ios#
Sat Aug 20 18:41:48.353 UTC
Cisco IOS XR Software, Version 7.7.1 LNT
Copyright (c) 2013-2022 by Cisco Systems, Inc.

Build Information:
 Built By     : ingunawa
 Built On     : Mon Jul 25 06:07:25 UTC 2022
 Build Host   : iox-lnx-121
 Workspace    : /auto/srcarchive12/prod/7.7.1/xrd-control-plane/ws
 Version      : 7.7.1
 Label        : 7.7.1

cisco XRd Control Plane
cisco XRd-CP-C-01 processor with 32GB of memory
ios uptime is 0 minutes
XRd Control Plane Container


RP/0/RP0/CPU0:ios#sh int brief
Sat Aug 20 18:41:56.784 UTC

               Intf       Intf        LineP              Encap  MTU        BW
               Name       State       State               Type (byte)    (Kbps)
--------------------------------------------------------------------------------
                Nu0          up          up               Null  1500          0
     Mg0/RP0/CPU0/0  admin-down  admin-down               ARPA  1514    1000000

RP/0/RP0/CPU0:ios#
```

I couldn't get out of the docker container, so I had to stop the container by opening another terminal session to the vm.

## It works!

# Thank You!

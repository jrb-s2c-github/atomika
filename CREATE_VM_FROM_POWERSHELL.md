See https://www.tenforums.com/tutorials/138380-create-set-up-new-vhd-vhdx-file-windows-10-a.html

Download Ubuntu22 ISO image from official downloads

Google how to enable HyperV on Windows should it not be enabled

Ensure that HyperV has the Default Network Switch, otherwise create it. Virtual machines on the default switch are 
assigned internal IP's. It remains an outstanding task to have the same IP assigned after reboots.

One can also create a virtual switch with a dedicated adaptor, e.g. ethernet cable should your PC normally use Wi-Fi. 
Since the router at the other end of the ethernet cable assigns the IP address this is more stable than the Default Switch.
The best solution is actually to configure the router to use static IP's. The switch name can be changed in the "New-VM" 
command below using the -SwitchName setting.

Run this command to set the location of the Ubuntu22 boot image:
```
Set-Variable VDiskHome C:\Users\skaap\vm_disks\ISO
```
Copy, paste and run all the commands below to create in Ubuntu22 VM in one go:
```
Set-Variable Name ubuntu22AK
New-VM -Name $Name -MemoryStartupBytes 4GB -NewVHDPath $Name+'.vhdx' -NewVHDSizeBytes 40GB -SwitchName 'Default Switch' 
Set-VM -Name $Name -ProcessorCount 4 -StaticMemory 
Add-VMDvdDrive -VMName "$Name" -Path $VDiskHome\ubuntu-22.04.4-live-server-amd64.iso 
```
This sequence of commands should be run for as many nodes as are required in the cluster.
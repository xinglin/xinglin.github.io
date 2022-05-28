# KVM Notes


*  List storage pools  
        ``virsh pool-list --details``  
*  List all volumes for a pool  
        ``vol-list --details POOLNAME``  
*  To create a Qemu guest with a image in qcow2 format, we need to first create a storage pool first.  
	``virsh pool-define-as --name qcow2 --type dir --target /mnt/qemu-img``  
	``virsh pool-start qcow2``  
* To create a Qemu guest with bridged network  
	``sudo virt-install -n vm1 -r 1024 --disk path=/mnt/qemu-imgs/vm1.qcow2,bus=virtio,size=5,format=qcow2 -c /mnt/sda4/ubuntu-12.04.5-desktop-amd64.iso --network bridge=virbr0 --graphics vnc,listen=0.0.0.0 --noautoconsole -v``

* To create a qemu guest with qemu_system-x86_64 binary directly  
	``qemu-system-x86_64 -enable-kvm -m 1024 -name vm2 -hda /var/lib/libvirt/images/vm2.img -vnc :2 -device e1000,netdev=net1,mac=22:22:33:44:00:02  -netdev tap,id=net1 -cdrom /mnt/sdb1/ubuntu-14.04.1-desktop-i386.iso -boot d``  

* To get console access for guest VMs created with virt-install  
  - Create a guest VM  
	``sudo virt-install -n vm1 -r 256 --disk path=/var/lib/libvirt/images/vm1.img,bus=virtio,size=10 -c ubuntu-14.04-server-i386.iso --network network=default,model=virtio --graphics vnc,listen=0.0.0.0 --noautoconsole -v``  
  - Added console to kernel parameter, by editing the file /etc/default/grub and added "console=ttyS0" to the variable GRUB_CMDLINE_LINUX_DEFAULT.  

  - Create /etc/init/ttyS0.conf and change last line to be "exec /sbin/getty -8 38400 ttyS0 xterm" 
	``cp /etc/init/tty1.conf /etc/init/ttyS0.conf``  
	`` +exec /sbin/getty -8 38400 ttyS0 xterm``



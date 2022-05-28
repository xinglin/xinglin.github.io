# Clone a Single KVM/qemu Virtual Machine


I am working on virtual machine cloning for security testing for the TCloud project. 
We are looking for an efficient and transparent virtual machine cloning technique
that has minimal impact on the virtual machine to be cloned. After some investigation,
we found that KVM supports _live_ snapshot for both memory and disk state while 
Xen4.4 currently does not support this feature yet but could be patched to do live snapshot
for memory state easily. btrfs has very impressive performance in taking snapshots: 
its performance is the best when compared with qemu-img and lvm 
(finished in about 50 ms for a 4GB disk file). So, we could possibly use both Xen or KVM
for our project. The other requirement from the project is we want to provide this security testing 
as a cloud service. So, we want to use OpenStack as our cloud platform. We had a really hard
time in setting up a Xen+libvirt OpenStack cluster and we finally gave up on this approach
since it is not supported well yet (in the group C support). As far as I know, 
there is no online documentation on this topic, until I wrote [this one][xen-openstack]. 

Because of all the difficulties in setting up a Xen+libvirt OpenStck cluster, we decided
to go with KVM. Here is how we can clone a KVM virtual machine and restart it at another machine.  
*  Take a live snapshot at the source machine  
        ``snapshot-create-as vm1 snapshot1 "firt snapshot for vm1" --live --memspec file=/dev/shm/vm1.memsnapshot``  
*  Transfer the memory image, disk file, and configuration file to the destination machine  
        ``scp /dev/shm/vm1.memsnapshot dest:/mnt/``  
        ``scp /etc/libvirt/qemu/vm1.xml dest:/etc/libvirt/qemu/vm1.xml``  
        ``scp /var/lib/libvirt/images/vm1.img dest:/var/lib/libvirt/images/vm1.img``  
*  Restart(Restore) vm1 at destination machine  
        ``virsh restore /mnt/vm1.memsnapshot``  

[xen-openstack]: http://wiki.xenproject.org/wiki/Xen_via_libvirt_for_OpenStack_juno


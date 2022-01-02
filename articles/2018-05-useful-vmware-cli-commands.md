## Useful VMware CLI commands

If you're running on the free version of VMware ESX 5 then you know the pangs of having to manually perform operations which are much easier to do with VMware Essentials or higher licenses.

This is not really a guide, more it's a collection of useful command line utilities which you can use to save yourself some time.

##### Initiate auto-shutdown of VMs

```
$ /bin/vmware-autostart.sh stop
```

Note: This requires VMware tools (and running) and auto shutdown rules configured

##### Initiate auto power on of VMs

```
$ /bin/vmware-autostart.sh start
```

Note: This requires VMware tools (and running) on the VM and auto shutdown rules configured

##### Unregister multiple VMs

```
$ for vm in <vm_name>; do vim-cmd /vmsvc/unregister /vmfs/volumes/<volume_name>/${vm}/${vm}.vmx; done
```

Note: This is the same as "Remove from inventory"

##### Register multiple VMs

```
$ for vm in <vm_name>; do vim-cmd /solo/register /vmfs/volumes/<volume_name>/${vm}/${vm}.vmx; done
```

Note: This is the same as "Add to inventory"

##### Power on multiple VMs

```
$ for vm in <vm_name>; do vim-cmd /vmsvc/power.on /vmfs/volumes/<volume_name>/${vm}/${vm}.vmx; done
```

Note: This is the same as CTRL+B them

##### Rename a VM

```
$ /vmfs/volumes/rename_vm.sh <datastoreName> <directory_name_of_copied_VM> <old_vm_name> <new_vm_name>
```

Note: You'll need to download the script from the iitggithub page and scp it to the VM host datastore.

....and that's all we have so far. If you have any more tips and tricks, let me know.

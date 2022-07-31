## Full install with UEFI && Encryption, Mostly I do this:


 
### Step 0 - Disk Partitioning
<br/>

**fdisk /dev/vda** <br/>

**/dev/vda1** &nbsp;&nbsp;&nbsp; # EFI System type_id: 1 ***Last Sector +512M***<br/>
**/dev/vda2** &nbsp;&nbsp;&nbsp; # BOOT type_id: SKIP_THIS_ONE ***Last Sector+512M***<br/>
**/dev/vda3** &nbsp;&nbsp;&nbsp; # LVM type_id: 43 ***Last Sector 100%FREE***<br/>
<br/>

### Step 1 - Making File System

`After this we need to make our filesystem`<br/>
**mkfs.fat -F32 /dev/vda1**&nbsp;&nbsp;&nbsp;&nbsp;`# for EFI`<br/>
**mkfs.ext4 /dev/vda2** &nbsp;&nbsp;&nbsp;&nbsp;`# for the future BOOT`<br/>
#### Step 1.1 - Enctyption part
`# And we also need to encrypt our /dev/vda3`<br/>
cryptosetup luksFormat /dev/vda3 <br/>
`<Make your self a good pass>` <br/>

`Now we need to unlock it`<br/>
cryptosetup open --type luks /dev/vda3 lvm<br/>
#### Step 1.2 - LVM partition
pvcreate --dataalignment 1m /dev/mapper/lvm<br/>
vgcreate volgroup0 /dev/mapper/lvm<br/>
lvcreate -L 30GB volgroup0 -n lv_root<br/>
lvcreate -l 100%FREE volgroup0 -n lv_home<br/>

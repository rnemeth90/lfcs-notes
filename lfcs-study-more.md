- [x] PAM
- [x] sed
- [x] autofs
- [x] egrep / regex
- [x] disk quotas
- [x] rsync
- [x] Virtual Machines
- [x] Containers
- [x] dnf / managing software
- [x] archive/compress files

</br>

# NOTES

- /etc/fstab network share device option:
  - _netdev

- redirecting stdin/stdout
  - sudo grub2-install /dev/vda > grub.txt 2>&1

- Disk Encryption:
  - sudo cryptsetup --verify-passphrase open --type plain /dev/sde1 myEncryptedDisk
  - sudo cryptsetup luksFormat /dev/sdd

- PAM
  - files in /etc/pam.d/

- umask
  - 0002 = standard users
  - 0022 = root
  - 666 = files
  - 777 = directories

- disk quotas
  - `sudo dnf install quota -y`
  - `UUID=0cd66824-77c1-4794-b83e-071b1ac8693b /                       xfs     default,usrquota,grpquota       0 0`
  - On Ext4 file systems (not necessary on XFS file systems):
    - `sudo quotacheck --create-files --user --group /dev/sdc1`
    - Turn on quota limits: `sudo quotaon /mnt/myDisk`
  - To edit quotas for user Aaron: `sudo edquota --user aaron`

- regex
  - Lines begin with:
    - `grep ^findme myfile`
  - Line ends with:
    - `grep findme$ myfile`
  - Match any character in position
    - `grep f.ndme myfile`
  - Search for any string containing any characters between forward slashes
    - `grep -r '/.*/' myfile`
  - Search for any string containing any characters between parentheses
    - `grep -r '(.*)' myfile`
  - Find all strings containing at least 1 zero
    - `egrep 0+ /`
  - Find all strings containing at least 3 zeros
    - `egrep -r 0{3,} /`
  - Find all strings containing at most 3 zeros
    - `egrep -r 0{,3} /`
  - Find enabled/disabled:
    - `egrep -r enabled|disabled /etc`
  - Find enable/enabled or disable/disabled:
    - `egrep -r enabled?|disabled? /etc`
  - Match any lower case letter (cat or cut)
    - `egrep -r c[a,u]t /etc/`


- find
  - `sudo find ./collection/ -mmin -60 -type f -exec cp --target-directory=/opt/oldfiles/ {} +\;`
  - `sudo find /home/bob/collection/ -type f -user adm -exec cp {} /opt/admfiles/  \;`

       -perm mode
              File's permission bits are exactly mode (octal or symbolic).  Since an exact match is required, if you want to use this form for symbolic modes, you may have to specify a
              rather complex mode string.  For example `-perm g=w' will only match files which have mode 0020 (that is, ones for which group write permission  is  the  only  permission
              set).   It  is more likely that you will want to use the `/' or `-' forms, for example `-perm -g=w', which matches any file with group write permission.  See the EXAMPLES
              section for some illustrative examples.

       -perm -mode
              All of the permission bits mode are set for the file.  Symbolic modes are accepted in this form, and this is usually the way in which you would want  to  use  them.   You
              must specify `u', `g' or `o' if you use a symbolic mode.  See the EXAMPLES section for some illustrative examples.

       -perm /mode
              Any  of  the  permission bits mode are set for the file.  Symbolic modes are accepted in this form.  You must specify `u', `g' or `o' if you use a symbolic mode.  See the
              EXAMPLES section for some illustrative examples.  If no permission bits in mode are set, this test matches any file (the idea here is to be consistent with the  behaviour
              of -perm -000).



- virsh
  - `sudo dnf install libvirt qemu-kvm -y`
  - `sudo vi testmachine.xml`
  - ```
    <domain type="qemu">
        <name>TestMachine</name>
        <memory unit="GiB">1</memory>
        <vcpu>1</vcpu>
        <os>
                <type arch='x86_64'>hvm</type>
        </os>
    </domain>
    ```

  - `virsh define testmachine.xml`
  - `virsh list --all` to show ALL machines, including powered off
  - `virsh start TestMachine` to start a machine
  - `virsh reboot TestMachine`
  - `virsh undefine --remove-all-storage TestMachine` to completely delete a VM
  - `virsh setvcpus TestMachine 2 --config`
  - `virsh dominfo TestMachine`
  - `virsh destroy TestMachine` to power off a VM
  - `virsh shutdown TestMachine` to shut down a VM gracefully
  - To change memory:
    - Adjust the max: `sudo virsh setmaxmem VM2 80M --config`
    - Change the memory: `sudo virsh setmem VM2 80M --config`
    - Power the VM off and then start it to apply the change:
      - `sudo virsh destroy VM2`
      - `sudo virsh start VM2`

- grub:
  - BIOS Systems
    - sudo grub2-mkconfig -o /boot/grub2/grub.cfg
  - EFI Systems

- Managing Software
  - `sudo dnf config-manager repolist --all`
  - `sudo dnf config-manager --enable powertools`
  - `sudo dnf config-manager --add https://myRepo.com`
  - `sudo dnf provides /etc/anacrontab`
  - `sudo dnf whatprovides /etc/anacrontab`
  - `sudo dnf repoquery --list nginx`
  - `sudo dnf group list --hidden`
  - `sudo dnf autoremove`
  - `sudo dnf history`


- Verify file systems
  - `xfs_repair /dev/vdb1`
  - `fsck.ext4 -v -f -p /dev/vdb2`
    - -v = verbose
    - -f = force
    - -p = preen mode (will fix errors)

`systemctl list-dependencies`


- Modify Kernel Params
  - `sysctl -a` to list current params
  - `sysctl -w net.ipv4.tcp_mem=4` to change a param
    - This is not persistent
  - To make persistent, add a file with a `.conf` extension to `/etc/sysctl.d`
    - `sudo vi /etc/sysctl.d/10-swappiness.conf`


- Routes added via the `ip` command are temporary. To make them permenant
  - `sudo nmcli connection modify eth1 +ipv4.routes "192.168.0.0/24 172.28.128.100"`
  - `sudo nmcli device reapply eth1`


- Install, Configure, and Troubleshoot BootLoaders
  - To regenerate grub2 config:
    - Boot into recovery media, then:
      - `chroot /mnt/sysroot`
      - To regenerate grub config for BIOS system
        - `grub2-mkconfig -o /boot/grub2/grub.cfg`
      - To regenerate grub config for EFI system:
        - `grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg`
  - To reinstall the boot loader:
    - BIOS systems:
      -  Use `lsblk` to look at block devices. Try to identify the boot device
      -  Use `grub2-install /dev/sda` to install grub to the boot device
   -  EFI Systems:
      -  Use `dnf reinstall grub2-efi grub2-efi-modules shim` to reinstall grub to the boot device


- Password aging default policies are stored in `/etc/login.defs`

- `swaplabel` can be used to configure the label of swap partitions
  - `sudo swaplabel -L myLabel /dev/sda1`

- after adding an entry to `/etc/exports`, you need to restart the nfs daemon
- After adding an entry to the `/etc/exports` file, you need to run `sudo exportfs -av` to complete the export process
- When adding an NFS share to `/etc/fstab`, use the `_netdev` option to ensure that the network is online before attempting to connect to the share:
  `192.168.1.0:/common  /local  nfs   _netdev   0 0`


- Manage and Configure LVM Storage
  - LVM creates an abstraction layer between physical storage and the file system, allowing the file system to be resized and span across multiple disks
  - Physical volumes are grouped into logical groups of volumes, called Volume Groups. These Volume Groups are then divided into Logical Volumes
  - LVM Acronyms
    - PV = Physical Volume, real physical storage devices
      - `sudo lvmdiskscan` can be used to list devices that may be used as physical volumes
        ```
        04:52:03 azureadmin@centos01 ~ → sudo lvmdiskscan
        /dev/sda1  [     500.00 MiB]
        /dev/sda2  [      29.02 GiB]
        /dev/sda15 [     495.00 MiB]
        /dev/sdb1  [     <64.00 GiB]
        /dev/sdf   [       5.00 GiB]
        /dev/sdh   [       5.00 GiB]
        /dev/sdi   [       5.00 GiB]
        3 disks
        4 partitions
        0 LVM physical volume whole disks
        0 LVM physical volumes
        ```
      - `'sudo pvcreate /dev/sdc /dev/sdd /dev/sde` can be used to create a new LVM volume from 3 disks
        ```
        04:52:06 azureadmin@centos01 ~ → sudo pvcreate /dev/sdf /dev/sdh /dev/sdi
        Physical volume "/dev/sdf" successfully created.
        Physical volume "/dev/sdh" successfully created.
        Physical volume "/dev/sdi" successfully created.
        ```
      - `sudo pvs` can be used to list physical volumes used by LVM
        ```
        04:52:19 azureadmin@centos01 ~ → sudo pvs
        PV         VG Fmt  Attr PSize PFree
        /dev/sdf      lvm2 ---  5.00g 5.00g
        /dev/sdh      lvm2 ---  5.00g 5.00g
        /dev/sdi      lvm2 ---  5.00g 5.00g
        ```

    - VG = Volume Group
      - After LVM has physical devices (pvs), you add the pvs to a volume group (vg). This tells LVM how it can use the storage capacity
      - `sudo vgcreate my_volume /dev/sdf /dev/sdh` will create a new VG with 2 PV
        ```
        04:58:13 azureadmin@centos01 ~ → sudo vgcreate my_vol /dev/sdf /dev/sdh
        Volume group "my_vol" successfully created
        ```
      - Once disks are added to a volume group, they are seen by the system as one contiguous block of storage
      - You can add another disk to the volume group using `vgextend`
        ```
        05:00:06 azureadmin@centos01 ~ → sudo vgextend my_vol /dev/sdi
        Volume group "my_vol" successfully extended
        ```
      - use `sudo vgs` to view the status of Volume Groups:
        ```
        05:00:14 azureadmin@centos01 ~ → sudo vgs
        VG     #PV #LV #SN Attr   VSize   VFree
        my_vol   3   0   0 wz--n- <14.99g <14.99g
        ```
      - use `sudo vgreduce my_vol /dev/sdi` to remove a physical volume from a volume group
        ```
        05:00:39 azureadmin@centos01 ~ → sudo vgreduce my_vol /dev/sdi
        Removed "/dev/sdi" from volume group "my_vol"
        ```

    - LV = Logical Volume
      - A logical volume is similar to a partition
      - `sudo lvcreate --size 2G --name partition1 my_vol` can be used to create a logical volume of 2 gigabytes
        ```
        05:02:22 azureadmin@centos01 ~ → sudo lvcreate --size 2G --name partition1 my_vol
        Logical volume "partition1" created.
        ```
      - You can view logical volumes using `sudo lvs`
        ```
        05:03:49 azureadmin@centos01 ~ → sudo lvs
        LV         VG     Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
        partition1 my_vol -wi-a----- 2.00g
        partition2 my_vol -wi-a----- 6.00g
        ```
      - to tell a logical volume to use all space on a logical volume, use `sudo lvresize`
        ```
        05:03:50 azureadmin@centos01 ~ → sudo lvresize --extents 100%VG my_vol/partition1
        Reducing 100%VG to remaining free space 3.99 GiB in VG.
        Size of logical volume my_vol/partition1 changed from 2.00 GiB (512 extents) to 3.99 GiB (1022 extents).
        Logical volume my_vol/partition1 successfully resized.
        ```
      - the path to LVs on the system can be found using `lvdisplay`
        ```
        05:08:32 azureadmin@centos01 ~ → sudo lvdisplay  | grep "LV Path"
        LV Path                /dev/my_vol/partition1
        LV Path                /dev/my_vol/partition2
        ```
      - You can then add a file system to a LV using common file system management commands
        ```
        05:08:41 azureadmin@centos01 ~ → sudo mkfs.xfs /dev/my_vol/partition1
        meta-data=/dev/my_vol/partition1 isize=512    agcount=4, agsize=261632 blks
                =                       sectsz=4096  attr=2, projid32bit=1
                =                       crc=1        finobt=1, sparse=1, rmapbt=0
                =                       reflink=1
        data     =                       bsize=4096   blocks=1046528, imaxpct=25
                =                       sunit=0      swidth=0 blks
        naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
        log      =internal log           bsize=4096   blocks=2560, version=2
                =                       sectsz=4096  sunit=1 blks, lazy-count=1
        realtime =none                   extsz=4096   blocks=0, rtextents=0
        Discarding blocks...Done.
        ```
      - If the LV contains a file system, you must take extra caution when resizing it. You must pass the `--resizefs` parameter to `lvresize`
      - `sudo lvresize --resizefs --size 3G my_vol\partition1`
      - XFS file system shrinking is not supported

- Encrypted Storage
  - use `cryptsetup --verify-passphrase open --type plain /dev/vde mysecuredisk` to encrypt a disk using plain mode
  - To use LUKS encryption, which is much easier to use:
    - `sudo cryptsetup luksFormat /dev/vde`
      ```
      05:19:37 azureadmin@centos01 ~ → sudo cryptsetup luksFormat /dev/sdi

      WARNING!
      ========
      This will overwrite data on /dev/sdi irrevocably.

      Are you sure? (Type 'yes' in capital letters): YES
      Enter passphrase for /dev/sdi:
      Verify passphrase:
      ```
    - You can change the encryption key easily using LUKS: `sudo cryptsetup luksChangekey /dev/vde`


- Disk Quotas
  - `sudo dnf install quota` to install the package that helps manage storage quotas
    - on CentOS these packages are usually installed by default
  - edit `/etc/fstab`:
    - Add a line similar to the following for the disk:
      - `/dev/vdb1       /mybackups      xfs     defaults,usrquota,grpquota 0 0`
  - If you are using EXT based file systems, you must run the following commands (not required for xfs file systems):
    - `sudo quotacheck --create-files --user --group /dev/vdb2`
      - This command creates the following files:
        - `aquota.group`
        - `aquota.user`
        - These files track how much space each user or group is using on the disk
    - turn on quota limits:
      - `sudo quotaon /mnt`
  - To edit quotas for a user:
    - `sudo edquota --user azureadmin`
  - Soft limits allow a user to still create files during the grace period. Hard limits do not.

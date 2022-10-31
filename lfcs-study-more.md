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


- `sudo nmcli device reapply`
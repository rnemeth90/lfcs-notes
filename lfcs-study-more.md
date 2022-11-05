# NOTES
- redirecting stdin/stdout
  - sudo grub2-install /dev/vda > grub.txt 2>&1

- Disk Encryption:
  - sudo cryptsetup --verify-passphrase open --type plain /dev/sde1 myEncryptedDisk
  - sudo cryptsetup luksFormat /dev/sdd
  - You can change the encryption key easily using LUKS: `sudo cryptsetup luksChangekey /dev/vde`

- PAM
  - files in /etc/pam.d/

- umask
  - 0002 = standard users
  - 0022 = root
  - 666 = files
  - 777 = directories

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

       -perm -mode
              All of the permission bits mode are set for the file. \

       -perm /mode
              Any  of  the  permission bits mode are set for the file.

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

- DNS
  - Allow queries from anywhere on the net:
    - Add to `/etc/named.conf`:
      - `allow-query { 0.0.0.0/0; };`
  - When copying the default zone file (to create your own), be sure to use the `--archive` option to preserve permissions/ownership

- for dovecot, to update the listening port, you need to edit `/etc/dovecot/conf.d/10-master.conf`

- You can make exceptions for a user in `/etc/ssh/sshd_config` by adding the following:
  - ```
    PasswordAuthentication no
    Match User ryan
      PasswordAuthentication yes
    ```
  - There is an example of this in /etc/ssh/sshd_config (at the bottom)

- example virtualhost can be found in: `/usr/share/doc/httpd/httpd-vhosts.conf`


  - `apachectl configtest`
  - SSL settings can be edited in /etc/httpd/conf.d/ssl.conf
  - `sudo htpasswd -c /etc/httpd/passwords ryan`

  - Restrict access to a page based on IP
    - ```
    - <Directory "/var/www/html/admin/">
        Require ip 127.0.0.1
      </Directory>
    - ```
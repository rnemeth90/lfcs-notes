- [x] PAM
- [x] sed
- [ ] autofs
- [x] egrep / regex
- [x] disk quotas
- [x] rsync
- [ ] Virtual Machines
- [x] Containers


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
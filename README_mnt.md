## For Mount Data to merge

### Temporal

1. sudo apt update and sudo apt install mergerfs
2. sudo mkdir -p /workAll
3. sudo mergerfs /work:/worktwo /workAll


### Permanent (if we want to)
3. sudo nano /etc/fstab
4. add this line
```
/work:/worktwo  /workAll  fuse.mergerfs  defaults,allow_other,use_ino,category.create=mfs,moveonenospc=true  0  0
```
5. sudo mount -a for test and if shows no error 
6. df -h | grep work
7. sudo reboot
8. after reboot to check df -h | grep work; If /workAll still exists: 🎉 permanent mount successful

## Export a file so another server can mount it.
On base server A:

1. sudo apt install nfs-kernel-server
2. sudo nano /etc/exports
/workAll  192.168.1.20(rw,sync,no_subtree_check,no_root_squash,fsid=0)
3. sudo exportfs -a
4. sudo systemctl restart nfs-kernel-server

On another client server to 
1. sudo apt install nfs-common
2. create the folder you want to mount to

Temporary test mount
sudo mount server A ip:/workAll ./

permanent mount
3. sudo nano /etc/fstab
serverA ip:/workAll  ./  nfs  defaults,_netdev  0  0

4.test before reboot sudo mount -a

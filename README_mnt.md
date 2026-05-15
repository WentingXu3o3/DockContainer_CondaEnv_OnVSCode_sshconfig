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
1. On base server A:
sudo apt install nfs-kernel-server

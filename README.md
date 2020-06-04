# Setup A NFS Share On Ubuntu 18.04 Desktop and Server

### Server Configuration


Create a new mount directory in `/mnt`:
```
sudo mkdir /mnt/<NewDirectoryName>
```    

Change ownership of the newly created directory from the `root` user:
```
cd /mnt/
sudo chown <user>:<group> <NewDirectoryName>
```

Identify the drive which will be mounted to the newly created directory by using:
```
sudo blkid
```

Locate the `UUID` and `type` of the drive which will be mounted.

Now add an entry to `/etc/fstab/`
```
UUID=<UUID> /mnt/<NewDIrectoryName> <type> defaults,user,rw,exec 0 0
```
Example:
```
UUID=3exxxxxx-f75c-404f-b63b-xxxxxxxxxxx /mnt/HDD1 ext3 defaults,user,rw,exec 0 0
```

To mount run:
```
sudo mount -a
```

Now install NFS server:
```
sudo apt-get install nfs-kernel-server
```

Check status of server
```
sudo systemctl status nfs-kernel-server
```

Once the server has been confirmed to be running edit `/etc/exports`
```
nano /etc/exports
```
  
Edit the exports file by adding the following entry:
```
/mnt/<NewDIrectoryName> 10.0.1.<ClientIp>/255.255.255.0(rw,sync,root_squash,subtree_check)
```
    
Now save and exit.

Now restart the server:
```
sudo systemctl restart nfs-kernel-server
```


### Client Configuration.

NFS client utils is already preinstalled, if not then install:
```
sudo apt install nfs-utils
```    

Create a directory to mount the `nfs` share:
```
sudo mkdir /mnt/<NewDirectoryName>
```    
    
Change ownership of the newly created directory from the `root` user:
```
cd /mnt/
sudo chown <user>:<group> <NewDirectoryName>
```

Now add an entry to `/etc/fstab/`
```
sudo nano /etc/fstab
```
    
Create a new entry:
```
<ServerIpAddress>:/mnt/<NewDirectoryName> /mnt/<NewDirectoryName> nfs rw,soft,intr,noatime,timeo=100 0 2

<------------------nfs server-----------> <-----local-----------> <type> <-----------permissions------->
```
    
Now check that the `nfs` share can be mounted locally:
```
sudo mount -a
```

### Permissions
    
rw            -  Read/Write access

soft / hard   -  Determines the recovery behavior of the NFS client after an NFS request times out.

intr / nointr -  This option is provided for backward compatibility

noatime       -  Setting this value disables the NFS server from updating the inodes access time

timeo=n       -  The time in deciseconds (tenths of a second) the NFS client waits for a response before it retries an NFS request.         
    

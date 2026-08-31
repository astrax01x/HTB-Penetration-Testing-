`Network File System` (`NFS`) is a network file system developed by Sun Microsystems and has the same purpose as SMB. Its purpose is to access file systems over a network as if they were local. However, it uses an entirely different protocol. [NFS](https://en.wikipedia.org/wiki/Network_File_System) is used between Linux and Unix systems. This means that NFS clients cannot communicate directly with SMB servers. NFS is an Internet standard that governs the procedures in a distributed file system

While NFS protocol version 3.0 (`NFSv3`), which has been in use for many years, authenticates the client computer, this changes with `NFSv4`. Here, as with the Windows SMB protocol, the user must authenticate.

#### ExportFS

	 root@nfs:~# echo '/mnt/nfs 10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports 
	 
	 root@nfs:~# systemctl restart nfs-kernel-server 
	 root@nfs:~# exportfs 
	 
	 /mnt/nfs 10.129.14.0/24

## Footprinting the Service

 When footprinting NFS, the TCP ports `111` and `2049` are essential. We can also get information about the NFS service and the host via RPC, as shown below in the example.

#### Nmap

	AstraX01@htb[/htb]$ sudo nmap 10.129.14.128 -p111,2049 -sV -sC

The `rpcinfo` NSE script retrieves a list of all currently running RPC services, their names and descriptions, and the ports they use.

	AstraX01@htb[/htb]$ sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049

Once we have discovered such an NFS service, we can mount it on our local machine. For this, we can create a new empty folder to which the NFS share will be mounted. Once mounted, we can navigate it and view the contents just like our local system.
#### Show Available NFS Shares

	AstraX01@htb[/htb]$ showmount -e 10.129.14.128

#### Mounting NFS Share

	AstraX01@htb[/htb]$ mkdir target-NFS 
	AstraX01@htb[/htb]$ sudo mount -t nfs 10.129.14.128:/ ./target-NFS/ -o nolock 
	AstraX01@htb[/htb]$ cd target-NFS 
	AstraX01@htb[/htb]$ tree .

There we will have the opportunity to access the rights and the usernames and groups to whom the shown and viewable files belong. Because once we have the usernames, group names, UIDs, and GUIDs, we can create them on our system and adapt them to the NFS share to view and modify the files.

#### List Contents with Usernames & Group Names

	AstraX01@htb[/htb]$ ls -l mnt/nfs/

#### List Contents with UIDs & GUIDs

	AstraX01@htb[/htb]$ ls -n mnt/nfs/

It is important to note that if the `root_squash` option is set, we cannot edit the `backup.sh` file even as `root`.

We can also use NFS for further escalation. For example, if we have access to the system via SSH and want to read files from another folder that a specific user can read, we would need to upload a shell to the NFS share that has the `SUID` of that user and then run the shell via the SSH user.

After we have done all the necessary steps and obtained the information we need, we can unmount the NFS share.
#### Unmounting

	AstraX01@htb[/htb]$ cd .. 
	AstraX01@htb[/htb]$ sudo umount ./target-NFS
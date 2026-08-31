[Secure Shell](https://en.wikipedia.org/wiki/Secure_Shell) (`SSH`) enables two computers to establish an encrypted and direct connection within a possibly insecure network on the standard port `TCP 22`. This is necessary to prevent third parties from intercepting the data stream and thus intercepting sensitive data. The SSH server can also be configured to only allow connections from specific clients. An advantage of SSH is that the protocol runs on all common operating systems. Since it is originally a Unix application, it is also implemented natively on all Linux distributions and MacOS. SSH can also be used on Windows, provided we install an appropriate program.

`SSH-2`, also known as SSH version 2, is a more advanced protocol than SSH version 1 in encryption, speed, stability, and security. For example, `SSH-1` is vulnerable to `MITM` attacks, whereas SSH-2 is not.

OpenSSH has six different authentication methods:

1. Password authentication
2. Public-key authentication
3. Host-based authentication
4. Keyboard authentication
5. Challenge-response authentication
6. GSSAPI authentication

#### Public Key Authentication

In a first step, the SSH server and client authenticate themselves to each other. The server sends its `public host key` to the client, which the client uses to verify the server's identity. Only when contact is first established there is a risk of a third party interposing itself between the two participants and thus intercepting the connection. A `host key` cannot be imitated because it is a unique public-private `key pair`, and an attacker cannot forge the private key’s signature without access to it

## Footprinting the Service

One of the tools we can use to fingerprint the SSH server is [ssh-audit](https://github.com/jtesta/ssh-audit). It checks the client-side and server-side configuration and shows some general information and which encryption algorithms are still used by the client and server.
#### SSH-Audit

	AstraX01@htb[/htb]$ git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit 
	AstraX01@htb[/htb]$ ./ssh-audit.py 10.129.14.132

#### Change Authentication Method

	AstraX01@htb[/htb]$ ssh -v cry0l1t3@10.129.14.132

For potential brute-force attacks, we can specify the authentication method with the SSH client option `PreferredAuthentications`.

	AstraX01@htb[/htb]$ ssh -v cry0l1t3@10.129.14.132 -o PreferredAuthentications=password

## Rsync

[Rsync](https://linux.die.net/man/1/rsync) is a fast and efficient tool for locally and remotely copying files. It can be used to copy files locally on a given machine and to/from remote hosts. It is highly versatile and well-known for its delta-transfer algorithm. This algorithm reduces the amount of data transmitted over the network when a version of the file already exists on the destination host.

It does this by sending only the differences between the source files and the older version of the files that reside on the destination server.

#### Scanning for Rsync

	AstraX01@htb[/htb]$ sudo nmap -sV -p 873 127.0.0.1

#### Probing for Accessible Shares

	AstraX01@htb[/htb]$ nc -nv 127.0.0.1 873

#### Enumerating an Open Share

Here we can see a share called `dev`, and we can enumerate it further.

	AstraX01@htb[/htb]$ rsync -av --list-only rsync://127.0.0.1/dev

From the above output, we can see a few interesting files that may be worth pulling down to investigate further. We can also see that a directory likely containing SSH keys is accessible. From here, we could sync all files to our attack host with the command `rsync -av rsync://127.0.0.1/dev`. If Rsync is configured to use SSH to transfer files, we could modify our commands to include the `-e ssh` flag, or `-e "ssh -p2222"` if a non-standard port is in use for SSH.

## R-Services

| **Command** | **Service Daemon** | **Port** | **Transport Protocol** | **Description**                                                                                                                                                                                                                                                            |
| ----------- | ------------------ | -------- | ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `rcp`       | `rshd`             | 514      | TCP                    | Copy a file or directory bidirectionally from the local system to the remote system (or vice versa) or from one remote system to another. It works like the `cp` command on Linux but provides `no warning to the user for overwriting existing files on a system`.        |
| `rsh`       | `rshd`             | 514      | TCP                    | Opens a shell on a remote machine without a login procedure. Relies upon the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files for validation.                                                                                                                 |
| `rexec`     | `rexecd`           | 512      | TCP                    | Enables a user to run shell commands on a remote machine. Requires authentication through the use of a `username` and `password` through an unencrypted network socket. Authentication is overridden by the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files. |
| `rlogin`    | `rlogind`          | 513      | TCP                    | Enables a user to log in to a remote host over the network. It works similarly to `telnet` but can only connect to Unix-like hosts. Authentication is overridden by the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files.                                     |
The /etc/hosts.equiv file contains a list of trusted hosts and is used to grant access to other systems on the network. When users on one of these hosts attempt to access the system, they are automatically granted access without further authentication.

#### /etc/hosts.equiv

	AstraX01@htb[/htb]$ cat /etc/hosts.equiv

Now that we have a basic understanding of `r-commands`, let's do some quick footprinting using `Nmap`

#### Scanning for R-Services

	AstraX01@htb[/htb]$ sudo nmap -sV -p 512,513,514 10.0.17.2

#### Logging in Using Rlogin

	AstraX01@htb[/htb]$ rlogin 10.0.17.2 -l htb-student 
	Last login: Fri Dec 2 16:11:21 from localhost 
	
	[htb-student@localhost ~]$


We have successfully logged in under the `htb-student` account on the remote host due to the misconfigurations in the `.rhosts` file. Once successfully logged in, we can also abuse the `rwho` command to list all interactive sessions on the local network by sending requests to the UDP port 513.

#### Listing Authenticated Users Using Rwho

	AstraX01@htb[/htb]$ rwho

#### Listing Authenticated Users Using Rusers

	AstraX01@htb[/htb]$ rusers -al 10.0.17.5


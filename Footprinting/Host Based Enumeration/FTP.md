The `File Transfer Protocol` (`FTP`) is one of the oldest protocols on the Internet. The FTP runs within the application layer of the TCP/IP protocol stack. Thus, it is on the same layer as `HTTP` or `POP`. These protocols also work with the support of browsers or email clients to perform their services. There are also special FTP programs for the File Transfer Protocol.

 *we need credentials to use FTP on a server. We also need to know that FTP is a `clear-text` protocol that can sometimes be sniffed if conditions on the network are right. However, there is also the possibility that a server offers `anonymous FTP`. The server operator then allows any user to upload or download files via FTP without using a password. Since there are security risks associated with such a public FTP server, the options for users are usually limited*

## TFTP

`Trivial File Transfer Protocol` (`TFTP`) is simpler than FTP and performs file transfers between client and server processes. However, it `does not` provide user authentication and other valuable features supported by FTP. In addition, while FTP uses TCP, TFTP uses `UDP`, making it an unreliable protocol and causing it to use UDP-assisted application layer recovery.

Let us take a look at a few commands of `TFTP`:

|**Commands**|**Description**|
|---|---|
|`connect`|Sets the remote host, and optionally the port, for file transfers.|
|`get`|Transfers a file or set of files from the remote host to the local host.|
|`put`|Transfers a file or set of files from the local host onto the remote host.|
|`quit`|Exits tftp.|
|`status`|Shows the current status of tftp, including the current transfer mode (ascii or binary), connection status, time-out value, and so on.|
|`verbose`|Turns verbose mode, which displays additional information during file transfer, on or off.|
Unlike the FTP client, `TFTP` does not have directory listing functionality.

## Default Configuration

One of the most used FTP servers on Linux-based distributions is [vsFTPd](https://security.appspot.com/vsftpd.html). The default configuration of vsFTPd can be found in `/etc/vsftpd.conf`, and some settings are already predefined by default.
#### Install vsFTPd

	AstraX01@htb[/htb]$ sudo apt install vsftpd

There are many different alternatives to it, which also bring, among other things, many more functions and configuration options with them. We will use the vsFTPd server because it is an excellent way to show the configuration possibilities of an FTP server in a simple and easy-to-understand way without going into the details of the man pages.
#### vsFTPd Config File

	 AstraX01@htb[/htb]$ cat /etc/vsftpd.conf | grep -v "#"

In addition, there is a file called `/etc/ftpusers` that we also need to pay attention to, as this file is used to deny certain users access to the FTP service. In the following example, the users `guest`, `john`, and `kevin` are not permitted to log in to the FTP service, even if they exist on the Linux system.
#### FTPUSERS

	 AstraX01@htb[/htb]$ cat /etc/ftpusers

s soon as we connect to the vsFTPd server, the `response code 220` is displayed with the banner of the FTP server. Often this banner contains the description of the `service` and even the `version` of it. It also tells us what type of system the FTP server is. One of the most common configurations of FTP servers is to allow `anonymous` access, which does not require legitimate credentials but provides access to some files. Even if we cannot download them, sometimes just listing the contents is enough to generate further ideas and note down information that will help us in another approach.

#### Anonymous Login

	 AstraX01@htb[/htb]$ ftp 10.129.14.136
	 ftp> ls

 However, to get the first overview of the server's settings, we can use the following command:
#### vsFTPd Status

	 ftp> status

Some commands should be used occasionally, as these will make the server show us more information that we can use for our purposes. These commands include `debug` and `trace`.

#### vsFTPd Detailed Output

	 ftp> debug
	 ftp> trace
	 ftp> ls

#### Hiding IDs - YES

    ftp> ls 
    
    ---> TYPE A 200 Switching to ASCII mode. 
    ftp: setsockopt (ignored): Permission denied 
    ---> PORT 10,10,14,4,223,101 200 
    PORT command successful. Consider using PASV. 
    ---> LIST 
    150 Here comes the directory listing. 
    -rw-rw-r--    1 ftp     ftp      8138592 Sep 14 16:54 Calender.pptx 
    drwxrwxr-x    2 ftp     ftp         4096 Sep 14 17:03 Clients 
    drwxrwxr-x    2 ftp     ftp         4096 Sep 14 16:50 Documents 
    drwxrwxr-x    2 ftp     ftp         4096 Sep 14 16:50 Employees 
    -rw-rw-r--    1 ftp     ftp           41 Sep 14 16:45 Important Notes.txt 
    -rw-------    1 ftp     ftp            0 Sep 15 14:57 testupload.txt 
    226 Directory send OK.

This setting is a security feature to prevent local usernames from being revealed. With the usernames, we could attack the services like FTP and SSH and many others with a brute-force attack in theory.

*Another helpful setting we can use for our purposes is the `ls_recurse_enable=YES`. This is often set on the vsFTPd server to have a better overview of the FTP directory structure, as it allows us to see all the visible content at once.*

#### Recursive Listing

	 ftp> ls -R

#### Download a File

	 ftp> ls
	 ftp> get Important\ Notes.txt
	 ftp> exit

We also can download all the files and folders we have access to at once. This is especially useful if the FTP server has many different files in a larger folder structure. However, this can cause alarms because no one from the company usually wants to download all files and content all at once
#### Download All Available Files

	 AstraX01@htb[/htb]$ wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136

Once we have downloaded all the files, `wget` will create a directory with the name of the IP address of our target. All downloaded files are stored there, which we can then inspect locally.

	 AstraX01@htb[/htb]$ tree .

#### Upload a File

	AstraX01@htb[/htb]$ touch testupload.txt

With the `PUT` command, we can upload files in the current folder to the FTP server.

	 ftp> put testupload.txt
## Footprinting the Service

Footprinting using various network scanners is also a handy and widespread approach. These tools make it easier for us to identify different services, even if they are not accessible on standard ports. One of the most widely used tools for this purpose is Nmap. Nmap also brings the [Nmap Scripting Engine](https://nmap.org/book/nse.html) (`NSE`), a set of many different scripts written for specific services. More information on the capabilities of Nmap and NSE can be found in the [Network Enumeration with Nmap](https://academy.hackthebox.com/course/preview/network-enumeration-with-nmap) module. We can update this database of NSE scripts with the command shown.

#### Nmap FTP Scripts

	 AstraX01@htb[/htb]$ sudo nmap --script-updatedb
 
 All the NSE scripts are located on the Pwnbox in `/usr/share/nmap/scripts/`, but on our systems, we can find them using a simple command.

	 AstraX01@htb[/htb]$ find / -type f -name ftp* 2>/dev/null | grep scripts

#### Nmap

	 AstraX01@htb[/htb]$ sudo nmap -sV -p21 -sC -A 10.129.14.136

The default script scan is based on the services' fingerprints, responses, and standard ports. Once Nmap has detected the service, it executes the marked scripts one after the other, providing different information. For example, the [ftp-anon](https://nmap.org/nsedoc/scripts/ftp-anon.html) NSE script checks whether the FTP server allows anonymous access. If so, the contents of the FTP root directory are rendered for the anonymous user.
#### Nmap Script Trace

	 AstraX01@htb[/htb]$ sudo nmap -sV -p21 -sC -A 10.129.14.136 --script-trace

#### Service Interaction

	AstraX01@htb[/htb]$ nc -nv 10.129.14.136 21
	AstraX01@htb[/htb]$ telnet 10.129.14.136 21

It looks slightly different if the FTP server runs with TLS/SSL encryption. Because then we need a client that can handle TLS/SSL. For this, we can use the client `openssl` and communicate with the FTP server. The good thing about using `openssl` is that we can see the SSL certificate, which can also be helpful.

	 AstraX01@htb[/htb]$ openssl s_client -connect 10.129.14.136:21 -starttls ftp


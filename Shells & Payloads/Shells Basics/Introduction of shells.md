A `shell` is a program that provides a computer user with an interface to input instructions into the system and view text output (Bash, Zsh, cmd, and PowerShell

## Why Get a Shell?

Remember that the shell gives us direct access to the `OS`, `system commands`, and `file system`. So if we gain access, we can start enumerating the system for vectors that may allow us to escalate privileges, pivot, transfer files, and more. If we don't establish a shell session, we are pretty limited on how far we can get on a target machine.

|**Perspective**|**Description**|
|---|---|
|`Computing`|The text-based userland environment that is utilized to administer tasks and submit instructions on a PC. Think Bash, Zsh, cmd, and PowerShell.|
|`Exploitation` `&` `Security`|A shell is often the result of exploiting a vulnerability or bypassing security measures to gain interactive access to a host. An example would be triggering [EternalBlue](https://www.cisecurity.org/wp-content/uploads/2019/01/Security-Primer-EternalBlue.pdf) on a Windows host to gain access to the cmd-prompt on a host remotely.|
|`Web`|This is a bit different. A web shell is much like a standard shell, except it exploits a vulnerability (often the ability to upload a file or script) that provides the attacker with a way to issue instructions, read and access files, and potentially perform destructive actions to the underlying host. Control of the web shell is often done by calling the script within a browser window.|

---

## Payloads Deliver us Shells

Within the IT industry as a whole, a `payload` can be defined in a few different ways:

- `Networking`: The encapsulated data portion of a packet traversing modern computer networks.
- `Basic Computing`: A payload is the portion of an instruction set that defines the action to be taken. Headers and protocol information removed.
- `Programming`: The data portion referenced or carried by the programming language instruction.
- `Exploitation & Security`: A payload is `code` crafted with the intent to exploit a vulnerability on a computer system. The term payload can describe various types of malware, including but not limited to ransomware.

#### Shell basics

- Replicate being able to get a bind and reverse shell.
- Bind Shell on Linux host.
- Reverse Shell on Windows Host.

#### Payload Basics

- Demonstrate launching a payload from MSF.
- Demonstrate searching and building a payload from PoC on ExploitDB.
- Demonstrate knowledge of payload creation.

#### Getting a Shell on Windows

- Using the recon results provided, craft or use a payload that will exploit the host and provide a shell back.

#### Getting a Shell on Linux

- Using the recon results provided, craft or use a payload to exploit the host and establish a shell session.

#### Landing a Web Shell

- Demonstrate knowledge of web shells and common web applications by identifying a common web application and its corresponding language.
- Using the recon results provided, deploy a payload that will provide shell access from your browser.

#### Spotting a Shell or Payload

- Detect the presence of a payload or interactive shell on a host by analyzing relevant information provided.

#### Final Challenge

- Utilize knowledge gained from the previous sections to select, craft, and deploy a payload to access the provided hosts. Once a shell has been acquired, grab the requested information to answer the challenge questions.


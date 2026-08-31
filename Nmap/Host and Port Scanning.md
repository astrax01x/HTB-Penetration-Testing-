
It is essential to understand how the tool we use works and how it performs and processes the different functions. We will only understand the results if we know what they mean and how they are obtained. Therefore we will take a closer look at and analyze some of the scanning methods

#### Scanning Top 10 TCP Ports

	 AstraX01@htb[/htb]$ sudo nmap 10.129.2.28 --top-ports=10

#### Nmap - Trace the Packets

	 AstraX01@htb[/htb]$ sudo nmap 10.129.2.28 -p 21 --packet-trace -Pn -n --disable-arp-ping

## Discovering Open UDP Ports

#### UDP Port Scan

	 AstraX01@htb[/htb]$ sudo nmap 10.129.2.28 -F -sU

	 AstraX01@htb[/htb]$ sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason

# Saving the Results

---

## Different Formats

While we run various scans, we should always save the results. We can use these later to examine the differences between the different scanning methods we have used. `Nmap` can save the results in 3 different formats.

- Normal output (`-oN`) with the `.nmap` file extension
- Grepable output (`-oG`) with the `.gnmap` file extension
- XML output (`-oX`) with the `.xml` file extension

## Style sheets

With the XML output, we can easily create HTML reports that are easy to read, even for non-technical people. This is later very useful for documentation, as it presents our results in a detailed and clear way. To convert the stored results from XML format to HTML, we can use the tool `xsltproc`.

	AstraX01@htb[/htb]$ xsltproc target.xml -o target.html


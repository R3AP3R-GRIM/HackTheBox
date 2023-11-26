---
{"dg-publish":true,"permalink":"/hack-the-box-dev-vortex-writeup/","noteIcon":"","created":"2023-11-26T09:53:47.438+05:30"}
---

Scanning:
![Pasted image 20231126095734.png](/img/user/Pasted%20image%2020231126095734.png)Port 80 (http) is open and port 22 (ssh) is open lets visit port 80 and see
It redirects us to http://devvortex.htb, Thus we need to edit /etc/hosts file
![Pasted image 20231126095650.png](/img/user/Pasted%20image%2020231126095650.png)
Let's do website enumeration using dirbuster
![Pasted image 20231126100322.png](/img/user/Pasted%20image%2020231126100322.png)
Didn't Find anything of interest, let's try searching the subdomains and see if we find anything
![Pasted image 20231126101246.png](/img/user/Pasted%20image%2020231126101246.png)
We find http://dev.devvortex.htb. Lets try to open it after editing /etc/hosts
Also, lets run dirbuster for http://dev.devvortex.htb and see what we get back.
Plus we find these two emails lets keep them for later.
	contact@devvortex.htb
	info@devvortex.htb
After the searching we hit http://dev.devvortex.htb/administrator/ which seems promising
![Pasted image 20231126102810.png](/img/user/Pasted%20image%2020231126102810.png)
After clicking the link for, `forgot your login details
We have 2 ways to recover the password.
	Method 1: configuration.php file
	Method 2: Direct Editing of Database
The login panel is running Joomla. After googling it found a website lets see https://hackertarget.com/attacking-enumerating-joomla/
We find some information about the same ![Pasted image 20231126104648.png](/img/user/Pasted%20image%2020231126104648.png)
The version is `4.2.6`
Which leads me to [Joomla! CVE-2023-23752 to Code Execution - Blog - VulnCheck](https://vulncheck.com/blog/joomla-for-rce)
Running the command according to the exploit : 
Visiting  http://dev.devvortex.htb/api/index.php/v1/config/application?public=true
we get the password and lewis as the user
Lets try logging in to the webpage
We have the credentials ![Pasted image 20231126105943.png](/img/user/Pasted%20image%2020231126105943.png)
for ssh lets find if we can get the password
Also, there is another user `logan@devvortex.htb `
We can upload at http://dev.devvortex.htb/administrator/index.php?option=com_media&path=local-images:/
Lets see if we can get reverse shell to work
![Pasted image 20231126110327.png](/img/user/Pasted%20image%2020231126110327.png)
It doesn't work out as i though after some more googling i find [p0dalirius/Joomla-webshell-plugin: A webshell plugin and interactive shell for pentesting a Joomla website. (github.com)](https://github.com/p0dalirius/Joomla-webshell-plugin)
Lets try it out , And yes it works http://dev.devvortex.htb/administrator/index.php?option=com_installer&view=install
Lets play around,
After downloading the git as .zip upload it and it got accepted
![Pasted image 20231126112250.png](/img/user/Pasted%20image%2020231126112250.png)
Lets now open it after opening manage extensions find **Webshell 
http://dev.devvortex.htb/modules/mod_webshell/mod_webshell.php?action=exec&cmd=
And for the command I used python3 reverse shell
And use this to stabilise the shell **python3 -c 'import pty; pty.spawn ("/bin/bash**")'
After that use mysql to get the passwords if possible from table `sd4fg_users;
![Pasted image 20231126140423.png](/img/user/Pasted%20image%2020231126140423.png)
and crack the logan paul one using hashcat.exe : hashcat.exe -m 3200 hash.txt rockyou.txt
![Pasted image 20231126135532.png](/img/user/Pasted%20image%2020231126135532.png)
After submitting user.txt;
For privilege escalation running `sudo -l
we get ![Pasted image 20231126135947.png](/img/user/Pasted%20image%2020231126135947.png)
lets see what it is
![Pasted image 20231126141152.png](/img/user/Pasted%20image%2020231126141152.png)
After googling it we find [fix: Do not run sensible-pager as root if using sudo/pkexec · canonical/apport@e5f78cc (github.com)](https://github.com/canonical/apport/commit/e5f78cc89f1f5888b6a56b785dddcb0364c48ecb) for **`CVE-2023-1326`**
According to it we have to create a crash file 
Lets see how to do that
Creating a bug and running it works 
![Pasted image 20231126141926.png](/img/user/Pasted%20image%2020231126141926.png)
																-Snipped-
![Pasted image 20231126142015.png](/img/user/Pasted%20image%2020231126142015.png)
And its done

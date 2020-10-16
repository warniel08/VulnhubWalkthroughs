# VulnOS 2

This is my first writeup for any box, CTF, that I've done. I'm determined not to have to consult the walkthrough, but who knows, I may still need it. 


> Your assignment is to pentest a company website, get root of the system and read the final flag.


I'm doing this box on VirtualBox. I have been trying to convert all VBox boxes to VMWare lately, but this one wasn't as easy of a conversion as others, so rather than spend more time trying to convert it, I decided to just do it in VBox.

First thing I had to do was figure out the IP address. I ran

```
$ nmap -sT -O 192.168.56.0/24 
Nmap scan report for 192.168.56.107 
Host is up (0.00078s latency). 
Not shown: 997 closed ports 
PORT     STATE SERVICE 
22/tcp   open  ssh 
80/tcp   open  http 
6667/tcp open  irc 
MAC Address: 08:00:27:7A:0C:8F (Oracle VirtualBox virtual NIC) 
Device type: general purpose 
Running: Linux 3.X|4.X 
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4 
OS details: Linux 3.2 - 4.9 
Network Distance: 1 hop 
```

I discovered that the IP for the vulnerable box is at 192.168.56.107 and that ports 22, 80, and 6667 are open. I've never done anything that dealt with port 6667 yet, but maybe I will now. It's hard to say yet if that's a rabbit hole, or actually helpful. I'm going to try to be a little more methodical on my approach for this box since I am doing a writeup.

Next I run:

```
$ nmap -sC -sV -oA nmap/vulnos 192.168.56.107
Nmap scan report for 192.168.56.107
Host is up (0.00024s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 f5:4d:c8:e7:8b:c1:b2:11:95:24:fd:0e:4c:3c:3b:3b (DSA)
|   2048 ff:19:33:7a:c1:ee:b5:d0:dc:66:51:da:f0:6e:fc:48 (RSA)
|   256 ae:d7:6f:cc:ed:4a:82:8b:e8:66:a5:11:7a:11:5f:86 (ECDSA)
|_  256 71:bc:6b:7b:56:02:a4:8e:ce:1c:8e:a6:1e:3a:37:94 (ED25519)
80/tcp   open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: VulnOSv2
6667/tcp open  irc     ngircd
MAC Address: 08:00:27:7A:0C:8F (Oracle VirtualBox virtual NIC)
Service Info: Host: irc.example.net; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

This tells me the same ports that are open but I also find out that ssh is an old version of 6.6.1p1. Ubuntu Linux is version 2.6? Really? This box is from 2016, I guess they used an old version of Ubuntu on purpose? I'm not super sure about that, but we'll see. It's running an older version of Apache, 2.4.7. There are some potentials here. Again, not sure what I'll do about port 6667. I think I'll look at it after I've gone through my usual web side scans.

I decide to use Firefox to navigate to the website at the IP address. I'm greeted with another description of the box and a link to the company website at ```/jabc```. I decide to navigate through the website to see if I can find anything interesting. I find when I view source that this site was written with Drupal 7? Never encountered that yet. I've heard of Drupal but never had to use it.

In the `JABS` tab is a list of products. I see that I can add products to a cart. After I click the `Add to cart` button the URL looks like: `192.168.56.107/jabc/?q=node/4`. I thought that could lead to something but after changing the trailing 4 to different numbers `-1 through 11`; I realize that some pages don't exist, and the ones that do just navigate through the different pages on the site.

On the `Documentation` page there wasn't anything there. I thought maybe something might be hidden. Out of habit I viewed the page source. In it I found:

```html
<p><span style="color:#000000">For security reasons, this section is hidden.</span></p>
<p><span style="color:#000000">For a detailed view and documentation of our products, please visit our documentation platform at /jabcd0cs/ on the server. Just login with guest/guest</span></p>
<p><span style="color:#000000">Thank you.</span></p>
```
Thank you for keeping that secure.

Let's try out those creds on this login page found at `/jabcd0cs/`. It appears to be run by OpenDocMan. I'm not familiar with that. According to the website `https://www.opendocman.com/` it's an Open Source Document Managment Software. It appears to be outdated too, since the copyright ends 2013. Maybe there's something there. I want to navigate around the portal and see what there is to see.

Using those great creds, I am indeed logged in as *guest*. Cool, there is a tab to add documents. I'm thinking that a web shell might be in play there. I think I'll test the uploader with a php file that prints out the phpinfo: 

`$ echo "<?php phpinfo(); ?>" > test.php`. 

I upload that file and it tells me that a .php file is not supported. I'll try adding .gif on the end of it like `test.php.gif`. 

**Fail!** It's smart enough to catch the .php in there. Fine I'll just add `GIF89a;` to the beginning of my little test php file...*that didn't work and I tried several different things*. Well, it uploaded my file just fine, but I couldn't view the file. It kept giving me an error. That error may be helpful to someone more experienced, but to me it just means that it didn't work. Maybe this is all just a rabbit hole.

Tried this command: `sqlmap --url "http://192.168.56.107/jabcd0cs/ajax_udf.php?q=1&add_value=odm_user*" --dbs` 
but it kept failing. This was an attempt at exploiting `OpenDocMan 1.2.7`.



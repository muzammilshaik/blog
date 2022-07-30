---
author: "Muzammil Shaik"
title: "forge"
description: ""
tags: ["ftp", "ssh"]
date: 2022-01-24
thumbnail: forge.png
---
{{< toc >}}

## Introduction
Today we will learn about the server-side request forgery attack. While enumerating, we discovered the FTP credentials through which we gain access to the server via ssh and the root using the Python library. With that stated, let's get started.</br>
As is customary, we will begin by scanning the server for open ports using the namp.
![forge](forge01.png)
We only have two open ports: SSH (OpenSSH) and apache (for the webpage).Let us now examine the webserver.</br> 
![forge image](forge02.png)
We have a standard website with an image upload option. We can upload images in two ways: one from a local computer and the other from a URL.
![forge image](forge03.png)
To receive the request from the server to our local system, I had setup the Python server from scratch. So we can be sure the server is sending us the request.
![forge image](forge04.png)

## Enumeration
While the webserver is operating, let's use the seclist wordlist to run the gobuster for directory listing.
![forge image](forge05.png)
However, we just have the uploads directory, which redirects /uploads to /upload, and I didn't find much in the directory, so I began the DNS enumeration using the wordlist present in seclist itself.
![forge image](forge06.png)
I discovered another subdomain admin, however it was only accessible via localhost.
However, we can send the request from the server to our localhost and then redirect the webserver to the admin page.
![forge image](forge09.png)
Let me describe the situation. We use the upload function to send the request to our localhost, from which we redirect the web server to the restricted admin panel. Let us try...
![forge image](forge08.png)
We received the request from the remote server, which was fulfilled by my scratch python web server, and we received success and an address to view our files.
![forge image](forge10.png)
Let's open this url in a browser, however I didn't get a response from the server when I tried to curl the same address, but I can see the content of the admin section. 
![forge image](forge12.png)

## Ftp accesss
I located the FTP credentials.
I am unable to access the FTP port since it has been filtered.
Considering that we have url redirection, let us redirect the web server to the FTP server to review the content.
![forge image](forge13.png)
As we can access the files, let us copy the user's private ssh keys in order to login via SSH, as the ssh port was open.
![forge image](forge14.png)
We have the ssh user's private key, which we store to a file and use to login to the server through ssh.
![forge image](forge15.png)

## Esclating Privileges
Let's look at the user's permissions. We can execute /opt/remote-manage.py. Because this script expected numbers as input, let's cause an error by pressing the random key, which takes us to the Python debugger ( PDB ). We can acquire root access via the PDB.
![forge image](forge16.png)

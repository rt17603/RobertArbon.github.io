---
layout: post
title:  "Easy access to remote computers"
subtitle: "How to automatically mount remote drive and log-in with one word and no password"
date:
categories: 
---

## This post will tell you how to:

* Automatically log in to your remote computing facilities using just one word. 

* Have your remote home directories appear in your local directory tree. 

This should work on a mac or linux machine. You will need to open up a terminal window, as usual `$` stands for 
whatever prompt you have on your terminal/command line program.  

I've taken and adapted bits from this [tutorial](http://www.linuxproblem.org/art_9.html). 

## Steps
1. Automatic authentication
  1. Check whether you have authentication keys on your local machine. 
  2. If you don't, generate keys and copy to the remote machine
2. Create log in alias
3. Automatically mount drives

### Automatic authentication
#### First check whether you have a `.ssh` directory in your home directory. 
Change directory (`cd`) to your home directory (`~`) then list all files (`ls`), including hidden files (`-a`). 
If you have a `.ssh` directory you will see something like this:
```
$ cd ~ 
$ ls -a
.	..	.ssh
```
Now change directory to `.ssh` and see what files you have. If you have authentication keys set up then you'll see this:
```
$ cd .ssh
$ ls
config      id_rsa      id_rsa.pub  known_hosts
```
If this is the case you can skip the next part. 

#### Generate keys
If you didn't find a `.ssh` directory or it was empty then you'll need to create it and the authentication keys. 
First change directory back home again (if you're not already there) and run the `ssh-keygen` command and tell the type of 
authentication (`-t`) you want is RSA. You will get a prompt
for a location to store the key and a passphrase. Just hit enter each time, i.e. don't enter anything. 
```
$ cd ~
$ ssh-keygen -t RSA
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/your_home_dir/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/your_home_dir/.ssh/id_rsa.
Your public key has been saved in  /Users/your_home_dir/.ssh/id_rsa.pub.
The key fingerprint is:
3e:4f:05:79:3a:9f:96:7c:3b:ad:e9:58:37:bc:37:e4
```

#### Copy keys 




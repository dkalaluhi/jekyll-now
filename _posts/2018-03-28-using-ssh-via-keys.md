---
layout: post
comments: true
title: "Using key-based authentication with ssh"
published: true
date: 2018-03-28
categories: [linux, ssh]
---

How to utilize key-based authentication with ssh. To improve system security, you can enforce key-based authentication by disabling the standard password authentication. Before doing that, make sure your keys are installed and working. We'll take a look at how to set up key-based authentication with [OpenSSH](https://www.openssh.com).


## Testing ssh
Before getting started, make sure you are able to ssh to system you would like to connect to with keys. Once you can do that, **AND LOG IN**, you can get started setting up your key pairs.

## Generating Key Pairs
Generating key pairs for SSH v2 is a fairly straight forward process. We are going to be doing all from the command line, but it's really not a hard process.

1. **Generate the key pair** - They are called key *pairs* because you need two(2) for this to work. A *public* key, which will go all systems you want to ssh *to* and a *private* key. The private key will sit on the system you are connecting *from*

{% highlight plaintext %}
...]$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/dave/.ssh/id_rsa):

{% endhighlight %}

The path in the ()'s is the default location. If you would want to put these some other place, you can enter that at the prompt. If the defaults work for you, and they should, just hit `enter`.

2. **Enter a passphrase (optional)** - This passphrase would be used **INSTEAD** of the *password* for your user account. If you do not enter a passphrase here, you can log in without needing to type anything. Pressing `Enter` twice here will allow you to not have a passphrase. 

After this is done, you should see something like the following:

{% highlight plaintext %}
Your identification has been saved in /home/dave/.ssh/id_rsa.
Your public key has been saved in /home/isesdyk/.ssh/id_rsa.pub.
The key fingerprint is:
bf:bb:73:58:b7:33:3a:f3:24:59:a7:3d:3a:03:88:18 dave@example.com
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|                 |
|                 |
|     E           |
|      o S .   . .|
|     . . o ..o.+ |
|          .o+.oo.|
|          o.+== .|
|          +=.*+o |
+-----------------+

{% endhighlight %}

3. **Change the permissions of the .ssh directory** - SSH is can be really picky about permissions, so you want to make sure that these are set correctly. These need to be the same on the local system, the one you are generating your key pair from, as well as the remote system, the one you are going to be ssh'ing to.

{% highlight plaintext %}
...]$ chmod 700 /home/dave/.ssh

{% endhighlight %}

4. **Copy your **_public_** key - This is important, only copy your **PUBLIC** key to the system(s) you want to ssh to. **DO NOT COPY YOUR PRIVATE KEY** There are many ways to do this. Probably the easiest is with `ssh-copy-id`. Just typing that command will give you a clue on how to do this:

{% highlight plaintext %}
...]$ ssh-copy-id
Usage: /usr/bin/ssh-copy-id [-i [identity_file]] [user@]machine

{% endhighlight %}

And if you check the man page, you would see that this command is used to install your id_rsa.pub into the authorized_keys for your user, on the remote systems. It will create `/home/dave/.ssh/authorized_keys` if it doesn't exist, and if it does, it will append your public key to the end of the file.

5. **Change the permissions of `.ssh/authorized_keys`** - Like we did before, we are going to chmod the `authorized_keys` file and set the proper permissions.

{% highlight plaintext %}
...]$ chmod 600 /home/dave/.ssh/authorized_keys

{% endhighlight %}

Once that is all complete, you should be able to ssh to the system where you installed your public key, and if you did not include a passphrase, you won't be prompted to enter it.

You can always contact me via [twitter](https://twitter.com/dkalaluhi) or in the comments below. Feel free to leave any constructive feedback, positive or negitive.
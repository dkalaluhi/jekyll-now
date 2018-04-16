---
layout: post
comments: true
title: "Ansible for Mere Mortals - Part II"
published: true
date: 2018-04-16
categories: [ansible, linux, automation, orchestration, Red Hat Accelerators]
series: "Ansible for mere mortals"
---

The Ansible inventory file, at it's most basic level is a grouping of machines or nodes that we want to control with Ansible. In this part, we are going to set up an inventory file, and run some ad hoc commands against those nodes.

## Ansible Inventory file

By default, your inventory file is located in `/etc/ansible/hosts`. You can either use this file, or create a new host file where ever you'd like. For me, I don't want to have a billion different host files, that I'll have to remember, so I want to use the default. If you look at that default inventory file, there is a bunch of comments that tell you how you can use it. I made a backup of that file, just in case, and built my inventory. Note: I am being careful to call this an inventory file, as the nomenclature of a host file already exists. But really, we could use both interchangably.

In smaller environments, you could just manually add hosts to groups manually, but what if you have 500 systems that you want to manage with Ansible? Surely you're not going to type those all in. That would suck. If you have a decent source of truth, like a CMDB of some kind, or operate in EC2, there is a dynamic inventory plug-in that'll get you what you want. For me, my source of truth is my Red Hat Satellite server, so I used the Satellite API to build my inventory file, but you can use hammer cli to the same effect, if you so choose. There is an article I wrote [here](https://dkalaluhi.github.io/satellite-6-api), on how I set this up. But for the impatient crew you can find the code available [here](https://github.com/dkalaluhi/satInventory). It's not the prettiest python you're ever going to see, but it works, and if I ever get time, I am going to head back to that project and optimize it for real world use.

Your inventory file should be a list of logically separated hosts. As you've seen in the code above, it grabs hosts from satellite, and moves them into either `[all]`, `[dev]` or `[prod]` groups. But you can put these into `[webservers]` or `[boston]` groups, what ever makes sense for you and your organization.

For our purposes here, we are going to use the default `/etc/ansible/hosts`, and in that file we are going to have three servers, and two groups. Fire up your favorite editor, vi right? And set it up in the following manner:

{% highlight plaintext %}
[webservers]
webnode1.example.com
webnode2.example.com

[database]
db1.example.com
db2.example.com
~
~
{% endhighlight %}

Now save that file `:wq` and we have our Ansible inventory file. Easy peasy! 

## Ansible ad hoc commands

Now that you've got your ansible inventory set up, let's try running some ad-hoc commands with `ansible`. Let's see if we can ping our webservers. In order to do that, we just need to use the `ping` module.

{% highlight shell %}
...]$ ansible -m ping webservers
webnode1.example.com | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ",
    "unreachable": true
}
webnode2.example.com | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ",
    "unreachable": true
}

{% endhighlight %}

Well, that's good! Oh, right! I didn't elevate myself. Now there are two ways to do this. If you have your user everwhere, with your public key deployed you can use --become --ask-become-pass. That command looks like this: `ansible -m ping weservers --become --ask-become-pass`. If you don't and you're using root keys everywhere, just toss a sudo in front and you'll be good as gold.

{% highlight shell %}
...]$ sudo ansible -m ping webservers
[sudo] password for dave:

webnode1.example.com | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
webnode2.example.com | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

{% endhighlight %}

Ok great! We can do the same thing with our database group, or we can do everyone with `all`. So that would be, `ansible -m ping all`. With ad-hoc commands, we can do a ton of stuff! We could copy files, manage packages, and manipulate users. You can read more about ad-hoc commands with ansible <a href="http://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html">here</a>. There are so many possibilities, and that is just the tip of the iceberg!

## Let's review

By this point we have: Enabled our repos to install the Ansible binaries, installed our control node and, if you were really on top of things, set up ssh keys on our managed nodes. We've created our inventory file in a way that allows us to manage `all` of our hosts or a subset of hosts like, `[webservers]`. In our next post, we'll start talking about Ansible Playbooks.

Ask me about becoming a <a href="https://access.redhat.com/accelerators" target="_blank">Red Hat Accelerator!</a>

<a href="https://access.redhat.com/accelerators" target="_blank"><img src="/images/image1.png" width="109" height="122" /></a>

{% include series.html %}
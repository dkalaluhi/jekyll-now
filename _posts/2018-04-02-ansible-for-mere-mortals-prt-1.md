---
layout: post
comments: true
title: "Ansible for Mere Mortals - Part I"
published: true
date: 2018-04-02
categories: [ansible, linux, automation, orchestration, Red Hat Accelerators]
series: "Ansible for mere mortals"
---

How to use Ansible to take back your time, control your configurations, and make a repeatable process to deploy 10 servers, or 100,000 servers.


## The Setup

I recently got an email from my boss one sunny but cold, Sunday afternoon: One of our servers wasn't configured to correctly rotate log files, and was in the process of filling up the /var filesystem. He went through and manually deleted them. Crisis overted! But wait, there's more. Why isn't logrotate everywhere, already? Why do the configuration files differ across our systems? Why did we not realize this sooner? Why, for the love of God, is our "standard configuration" not standard?


While those are [mostly] all pretty good questions, those aren't the ones to be asking. Better questions would be; How do I make sure that logrotate is everywhere, throughout my environment? How can we ensure that the configuration files are everywhere and standard? And finally, how can we make sure, that new systems we provision, look exactly the same? Enter, Ansible.


Now before you say anything, I intentionally left off the last two questions. Those aren't the important ones. I don't really care why we didn't realize this sooner. All of us kind of knew this already. And who cares why it's not standard? That line of questioning will just turn into some nasty finger pointing, and won't help us fix the real issues!


## Ansible to the rescue

Ok, so in a nutshell, yes, we will be using Ansible. But what is this thing called "Ansible"? Quite simply, Ansible is:

> App deployment, configuration management and orchestration - all from one system. Ansible is powerful automation that you can learn quickly. 

You could dig a little deeper than that quote and add; this is all done over ssh with no agent to install. 


What is really needed here, for all of this magic to work, is; The Ansible binary, a good inventory file so Ansible knows what hosts to act on and one or many Ansible Playbooks that will do the "stuff" you need done.


## Installing Ansible Control Node

This is probably the easiest part of all of this. From a RHEL7 system, you need make sure the _server-extras_ repository is enabled, then just install ansible with yum.

<div class="panel panel-warning">
    <div class="panel-heading"> Warning </div>
    <div class="panel-body">The repository, server-extras is being deprecated in a future relase of RHEL7. Install Ansible from either EPEL or from the Ansible repository</div>

</div>

### Control Node requirements
Python2 - Either 2.6 or 2.7
or
Python3 - 3.5+

Depending on what you're used to running, what's required to run in your environment, or what you are trying to learn, this is completely up to you. I prefer python3, but that's also what I'm trying to learn. 

<div class="panel panel-info">
    <div class="panel-heading"> Note </div>
    <div class="panel-body"> Ansible 2.2 introduces a tech preview of support for Python3. For more information, see [Python3 Support](http://docs.ansible.com/ansible/latest/python_3_support.html)</div>
</div>


### RHEL7
{% highlight shell %}
$ subscription-manager repos --enable rhel-7-server-extras-rpms
$ yum install -y ansible

{% endhighlight %}

### RHEL6 based system
The installs for RHEL6 systems are still located in EPEL since extras is not part of version 6.

<div class="panel panel-info">
	<div class="panel-heading"> Note </div>
	<div class="panel-body">For fresh installs, I would recommend just starting with RHEL7. This should result in less re-work for you when you have to make the change from RHEL6 to RHEL7.</div>
</div>


## Configuring Managed Nodes
Ansible manages nodes via ssh, as such, there is nothing to install on your managed nodes to make this work.

### Managed Node Requirements
Python2 - 2.6+
sftp - If this is not available in your environment, you can switch to scp in `ansible.cfg`

### Setting up ssh
One thing you're going to want to do at this point, is set up host keys across your environment so ansible can connect and run the tasks that it'll need to run. If you have root keys everywhere, that's an option, though I don't recommend doing it this way. Instead, make sure your user exists across the entire environment, install your public ssh key, and use the ansible built-in become: yes, to ssh (via keys) and sudo to run the tasks. In fact, if you currently have root keys set up, this would be a great way to start phasing that out. Once that's done we can go ahead and start on the other items of the list. If you need help setting up keys, check out [this post](https://dkalaluhi.github.io/using-ssh-via-keys/) to get started.

<div class="panel panel-info">
	<div class="panel-heading"> Note </div>
	<div class="panel-body">In order to use `become` make sure our admins are in the wheel group.</div>
</div>

<div class="panel panel-warning">
	<div class="panel-heading"> Warning </div>
	<div class="panel-body">`become:` is superseding the old method `sudo:`. `sudo:` will still work, for now, but this is a fresh install, so just use `become: yes`.</div>
</div>

This will have to be done all of the managed nodes in your environment. If there are already root keys deployed, a simple one liner to get that file out there, will work. Now is the time to do it, by the end of this, we'll be removing all of the root keys from our managed systems.


## Let's review
So what we've done so far is; Installed Ansible on our control node and set up ssh on our managed nodes for key-based authentication. In part 2 of *Ansible for Mere Mortals*, we'll create our inventory file and run some ad-hoc commands with ansible to make sure everything is working correctly.

As always, feel free to contact me via [twitter](https://twitter.com/dkalaluhi) or in the comments below. Please leave any constructive feedback, positive or negative.

Ask me about becoming a <a href="https://access.redhat.com/accelerators" target="_blank">Red Hat Accelerator!</a>

<a href="https://access.redhat.com/accelerators" target="_blank"><img src="/images/image1.png" width="109" height="122" /></a>

{% include series.html %}
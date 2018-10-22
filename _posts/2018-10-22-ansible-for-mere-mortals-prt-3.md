---
layout: post
comments: true
title: "Ansible for Mere Mortals - Part III"
published: true
date: 2018-10-22
categories: [ansible, linux, automation, orchestration, Red Hat Accelerators]
series: "Ansible for mere mortals"
---

And...we're back! It's been a while since I've had time to sit down and write; summer was a blur, work has been kicking my behind, designing our infrastructure for the new data center we're building and to top all of that off, my wife and I are expecting our fourth! So I've been busy painting, building furniture going to doctor appointments and all that jazz. It's been a busy summer to say the least!

If this is your first time here, scroll down to the bottom of this article and start with Part I of the Ansible for mere mortals series. But now, without further ado, it's time for our next installment of Ansible for mere mortals!

## Ansible Playbooks
So far, we've been able to get Ansible engine set up, built an inventory file, and ran some ad-hoc commands through Ansible. Ad-hoc commands are great and all, but the real power of Ansible is...

## Terms and Structure
Ok, first things first. Playbooks are wonderful thing once you understand them. I think before we get into the power of the plays, it's probably prudent to talk about the terms you'll see when discussing playbooks and the over all structure of a playbook. I hope this will make a little bit more sense to you, when planing your plays and playbooks. They really aren't that hard, but I had a hard time not putting the cart before the proverbial horse when I was starting out with this.

**Play** - As the <a href="https://docs.ansible.com">Ansible Documentation</a> states - *"The goal of a play is to map a group of hosts to some well defined roles, represented by things ansible calls tasks."* Which in my opinion is probably one the worst definitions I've read, so let me take a quick stab at defining a *play*: A Play is one or more tasks that run against a system or a group of systems.

**Task** - This one is pretty easy: A task is really nothing more than a call to an Ansible module. *eg. ansible -m ping*. *ping* would be the Ansible module here.

## Ansible Playbooks (cont'd)
...but the real power of Ansible is, the playbook!!! With some basic terminology out of the way, we can now take a look at an Ansible playbook.

 Again, from the <a href="https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html">Ansible Documentation</a>, here is a simple playbook that contains a single play:

```yml
---
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    yum:
      name: httpd
      state: latest
  - name: write the apache config file
    template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf
    notify:
    - restart apache
  - name: ensure apache is running
    service:
      name: httpd
      state: started
  handlers:
    - name: restart apache
      service:
        name: httpd
        state: restarted
```

As you can see it's a single play, against a single group of servers, *webservers*. This particular playbook, runs three(3) tasks. First, we make sure that apache is at the latest version, and if not, get it there with a call to the `yum` module. Second, we are going to write the apache configuration file with the `template` module. Finally, we'll make sure that httpd is running with a call to the `service` module.

There are some other things in here, like `notify` and `handlers` which we will talk about later, but just know, that after the template is written, that task notifies the handler *restart apache* to, you guessed it, restart it!

## Let's review

You should now have enough information to write a simple playbook that can install/update/upgrade/remove a package utilizing the `yum` module. You can write/modify any configuration files you'll need with the `template` module. And finally, you can stop/start/restart those services with the `service` modules. Next time, we'll talk more playbooks, a little bit about version control and explore some of the more advanced things you can do with a dynamic playbooks.

Ask me about becoming a <a href="https://access.redhat.com/accelerators" target="_blank">Red Hat Accelerator!</a>

<a href="https://access.redhat.com/accelerators" target="_blank"><img src="/images/image1.png" width="109" height="122" /></a>

{% include series.html %}
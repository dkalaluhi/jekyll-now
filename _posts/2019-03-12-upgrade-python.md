---
layout: post
comments: true
title: "Upgrading Python in a Virtual Environment"
published: true
date: 2018-03-12
categories: [python, virtualenv, ops, devops, sre]
---

# Help my app has a bug

We've all been there before, you get a call or an email, or even a driveby at your desk from one of your developers. They've identified an issue in a version of something and they need you to fix it. That statement is typically followed up pretty closely with, JimJoe was "the guy" who made this app and when he left the company, he left no documentation behind and I just sort of inherited these systems. It's the stuff of nightmares, defunct developer, old application, new person supporting the application and no documentation! In spite of all of that, we as ops people try to help out the best we can, so we start digging in.

## Problem Statement

I have identified an issue with our application and we appear to be hitting **Bug #XXYYZZ** in python. I've read the BZ and there's a patch to fix this. Can you apply this patch? The chills that go down your spine make you shudder, and from the corner of your eye a small tear forms at the realization that whoever did this, might have done it the wrong way and actually built their _virtualenv_ using the system's own python.

## The discovery

You start taking small bites out of the application and begin to try to understand what a some developer did long ago. You start with the basics. _python --version_ and it returns the exact version that the current developer is telling you. _2.7_ That doesn't seem right, This is an Enterprise Linux 6.10 system, I should be looking at python 2.6! _whoami_ **root**. Shit, did they really change the system version of python? How the hell is this thing even working? You run to get another cup of coffee, walk around the office in a daze. At some point on your jaunt, you come to a realization: Wait! I think... You run back to your terminal, _which python_ _/usr/local/bin/python_. You breath a sigh of relief, ok, they didn't touch the system.

## The discovery part 2

Before you can fix their issue, you need to understand the basics of how the app works. In my case, it was Django application, running in a virtualenv, tied to a compiled version of Python, 2.6.9 that was in /usr/local/bin. With mod_wsgi that was also compiled for the application. I don't know why they just didn't pip install mod_wsgi, but there I was. I had most of the information I needed in order to get python up to snuff.

## The fix

The fix was actually pretty simple. The problem we have in our group, is we are ops people. We live in infrastructure. If you tell us that your application is down, we ssh to the box and inform you that it's up and you need to check the app. If we are feeling particularly frisky, we might even check the previous day's changes just to verify we didn't do something to cause the outage. And LORD, do not tell me "My system feels slow"! I digress.

So we are ops people, not developers. So we do what any self respecting ops person does...we google. In my case, I was looking for a way to upgrade the version of python that the virtualenv uses. If you google this, you will see a bunch of posts talking about _pip freeze > requirements.txt_ removing the virtual environments, recreating the environments, pip installing the requirements, etc. It's actually MUCH easier than this.

I compiled a new version of Python, in this case 2.7.16 with the altinstall on the make. So it goes like this:
_./configure_
_make_
_make altinstall_

Once I was done with that, I deactivated the virtual environment and ran _mkvirtualenv --python=/usr/local/bin/python2.7 /path/to/virtenv_ recycled httpd and BAM! **It was still broken!**

Ok some random 500. Check the logs...in the logs I saw that mod_wsgi was configured to use the old version of python. (Learn something new everyday). So I pip install mod_wsgi saw some warning at the top of the scroll fly by, but warnings aren't looked at as fatal, so once it was done, I recycled the app and BAM! **STILL NOT WORKING** In this case the warning was clear, the new version of python wasn't configured with the _--enable-shared_ directive, so I did it again.

_./configure --enable-shared_
_make_
_make altinstall_

Created the virtual environment again, and BAM! **IT WORKS!** Bug is fixed, and Life is going on!
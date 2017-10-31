---
layout: post
comments: true
title: "How to Establish a Session and Authenticate with the vSphere 6.5 API"
published: true
date: 2017-10-12 14:11:00
categories: projects tech
tags: python rest vcsa
---
There have been plenty of people to post how to connect to the vcsa's restful api: Typically around powershell and powercli. One of my favorites is from [@chriswahl](https://twitter.com/chriswahl) and is available at [Wahl Network](https://wahlnetwork.com/2017/02/24/vsphere-6-5-restful-api/). If you're a powershell person, please check that out! If you've read my [first post](https://dkalaluhi.github.io/projects/jekyll/2017/09/30/first-post.html), you already know that what I am doing here is learning. So keep that in mind while looking at [my code](https://github.com/dkalaluhi/restDemo). You should also know that this works explicitly with python-3.6, due to the use of f-strings.

## Creating a Session

So the first step is to set-up a basic auth key-value pair. As [@chriswahl](https://twitter.com/chriswahl) points out:
> This is fairly universal across everyone's RESTful API.

In my example below, we are gathering some user input for credentials. Namely, we are using `input()` and `getpass()`. We also bring in your vcsa fqdn, to build the URI.

{% highlight python %}
import requests
import getpass
from requests.auth import HTTPBasicAuth
import json

fqdn=input('Enter FQDN of your vcsa: ')
uri = f'https://{fqdn}/rest/com/vmware/cis/session'
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}

requests.packages.urllib3.disable_warnings()

r = requests.post(uri, auth=HTTPBasicAuth(input('username: '), getpass.getpass('password: ')), headers=headers, verify=False)
{% endhighlight %}

In the above code, what we've done is; Brought in the name of your vcsa, Built the URI, using `f-Strings`, built some generic headers to send with our request, supressed warnings (like SSL cert warnings), and sent the POST request to the vcsa server. At this point, if we take a look at `r`, we'll see that we got back `<Response [200]>` which is what we are looking for. From this point on, we can exchange the token for user/password credentials.


{% highlight python %}
response = r.json()
token = response['value']
headers['vmware-api-session-id'] = token
{% endhighlight %}

What we are doing here, is taking the response that we got back, and pulling the token out. This bit is fairly self explanatory, but we are pulling the json out of the `r` and assigning it to response. From there we pull the token out of the json, and inject that into the headers we built above. There are two points to make here.

1. In most cases, we would most likely be using 'bearer', but [@VMware](https://twitter.com/vmware) decided to use that fairly clunky 'vmware-api-session-id'.
2. If that header bit `headers['vmware-api-session-id'] = token` looks odd, it's because our headers, and our json response in general, are [dictionaries](https://docs.python.org/3/tutorial/datastructures.html) in python.

## Calling the API

Now that we have our token and added it to our `headers`, we can make calls to other endpoints.

# Get VM Inventory Summary

This is a simple call using the token we got above from the ..com/vmware/cis/session endpoint. It just pulls back the inventory for vcenter.


{% highlight python %}
r1 = requests.get(f'https://{fqdn}/rest/vcenter/vm', headers=headers, verify=False)
x1 = r1.json()
print(json.dumps(x1, indent=4, sort_keys=True))
{% endhighlight %}


Now we can make requests using our `token`, that we pass in the python dictionary, `headers`, pulling the json out with that `x1` variable, and then printing to our screen, in a way that we can actually read. and it looks something like this:


{% highlight json %}
{
    "value": [
        {
            "cpu_count": 2,
            "memory_size_MiB": 3000,
            "name": "darthVader",
            "power_state": "POWERED_ON",
            "vm": "vm-666"
        },
        {
            "cpu_count": 4,
            "memory_size_MiB": 16384,
            "name": "skywalker",
            "power_state": "POWERED_ON",
            "vm": "vm-42"
        },
{% endhighlight %}

From this we can do a multitude of other things with the information we got back, based on other endpoints if we wanted to. And that's all there is to it!

## What I learned

As I have said before: This is an exercise in learning. I'm always trying to work to get better. Know your craft. As I progress, I hope that everything I write about, has this section!

# From the code

So, I'm fairly new to python. In the past I've done some pretty simple stuff, and this by no means, is hard! But it was a fantastic exercise for me, to be able to connect to a RESTful API, build a session and parse some information. I learned the basics if `requests`, and got a better understanding of dictionaries and json, in the process.
---
layout: post
comments: true
title: "Building a Dynamic Inventory with Satellite 6 and Python"
published: true
date: 2018-03-21
categories: [api, python, satellite, ansible, inventory files]
---

I needed a way to build a true inventory file for [Ansible](https://www.ansible.com) to use. We ditched our old CMDB system in favor of a new application, but it has the same drawbacks. For one, the hosts are entered manually. I don't really trust myself to remember to enter information into another application let alone expect the other 4 guys on the team, to use a system so underwhelmingly un-utilized. So I needed a better source of truth. What I do have, is a Red Hat [Satellite 6](https://www.redhat.com/en/technologies/management/satellite) server. Because we use Satellite to patch and maintain all of our systems, and because registering to Satellite is a requirement, I know this is probably the best source of truth we have. But how can I turn this system into an Ansible inventory file? And how can I make this as dynamic as possible with out Red Hat Ansible [tower](https://www.ansible.com/products/tower)? Easy, I can turn to the Satellite 6 API.



## API Examples
Running through the [python example](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.2/html/api_guide/sect-api_guide-running_queries_using_python) to query the Red Hat Satellite API, I noticed, among other things, that basic auth was being used in the form of username = blah password = plaintext. Wow, so that's not good! In order to get this working, I was fine with getting this set up as is, I decided to just use base64 encoding to get a little tighter security around it, but this is in no way a decent solution. There should be a way to exchange credentials for at the very least a token, then I can do something similar with how I connect to my vCenter APIs. You can check that out [here](https://dkalaluhi.github.io/ConnectingToVcsaApi/), if you want. Or just the code out on [github](https://github.com/dkalaluhi/restDemo).


With no swagger for the Satellite API, (short sighted), there's really no way to do it, without actually doing it, so I set out to write the code that would get me what I needed. I'm not the best coder by any stretch of the imagination, but all in all, this excercise took at most an hour and half to get the results that I needed in order to build a half decent inventory file.


## The Code
[The code](https://github.com/dkalaluhi/satInventory) is ugly. It was a quick answer to old problem we were having at work. It should be refined, it should be made better. I don't have time for this! But let's take a quick look and I'll explain what's going on in each part. Some of this code, I am responsible for, some of this code, came directly out of the python example above.


### Creating a 'session'
In the code below, there's nothing for the user to input, like in the [REST Demo](https://github.com/dkalaluhi/restDemo) above, so these values are going to need to be set prior to running the script. But this process shouldn't look crazy to you, if you've ever written code to connect to an API. If you haven't, I've written an article [here](https://dkalaluhi.github.io/ConnectingToVcsaApi/), pretty explicit for vCenter, but it walks you through creating a session. As always, [@chriswahl](https://twitter.com/chriswahl) has some great stuff on this, [here](https://wahlnetwork.com/2017/02/24/vsphere-6-5-restful-api/) If all of that is TLDR; Let's walk through what I'm doing in this instance.

{% highlight python %}
import json
import sys

try:
    import requests
except ImportError:
    print "Please install the python-requests module."
    sys.exit(-1)

sat_api = 'https://satellite.example.com/api/v2/'

headers = {'Content-Type': 'application/json', 'Accept': 'application/json', 'Authorization': 'Basic *****************'}

requests.packages.urllib3.disable_warnings()

{% endhighlight %}

Ok, like I said before, pretty straight forward, but... What we are doing is building the foundation of our URI with the `sat_api` variable, bulding `headers` to send with our requests and silencing those pesky SSL certificate warnings with that `requests.packages.urllib3.disable_warnings()`


### The functions
The first 3 functions are from the original examples. I will cover these briefly now.

{% highlight python %}
def get_json(url):
    r = requests.get(url, headers=headers, verify=False)
    return r.json()

def get_results(url):
    jsn = get_json(url)
    if jsn.get('error'):
        print "Error: " + jsn['error']['message']
    else:
        if jsn.get('results'):
            return jsn['results']
        elif 'results' not in jsn:
            return jsn
        else:
            print "No results found"
    return None

def display_info_for_hosts(url):
    hosts = get_results(url)
    if hosts:
        for host in hosts:
            print "ID: %-10d Name: %-30s IP: %-20s OS: %-30s" % (host['id'], host['name'], host['ip'], host['operatingsystem_name'])


{% endhighlight %}

The first function `get_json(url)`, builds a request and returns the json of that request. This is pretty standard when it comes to consuming APIs.

The next function `get_results(url)` actually calls `get_json()`, stores `r.json()` that was returned, into the variable `jsn`. Has some error handling in place, then makes sure `'results'` are in the the json, and if not, it'll inform the user that no results were found.

The final O.G. function here, is the `display_info_for_hosts(url)`, which builds on the first 2 functions and prits out some things about the host. As you will see below, in the code that we are looking at today, we don't actually use this one, but I kept it around, as this was the basis for my functions.

The functions that we are going to concern ourselves with, are the below that. Namely, `write_info_for_all_hosts()`, `write_info_for_dev_hosts()` and `write_info_for_prod_hosts()`

As I said before, there's a lot of tweaks that need to made to this code. For one, it needs to be cleaned up, and for another, those three(3) functions really only have to be one function, one file, one open etc. But this was quick and dirty to fix an issue. I wasnt' going for style points, I was going for working. Style points can come later.

{% highlight python %}
def write_info_for_all_hosts(url):
    hosts = get_results(url)
    f = open("/etc/ansible/hosts", "w+")
    f.write("[all]\n")
    if hosts:
        for host in hosts:
            f.write(host['name'])
            f.write("\n")
f.close()
{% endhighlight %}

Each one of these functions do pretty much the same thing. The biggest difference here, is one opens the hostfile and truncates that file to 0 bytes, and the other two append. Above is the code for `write_info_for_all_hosts(url):`. So this is where our truncation happens.

First thing is first here, we need to get some results from some url, actually, it's a URI, but what ever. So we need to get these results, and in this case, the results for `all` hosts in Satellite. We are going to do this by calling `get_results(url)` and assigning the return to the variable `hosts` The next thing we'll need to do, is to open the file we want and assign that to a variable, we do that with the very next line, `f = open("/etc/ansible/hosts", "w+")` , remember that w+ is going to truncate the file for us. And the very next thing we do, is write our static group `[all]` to our file with `f.write()` Later we can use that group to run tasks out of Ansible. After that's done, for each host we have, we write the name followed by a newline. Finally, because I often times know what I _should_ be doing, we close our file with `f.close()`


### Putting it all together
Ok so our functions are all completed, the last thing we need to do is to call our function with the right URI. We do this with our `main()` function. Technically, you don't need a main function, but I guess it's good practice, based on what people who know more about this than I, tell me.

So now we have the lines:

{% highlight python %}
    write_info_for_all_hosts(sat_api + 'hosts?utf8=%E2%9C%93&per_page=10000&search=name++%21%7E+virt-who')
    write_info_for_dev_hosts(sat_api + 'hosts?utf8=%E2%9C%93&per_page=10000&search=hostgroup+%7E+Dev+and+name+%21%7E+virt-who')
    write_info_for_prod_hosts(sat_api + 'hosts?utf8=%E2%9C%93&per_page=10000&search=hostgroup+%7E+prod+and+name+%21%7E+virt-who')
{% endhighlight %}

While the above lines may look a little more cryptic, they are pretty simple. We are calling our three(3) functions from above with the `sat_api` variable plus the string `'hosts?'` and some funky look stuff, namely, that `utf8=...` I actually got these addresses from searching within Satellite in order to build this out. It's important to note here, that the search strings we are looking for, are specific to our environment. We are searching on hostgroups, namely, the Dev and Prod hostgroups. We are also filtering out our virt-who hosts, which are there for the vHosts in our environment. Now, we can skip over the utf8 garbage and focus on the important stuff:

`&per_page=10000` - by default, Satellite 6 only displays 20 records by default. I attempted to disable this pagation, but the instructions I found didn't work. I know I have less than 10,000 records, so I just set the per_page. This is one of those things I have to go back and fix.

Next, is the `&search=name++%21%7E+virt-who`. All that says, is give me all systems that don't have a name like (!~) virt-who.

In the next two function calls, there is one more search query, `&search=hostgroup+%7E+Dev+and+name....`. Well we know from above that the `%7E` is literally a tilde, `~`. So you can read that line like, search=hostgroup LIKE(~)Dev and name NOT LIKE(!~)virt-who. The same can be said of prod hosts line, just replacing Dev with Prod.

So, that is the long winded way to explain how I get dynamic-ish inventory files to run Ansible playbooks and ad-hoc tasks against. Let me know if you are doing something similar, or better in the comments below.

Ask me about becoming a <a href="https://access.redhat.com/accelerators" target="_blank">Red Hat Accelerator!</a>

<a href="https://access.redhat.com/accelerators" target="_blank"><img src="/images/image1.png" width="109" height="122" /></a>
---
layout: post
title:  "Using Cygwin sshd on a Windows Domain"
date:   2016-04-29 16:12:54 +0000
author: wes
---
Installing cygwin sshd on a normal machine is pretty straightforward. You simply run `ssh-host-config` and walk through the prompts. On a Windows domain it's more complicated. Your sshd will still run and you can still connect and run normal commands, but as far as I can tell there's no way to elevate your permissions and do admin actions (that would trigger a UAC prompt if you were using the Windows GUI). To fix this we'll need to run sshd as a domain user that has the permissions to `setuid` like it would on a unix. For some details check [this post by Cygwin project lead Corinna Vinschen][cygwin-ml].

Make a domain account to run the service
---

I won't cover creating a domain user, as that's pretty simple and steps for that can be easily conjured with a web search. We're going to call our domain user `cyg_server`, but you can call it whatever you want. That user is going to need the permissions to basically `setuid` (or whatever the Windows equivalent is called). Create a group policy object and link it to your org unit with the following policies:

Under Computer Configuration / Policies / Windows Settings / Security / Local Policies / User Rights Assignment add your user to the following policy settings

* Act as part of the operating system
* Create a token object
* Deny log on locally
* Deny log on through Remote Desktop
* Replace a process level token

Once the policy is distributed you can install sshd or replace your existing local sshd.

Remove the existing sshd service and user
---

If you have an existing machine-local sshd running, get rid of it (you can use the Windows service tools (sc) as well).

{% highlight bash %}
cygrunsrv --stop sshd
cygrunsrv --remove sshd
{% endhighlight %}

Then delete the local `cyg_server` user. It'll only serve to confuse you otherwise.

Update cygwin's view of the domain users
---

From a cygwin terminal with admin rights, run these to update cygwin's list of available domain users and groups. The ssh-host-config script will use these to decide if your chosen domain `cyg_server` user has the proper permissions to run the ssh daemon.

{% highlight bash %}
# these will obviously clobber your files, so if you want to preserve
# some users/groups you've made outside Windows you'll need to take steps
# to protect those changes

mkpasswd -l -d > /etc/passwd
mkgroup -l -d > /etc/group
{% endhighlight %}

You should be able to see your user in the list of users output by mkpasswd

{% highlight bash %}
$ mkpasswd -l -d | grep cyg_server # or your chosen username
cyg_server:*:1049696:1049089:U-DMZ\cyg_server,S-1-5-21-10613669-1566996133-3585012528-1120:/home/cyg_server:/bin/bash
{% endhighlight %}

You can see there that cygwin has mapped a cygwin user `cyg_server` to the domain user `DMZ\cyg_server`. In cygwin, we'll use just `cyg_server` to refer to that user. (This is why it's important to delete the existing `cyg_server` user if you already had a local sshd set up).

Run ssh-host-config
---

sshd has a handy setup script that will walk you through the process. The important options we want are:

* strict modes -> yes
* privilege separation -> yes
* value of CYGWIN for the daemon - ntsec
* user to run as -> cyg_server (or whichever one maps to your domain user in /etc/passwd)

Test it out
---

If you've done everything right, you should be able to connect over ssh with your domain credentials. Connect with the cygwin user name that maps to your domain account, e.g. `wes`, for `LAN\wes`. Once connected try to sc stop and sc start a service. It should work, rather than getting a permission failure message, like you would have before.

[cygwin-ml]: https://cygwin.com/ml/cygwin/2010-01/msg00334.html

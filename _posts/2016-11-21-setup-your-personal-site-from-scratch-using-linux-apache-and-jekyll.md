---
layout: post
title:  "Setup Your Personal Site From Scratch Using Linux, Apache And Jekyll"
date:   2016-11-21 06:04:32
categories: vps jekyll apache fedora
comments: true
author: Koushtav Chakrabarty
---

Like I promised in my
[last post](https://fleptic.eu/fleptic/update/vps/fedora/jekyll/2016/11/13/migrating-fleptic-to-a-vps.html)
, this is going to be a comprehensive tutorial to get your site up and running on a VPS. It is assumed
that you already have root access to a VPS running Linux ready and also have registered `www.example.com`.
If you haven't and want to, check out [DigitalOcean](https://www.digitalocean.com) for cheap a VPS and
[NameCheap](https://www.namecheap.com) for a domain.

And...

...to go lashing out our linux-fu, please make sure you know the basics of some command line plaintext
editor such as `vi`, `emacs`, `nano`, etc :smile:

<br>

### Introduction to Some Basic Terminologies

_(NOTE : Advanced users may skip this section)_

Before we jump into the terminal and get our hands dirty, I'd like to familiarize you with the basics of
how a website works. Let's assume that your domain is `www.example.com`. Now, when someone types this in their
favourite web browser and hits go, what happens? What happens can be broken down as follows:
* Your **browser** requests the landing page of `www.example.com` to the **host** of `www.example.com`
  by translating the domain name using a **DNS** (domain name system).
* The remote server receives the request, does some processing, and sends the requested page back to you.
* The requested page arrives at your device and the browser displays the page as per the HTML code of the
  page.

Our interest is in step two of this process, i.e., listening for incoming requests by clients for webpages,
processing their requests and then serving them the requested page(s). The program that does this is called
a **webserver**. A webserver uses the **HTTP/HTTPS** protocol in the application layer to do this. There
exist a variety of webservers around - Apache, Nginx, IIS, GWS., etc.

The webpages that the webserver serves may be either generated **statically** or **dynamically**. The difference
between the two is that in case of dynamic webpages, they have to be rebuilt everytime they're requested
whereas in case of static webpages the pages are generated just once and are served without rebuilding. We
are going to build a static website because it's elegant, faster and efficient. Thus, we'll be also needing a
static site generator too.

<br>

### Which Tools Will We Use?

We are going to use a VPS running Linux, [Apache](https://www.apache.org) for the webserver and
[Jekyll](https://jekyllrb.com) as the static site generator. The popular choices of Linux distros for servers are
Fedora, Ubuntu, Debian, CentOS, OpenSUSE, RHEL, etc. The choice doesn't really matter _much_ since Apache and
Jekyll are available for all popular distros. I'd advise you pick either Ubuntu or Fedora.

I'll be demonstrating all commands for Fedora. Exact usage may vary for other distros.

How will our site work, you ask? You see, we'll setup Apache and configure it properly to receive incoming
HTTP requests on port 80 of the server. We'll also add a repository to which we can push our site's code
using [git](https://git-scm.com) version control. By using a post receive hook that automatically builds
the code using Jekyll and copies it to the document root of Apache, we take your site live within seconds of
pushing to the repository! Back on our local machine, we'll create a simple Jekyll site that will be pushed
into the VPS's repo which was setup previously.

But before doing any of this, it's important that you make `www.example.com` point correctly to your
VPS's IP address and configure it properly. If you already know how this is done, skip the next section.
If not, then fret not and read on! :wink:

<br>

### Making www.example.com Point to your VPS

_(NOTE : Advanced users may skip this section)_

To do this first go to your domain name registrar's site and access the control panel for managing your
domain. Make it point to the nameservers of your VPS provider and then create the A records.

The nameservers for your VPS can be found on your VPS access control panel. They typically look like
`nsx.myvpsprovider.com`. Make your domain name point to the nameservers from the domain control panel
and create A records for `www` and `www.examle.com`.

Now that this is done, it should take some time (from a few minutes to upto 24 hours) until all cached
copies of DNS records are updated to reflect this change.

<br>

### Configuring Your VPS to Run Apache

Apache is the most widely used webserver that's out there. Hence it's available in most Linux
distros' package repositories. Install Apache:

```
# dnf install httpd
```

Before running and testing the webserver, we need to make sure that HTTP requests on port 80
are allowed to pass through the firewall. There's a wonderful tool called `iptables` that is made for
managing the firewall and traffic through it.

Go ahead and install the `iptables-services` package.

```
# dnf install iptables-services
```

To enable the `iptables` service to start automatically on boot and also to start it, do:

```
# systemctl enable iptables
# systemctl start iptables
```

Next, save the current runtime rules of `iptables` (to `etc/sysconfig`) by doing:

```
# /usr/libexec/iptables/iptables.init save
```

One last thing remains is to allow HTTP traffic so that requests can reach our webserver:

```
# vi /etc/sysconfig/iptables
```

In the rules file, just add this line:

```
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
```

After all this is done, the VPS's firewall has been configured to allow HTTP traffic through port 80.

We can now tell Fedora to start Apache on boot and also start the Apache daemon.

```
# systemctl enable httpd.service
# systemctl start httpd.service
```

Now if you visit `www.example.com` using your browser, you'll be greeted by a sample Apache Foundation
webpage. This means Apache has been installed and is ready to be configured.

#### Configuring Apache

The configuration file of Apache resides in `/etc/httpd/conf/httpd.conf`. We will now tailor it to suit our
needs.

Open the `httpd.conf` file:
```
# vi /etc/httpd/conf/httpd.conf
```

First, we'll tell Apache which IP it will listen for HTTP requests. Replace `xxx.xxx.xxx.xxx` with
the IP of your server.

```
Listen xxx.xxx.xxx.xxx
```

Then find the `ServerAdmin` and `ServerName` sections and update them with your email and domain name.

```
ServerAdmin myemail@somemail.com
...
ServerName www.example.com
```

Next, locate the `<Directory />` block and make it like so:

```
<Directory />
    AllowOverride FileInfo
    Require all denied
</Directory>
```

What we just did is tell apache that the contents of the root of the webserver should not be openly
accessible to the outside world. We also set `AllowOverride` to `FileInfo` from the default `Non` as
we want to be able to use the `ErrorDocument` directive for configuring custom error pages.

All the other configuration options can be safely left to their default values for now.

<br>

### Installing Jekyll

Jekyll has to be installed on both your VPS and also on your local machine; the Jekyll installation on
the VPS will build the entire site everytime you push ay changes to it and the local Jekyll copy will
help you build and design your site.

Jekyll is written using Ruby. Hence, we'll be using the Ruby package manager called RubyGems to install
all dependencies of Jekyll and Jekyll itself.

Install Ruby and RubyGems:

```
# dnf install ruby rubygems 
```

Now, install & update Jekyll:

```
$ gem install jekyll
$ gem update jekyll
```

<br>

### Making a Simple Jekyll Site

After you've installed Jekyll on both the VPS and your local machine, it's finally time to run Jekyll
and build our site!

Jekyll works by converting markdown into HTML files based on the configuration you set. Since this post is
not going to be a comprehensive Jekyll gide by itself, you are encouraged to take a look at the official
[Jekyll docs](https://jekyllrb.com/docs/home/) which is as beginner friendly as it can be.

On your local machine, go into the directory of your choice and create a new Jekyll site.

```
$ jekyll new mysite
```

This will create a new folder called `mysite` in the present working directory with bare minimum Jekyll
configuration.

Go into `foo` to build and test the default Jekyll site.

```
$ cd foo
$ jekyll build
$ jekyll serve --watch
```

Now, head over to `http://localhost:4000` and you should see your first ever Jekyll generated site!
This bare minimum default theme is called _Minima_, BTW.

Let's take a look at the contents of `mysite` directory. You will find that a lot of subfolders and files
have been generated. The file called `Gemfile` is responsible fot bundling required Jekyll dependencies
when a new site is being created. Aadvanced users usually use the Ruby gem called `bundle` with Jekyll
to create a Jekyll site based upon their configuration choice.

Next in line is `_config.yml`, which contains Jekyll configuration options. Typically,
it look likes this:

```
title: Title of MySite
email: myemail@example.com
description: > # this means to ignore newlines until "baseurl:"
  Welcome to MySite! I'm MyCoder! Nice to Meet ya.
baseurl: "" # the subpath of your site, e.g. /blog
url: "http://example.com"
twitter_username: mytwitterhandle
github_username:  mygithubhandle

# Build settings
markdown: kramdown
theme: minima
gems:
  - jekyll-feed
exclude:
  - Gemfile
  - Gemfile.lock
```

Update the relevant parts of the config file with your personalized info. Know that
you have to use exactly _two_ spaces for indentation at the beginning of each line
in the `_config.yml` file. Else your changes won't be recongnized by Jekyll!

The individul pages of our site are in markdown format (end with `.md`) extension
prior to Jekyll working its magic. Open `about.md` and notice the header of the file.
It should be something like this:

```
---
layout: page
title: About
permalink: /about/
---
```

This is called the front matter which is necessary in any page that you want Jekyll to
churn. Jekyll will only process and generate files with the front-matter.

All that remains to touch about Jekyll is the blog section. Blogging is what Jekyll
was built in the first place. To create a new post, just add a file with the naming
convention `yyyy-mm-dd-your-title-goes-here.markdown`. Don't forget to add the relevant
front-matter that Jekyll uses for blog posts:

```
---
layout: post
title:  "Your Title Goes Here"
date:   YYYY-DD-MM HH:MM:SS
categories: tag1 tag1 tag3 tag4
---
```

Notice how the about page uses the layout called `page` and the blog post uses a layout
called `post`. Layouts are one of the many powerful features of Jekyll. You can define
multiple custom layouts that can be shared by similar pages, e.g., the `page` layout
might be shared by the home, about and projects page whereas the `post` layout is used
by blog posts.

Lastly, the `_site` directory is where Jekyll puts the static HTML files after processing
the markdown files we supplied it. The contents of `_site` has to be copied to the document
root of Apache everytime we update our site and build it.

<br>

**NOTE:**

The `baseurl` and `url` fields should be properly set to the url you want your site
to be accessible from. Typically, `url` will be set to `www.example.com` and `baseurl`
will be set when you want your site to be accessible as a subpath of `url`. Make these
changes everytime before pushing to the repo on your VPS or else your site won't work
properly.

<br>

We will now create a remote `git` repo to track and push our changes to the main repo
on the VPS (which we will create in the next section).

```
$ git init
$ git add .
$ git commit -m "Initial commit"
```

<br>

### Taking Your Site Live

Just one last step remains to go live! We now have to create a bare `git` repo on the VPS
and configure the repo to build the site everytime you push into it and copy the generated
site to the document root.

Switch to a comfortable place in your VPS where you'd like to place the repo and then create
the repo.

```
$ cd /path/to/repo/dir/
$ mkdir mysite.git
$ cd mysite.git
$ git --bare init
```

So far so good. Now if you push your changes to this repo from your local machine, the changes
will be updated to this repo, but your site won't be updated; it needs to be built by Jekyll and
the contents of the `_site` folder have to be copied to the document root (`/var/www/html`).

To do that, we'll now create a git post-receive hook.

```
$ cd hooks
$ cp post-receive.sample post-receive
$ vi post-receive
```

We just copied the sample `post-receive` script supplied by `git`, which we'll now configure.

```
GIT_REPO=/path/to/repo/dir/mysite.git
TMP_GIT_CLONE=$HOME/tmp/mysite
PUBLIC_WWW=/var/www/html/

git clone $GIT_REPO $TMP_GIT_CLONE
jekyll build -s $TMP_GIT_CLONE -d $PUBLIC_WWW
rm -Rf $TMP_GIT_CLONE
exit
```

We also need to give this `post-receive` script executable rights.

```
# chmod +x post-receive
```

To enable the non-root user, using whose rights this script will be run,
to have write access to the document root (`/var/www/html`), add them to
the `apache` group.

```
$ sudo usermod -a -G apache <username>
```

Let's go back to your local machine now. Since we've already commit our changes, all that
needs to be done is push our changes, sit back and watch! :smiley:

```
$ git remote add <vps-deploy-alias> ssh://<username>@<vps-address>:/path/to/repo/dir/mysite.git
$ git push <vps-deploy> master
```

That's it! You've done it! Head over to `www.example.com` and if you've followed this post correctly,
you'll be greeted by your very own pseronal webpage! :wink: :beer: :pizza:

<br>

### Troubleshooting

***TO DO***

<br>

### References

1. DigitalOcean article on configuring a VPS:<br>
   * [www.digitalocean.com/community/tutorials/initial-setup-of-a-fedora-22-server](www.digitalocean.com/community/tutorials/initial-setup-of-a-fedora-22-server)
   * [https://www.digitalocean.com/community/tutorials/how-to-deploy-jekyll-blogs-with-git](https://www.digitalocean.com/community/tutorials/how-to-deploy-jekyll-blogs-with-git)
   
2. Apache HTTP Server Documentation:<br>
   [http://httpd.apache.org/docs/2.4/](http://httpd.apache.org/docs/2.4/)
   
3. Jekyll Documentation:<br>
   [https://jekyllrb.com/docs/quickstart/](https://jekyllrb.com/docs/quickstart/)
   
<br>

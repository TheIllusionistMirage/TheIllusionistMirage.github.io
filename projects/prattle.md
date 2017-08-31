---
layout: project
title: Prattle
permalink: prattle/
---

### What is Prattle?

Prattle is a simple chat application written using C++11, SFML and TGUI as a group project by me and [Amish Naidu](http://amhndu.github.io), with support from [Manasij Mukherjee](https://github.com/manasij7479) and [Bruno Van de Velde](https://texus.me/about).

Prattle uses a single server, multi client model and different instances of the server cannot communicate with each other. Full protocl and technical documentation guide can be found [here](https://github.com/TheIllusionistMirage/Prattle/blob/master/Documentation.md).

Current features supported are:
 * Login
 * Signup
 * Chatting with multiple clients
 * Friend system
 * Basic Notifications (unread messages)
 * User statuses (online/offline)
 * Server controller
 
Planned features can be found on our [TODO](https://trello.com/b/7T367Ya3/current-to-do-list).
 
**Prattle still doesn't boast of ANY security layer. Using Prattle for regular communication is not advisable.** :innocent:
<br>

### Screenshots
[![Prattle-1]({{site.url}}/resources/images/prattle-1-thumb.png "Login screen")]({{site.url}}/resources/images/prattle-1.png)

[![Prattle-2]({{site.url}}/resources/images/prattle-2-thumb.png "Connecting screen")]({{site.url}}/resources/images/prattle-2.png)

[![Prattle-3]({{site.url}}/resources/images/prattle-3-thumb.png "Chat screen")]({{site.url}}/resources/images/prattle-3.png)

[![Prattle-4]({{site.url}}/resources/images/prattle-4-thumb.png "Friend list")]({{site.url}}/resources/images/prattle-4.png)

[![Prattle-5]({{site.url}}/resources/images/prattle-5-thumb.png "Notification")]({{site.url}}/resources/images/prattle-5.png)

[![Prattle-6]({{site.url}}/resources/images/prattle-6-thumb.png "Chat in progress")]({{site.url}}/resources/images/prattle-6.png)

[![Prattle-7]({{site.url}}/resources/images/prattle-7-thumb.png "Adding a friend")]({{site.url}}/resources/images/prattle-7.png)
<br>

### Source, Building & Running

To build Prattle, there are a few dependencies:
 * C++11 compiler support (we used g++)
 * SFML 2.3.x or greater ([get SFML here](http://http://www.sfml-dev.org/download.php))
 * TGUI 0.7.x or greater ([get TGUI here](https://tgui.eu/download/))

First clone the repo:

`$ git clone https:github.com/TheIllusionistMirage/Prattle`

There are three separate CMake build scripts to build `prattle-server`,
`prattle-client` and `prattle-server-controller` separately.

Doing this will build the server(`prattle-server`):
```
$ cd Prattle
$ mkdir build && cd build
$ cmake ..
$ make
```

Doing this will build the client(`prattle-client`):
```
$ cd ../../
$ cd Client
$ mkdir build && cd build
$ cmake ..
$ make
```

Doing this will build the server-controller(`prattle-server-controller`):
```
$ cd ../../
$ cd Server-Controller
$ mkdir build && cd build
$ cmake ..
$ make
```

<br>
**How to run the server**

Now put the contents of `Prattle/Server/build` into a server (e.g., a VPS),
allow firewall exceptions for TCP traffic through port 19999 and then
invoke the server:

`$ ./prattle-server`

Log messages will be written to `server_log.txt`, which you can consult in
case of errors, warnings etc.

<br>
**Client and Server-Controller usage**

Now that the `prattle-server` is running, `cd` into the directory where
`prattle-client` was built. Then edit the `server_addr` field in
`Prattle/Client/resources/config/client.conf` to the public IP of the server
where `prattle-server` is running and invoke it:

`$ ./prattle-client`

Log messages will be written to `client_log.txt`, which you can consult in
case of errors, warnings etc.

Similarly, the `prattle-server-controller` can be used to control the
`prattle-server`. It requires using a passphrase to bind the `prattle-server-controller`
to the `prattle-server`.

<br>
**NOTES**

1. Port 19999 is used by default. You are free to edit the port of course. Just make
   the necessary changes in `Prattle/Server/resources/server.conf` and 
   `Prattle/Client/resources/config/client.conf`.
   
2. You can also host the `prattle-server` on a regular machine provided the machine
   has a static IP and required network configuration so that clients can connect to it
   (firewall settings, NAT filters, etc.).
<br>

### License

Prattle is licensed under the [MIT License](https://opensource.org/licenses/MIT).

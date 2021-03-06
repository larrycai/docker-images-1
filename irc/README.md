# IRC

### Status

WIP - Work in progress.

We now have basic and reliable working chain of services. Everything comes up automatically. All with basic and generic settings.

Left todo:

* Improve documentation
* Upgrade irssi version, retest the default selection of scripts
* Configure weechat client
  * Setup glowing-bear web interface
* Publish on Docker Hub

### Introduction

A collection of pre-configured irc programs and services. Designed to make IRC easy. Everything designed to come up on container start, all work together.

What's included:

* IRC clients
  * irssi - config finished
  * weechat - WIP
    * glowing-bear (web interface)
  * tmux and sshd for terminal server

* IRC services
  * ZNC bouncer
  * Bitlbee (for IM accounts)
  * ngircd - your own local / private IRC server
    * atheme services - for nickserv, sasl etc
  * Limnoria IRC bot (aka supybot)

### Connection diagram

General diagram how these programs are connected up:

```sh

atheme <--- ngircd <-\        /- limnoria IRC bot (supybot)
Public IRC Servers <--- ZNC <--- irssi <--- TMUX Session <--- SSH <--- Client computer
           Bitlbee <-/        \- weechat -/
                                    \- weechat relay <--- glowing bear <--- web browser
```

And thanks to TMUX, simultaneous ssh logins are supported. So you can have the same irssi or weechat text-based session open on multiple machines no problem. And multiple glowing-bear instances of the web interface too.

### IRC Servers

IRC Networks and channels pre-configured

* Public IRC networks

  * barton
    * `#ngircd`
      * public help / support channel

  * dalnet

  * efnet

  * freenode
    * `#atheme`
      * public help / support channel
    * `#irssi`
      * public help / support channel
    * `#limnoria`
      * public help / support channel
    * `#weechat`
      * public help / support channel
    * `#znc`
      * public help / support channel

  * gnome

  * ircnet

  * mozilla

  * oftc
    * `#bitlbee`
      * public help / support channel

  * quakenet

  * undernet


* Private / localhost services

  * "ngircd"
    * `#local` sandbox channel (for limnoria / "supybot")

  * bitlbee
    * `&bitlbee` - connect using IM chat protocols



### Irssi Scripts

The latest irssi scripts are already downloaded into the container with `git`. In fact, every time irssi program is restarted, we run the following command automatically:

```sh
cd /scripts.irssi.org && git pull
```

So no need to worry about using script-assist to manage your scripts for you. There is no need for it. The local copy of the scripts repo always be kept fully up-to-date with every container restart. Or whenever you have manually `/quit` the irssi program.

Most scripts are symlinked from irssi's config folder, into the git repo:

To add a new script just create a symlink into your irssi scripts folder. For example:

```sh
ln -s /scripts.irssi.org/scripts/SCRIPTNAME.pl /config/irssi/scripts/autorun/
```

Linking into the `.../autorun/` subfolder makes the script load automatically on irssi startup. More information at:

https://scripts.irssi.org/

### Tweaked scripts

A few of the scripts had to be modified / tweaked in order to work. So those ones are not symlinks but copied files. Other scripts are not tweaked, but also copy files (not simplinks). Which did not exist at time of writing in irssi's official scripts repo.

### Not met script dependancies

All the included scripts were tested to work properly in the container. However you may want to add other scripts to your config later on. So why isn't / aren't my optional irssi script(s) working then?!!?

Well many optional irssi scripts also require their own specific dependancy(ies). Which are not necessarily come pre-installed. There are 2 kinds of dependancies: APT deps and perl deps.

For perl deps, you can look at the import / include lines  like `use Perlmodule::Name;` at the top of the script. Then use `cpanm Perlmodule::Name` to install them.

### Solarized Irssi Theme

This theme has been modified with statusbar, awaybar, hilight colors, etc. The original verison of the theme is available here:

https://github.com/huyz/irssi-colors-solarized

### Editing configuration files

The text editor `nano` is included for simple editing of configuration files. e.g. `nano /config/irssi/config`. Be careful to do that under the right unix user instead of `root`.

For example:

```sh
sudo su -l znc [return]
nano /config/znc/configs/znc.conf [return]

# or
sudo su -l irssi [return]
nano /config/irssi/config [return]
```

### Configuration

Pre-seeded Configuration Files:

All configuration files are held in the working folder `/config`. There are different versions of the config folder. See [`config.default/README.md`](#config.default/README.md) and [`config.custom/README.md`](#config.custom/README.md) for more information about those.

SSH Configuration:

For ssh terminal access you can put your existing ssh public key (e.g. `id_rsa.pub`) into `/config/irssi/.ssh/known_hosts` file. Else a new keypair will be automatically generated per container on first run. And the resultant `id_rsa` private key file can be copied from `/config/irssi/.ssh/id_rsa` of your new running container's mounted `/config` volume.

ZNC Configuration:

Primarily you will want to change the znc admin username and password to something else more secure. And to be your own IRC nickname.

The znc server will be configurable from any standard web browser, available on SSL protocol `https://$your_container_ip:6697`. The default username and password for znc admin user is `znc:znc`. It is recommended to then goto manage users --> Duplicate the default `znc` user with your own IRC nick name.

The `znc.conf` text file can also be edited directly. It is stored in the usual loacation at `/config/znc/configs/znc.conf`

Weechat Config:

* Placeholder
* Not setup yet

IRSSI Config:

Assuming that you have changed the znc username and password, then you must also change those `/server` lines in the irssi config file. To use your new znc nick name and password. This file is located at `/config/irssi/config`.

IRC Data:

Generated data gets written to thedocker volume `/irc`.

Note: An additional docker image, `dreamcat4/nginx` or `dreamcat4/samba` is also useful. To more easily access the contents of this `/irc` data folder on your LAN.

 Logs:

By default all IRC channel logs are kept by ZNC and written to the `/irc/znc/logs` folder.

 URLs:

The `urlplot` script is pre-configured to write all posted urls to `/irc/irssi/urlplot` folder. In both HTML and CSV format. The generated HTML file can bookmarked in your local web browser for easy access.

Bitlbee:

Tends to be configured interactively by commands to the `&bitlbee` control channel, all done while running irssi client. Or by hand-editing the config file at `/config/bitlbee/bitlbee.conf`.

TMUX:

The config file is located at `/config/irssi/.tmux.conf`

This default tmux config does not really need any specific per user configuration. Unless you want to customize the behaviour of tmux more to your liking. For example to hide the `tmux ...` line at the top of the screen, or change the key-bindings etc.

Limnoria:

NOTE: YOU SHOULD CHANGE YOUR BOT'S NICK FROM `supybot` ---> `YOUR BOT`

* The config file is located at `/config/limnoria/supybot.conf`
* The default owner nickname of this bot is `znc_user`
* The default owner password for this bot is `supybot`
* Connects to the znc user named `supybot` with znc password `supybot` <-- CHANGE THIS IN YOUR ZNC CONFIG TOO
* This bot is pre-configured to only connect join the channel `#local` on your own private `ngircd` IRC server
* Public IRC networks are also pre-configured in `supybot.conf`. They are just all disabled by default except for ngircd

How to identify yourself to the bot as it's owner:

```
< znc_user> @list --unloaded
#. supybot# znc_user: Error: You don't have the owner capability. If you think that you should have this capability, be sure that you are identified before trying again. The 'whoami' command can tell you if you're identified.

< znc_user> @whoami
#. supybot# znc_user: I don't recognize you. You can message me either of these two commands: "user identify <username> <password>" to log in or "user register <username> <password>" to register.

/query supybot user identify znc_user supybot
<znc_user> user identify znc_user supybot
-supybot(~supybot@localhost)- The operation succeeded.
```

You will need to change the owner in `supybot.conf` to your real nick, and change the owner password to something more secure than `supybot` before taking it online to public irc servers.

EXTRA NOTE:

Going online to public servers with an IRC bot can be considered bad behaviour. Or suspicious due to botnets etc. So using a bot in the wrong way(s) can very quickly get you banned. Either from a specific channel, or IRC network, or even multiple IRC networks! Due to shared global blacklisting mechanisms. So be sure to check any relevant bot policies for those specific network(s) and channel(s) before taking your supybot online to any specific networks or channels. Check with a actual human being, and be asking for clarification / permission to do so where applicable.

Ngircd:

Provides the `#local` channel. Which is where znc and the supybot are pre-configured to automatically join to. This is your playground to configure / use the bot before taking it online elsewhere.

* The config file is located at `/config/ngircd/ngircd.conf`
* This irc server is pre-configured to listen only on the localhost `127.0.0.1` network interface.
* Includes pairing passwords for the localhost `atheme` services daemon.

NOTE:

This IRC server is minimally configured. But should be safe to use for private localhost use only. To be taking this ngircd instnace public (exposing ports to outside). Then at very least you should change the atheme pairing passwords. In both ngircd config file and atheme's config file.

However its not recommended to use this single fat image for any public of semi-public hosting. As other IRC programs are running in the same image. For better hardening, use seperate sigle-service docker images, each linked together. For example 1 container for ngircd, 1 container for atheme services.

atheme services:

* The config file is located at `/config/atheme/atheme.conf`
* These servcies are only minimally configured (enough to work with the ngircd instance)
* The actual services offered (nickserv, chanserv, sasl, etc) are all left at defaults
* The pairing passwords (for connection to this IRC services daemon) are also configured in `ngircd.conf`

### Bitlbee

Comes installed with all the necessary 3rd party plugins for all the most popular non-core protocols including WhatsApp, Steam Chat, Telegram, Facebook, Skype (via 'SkypeWeb') and so on.

So for any of the below links, you do not need to install any new pkgs / files, they should already be available inside of Bitlbee. Some IM procotols do requires a specifal setup instructions however. As can be read about in the following pages:

* LibPurple
  * https://wiki.bitlbee.org/HowtoPurple

* WhatsApp
  * https://wiki.bitlbee.org/HowtoWhatsapp

* Skype
  * https://wiki.bitlbee.org/HowtoSkypeWeb

* SIPE
  * https://wiki.bitlbee.org/HowtoSIPE

* Facebook
  * https://wiki.bitlbee.org/HowtoFacebookMQTT

* Steam Chat
  * https://github.com/jgeboski/bitlbee-steam

* Telegram
  * https://github.com/majn/telegram-purple

* Torchat
  * https://github.com/prof7bit/TorChat


**Protcol not listed in &bitlbee?**

Bitlbee's `help purple` command will list all of the available 3rd party plugins *which were installed via libpurple*. However certain other 'native' bitlbee plugins do not appear there. And are not listed elsewhere, eg in `help account add`. Yet they are in fact already installed and fully installed.

**How to check if a protocol is supported:**

To check that a specific protocol is supported, type `account add PROTOCOL username password`. If the answer isn't `unknown protocol`, then that IM protocol is indeed present and supported. For example:

```sh
&bitlbee:
11:44 <@znc_user> account add PROTOCOL username password
11:44 <....@root> Unknown protocol

11:44 <@znc_user> account add steam username password
11:44 <....@root> Account successfully added with tag steam

11:44 <@znc_user> account add facebook username password
11:44 <....@root> Account successfully added with tag facebook
```

**Missed Protocol:**

* tox-prpl - tox API Changed, plugin broken and needs updating
  * https://github.com/jin-eld/tox-prpl/issues/54

**Additional Protocols:**

https://developer.pidgin.im/wiki/ThirdPartyPlugins#AdditionalProtocols

### Service ports

This container runs multiple services on it, and as several different user accounts. They are as follows:

```sh
INTERFACE, TCP_PORT, (PROTOCOL), UNIX_USER, SERVICE, COMMENT

0.0.0.0 (all interfaces), 22, (SSH term), irssi, irssi client in a tmux session, always running

0.0.0.0 (all interfaces), 6697, (SSL HTTPS), znc, znc web interface, for configuring the znc service

0.0.0.0 (all interfaces), 6697, (SSL IRC), znc, znc irc server, for other irc clients / mobile apps etc. can be disabled in znc.conf file, then restart container / znc server

localhost, 9997 (SSL IRC), znc, znc irc server, only visible inside container, for client: conatiner's local instance of irssi program

localhost, 6111, (http IRC), bitlbee, bitlbee irc server, only visible inside container, for client: local ZNC server

localhost, 6667, (http IRC), ngircd, ngircd irc server, only visible inside container, for client: local ZNC server
```

The user is intended to access the IRC service primarily as the user `irssi`, on ssh port 22. Via public-private SSH key authentication (no ssh password). The container's irssi program itself is always kept running under a perpetual tmux session named `tmux`. Which supports simultaneous logins from multiple computers.

### Connecting to the irssi session

From a remote machine:

Connect via ssh terminal session by logging in as the user `irssi`. For example: `ssh irssi@your_container_ip`. You must have a keyfile which is listed in the container's ssh `known_hosts` file.

From local machine (same docker host):

```sh
docker exec -it CONTAINER_NAME bash [return]
irssi [return]
```

### Disconnecting from an irssi session

In these cases, the irssi program and it's TMUX session will be kept running as normal:

* It is preferred to disconnect with the default TMUX keybinding of `Ctrl-B` then `d`.
* You may also disconnect by killing ssh window in your ssh client. Eg `ctrl+c`

In these cases, the irssi program will exit:

* Quit the irssi program and naturally end the tmux session with `/quit` in irssi
* Kill the irssi program and disconnect by killing the tmux session with `Ctrl-B` then `@`

However in these last 2 cases ^^ a new tmux session and instance of the irssi program will be re-launched within < 3 seconds. So in general your irssi client is always kept running. So long as its docker container is kept running.

The container's ZNC and Bitlbee servers are also kept running too. Like irssi, if they crash or are exited for any reason, then they will be automatically restarted within a few seconds.

## Per-user setup

1. Configure your znc user account
2. Setup nickserv / identify with services

### ZNC user configuration

This is set your own IRC nickname.

* Using a web browser connect to znc's web interface. It is available on HTTPS and port 6697: `https://CONTAINER_IP:6697`
* Log in to znc with username: `znc`, password: `znc`
* Go to "Manage users" page
* Click "Clone" next to znc user
* Replace all instances of 'znc' and 'znc_user' (all the various different usernames) with your own IRC nickname, realname etc. 
* Set a new and more secure password for you to login to znc server with.
* Click "Clone and return" at the bottom of page.
* Make sure your new user account still has admin status, and logout / login with it.
* Now you can disable / delete the previous `znc` user account. To stop it trying to log into IRC servers. Or at least change it's password to something more secure.

After changing our username, we now need to also edit the irssi config file at `/config/irssi/config`. You can do this inside the container with the cmd `nano /config/irssi/config`. Or from the host, looking inside the location of the mounted `/config` docker volume. Either way just make sure that the file's user ownership remains the same after editin. Which is the container's `irssi` unix user account. And file ownership is not changed to `root` user etc.

* In the irssi/config file, update the username and password login credentials on each server's `password = ` config line. To be the new login for your znc user account. There is only 1 znc user account. But there are multiple `password = ` declarations, one for each unique IRC service (Freenode, EFNet, etc).

Now restart irssi program with `/quit` command, or `docker restart` your container for changes to take effect. Irssi should now be trying to login under your own IRC nickname.

### Setting up nickserv on znc

These instructions explain how to register your nick with nickserv, and then to save that login credention into znc's `*nickserv` module. The proceedure is the the same here for all of the pre-configured networks listed here. Other networks may be slightly different (e.g. quackent undernet), where they may call nickserv service something else.

1. Be sure znc is already setup with your chosen / desired nickname for target server/network (eg freenode or whatever). And that znc has logged you into that server with the right nick you want to use.

2. Just browse onto any channel, which is on the same network where you want to setup nickserv services with. eg:

```
/join -[SERVER] [ANYCHANNEL] # switches you to the target SERVER in irssi
```

3. Open up a new window to the server's real nickserv service. And check you are logged on to the legitimate service:

```
/nickserv help
```

4. Register your nickname / password / email address

```
/nickserv register [YOUR PASSWORD] [YOUR EMAIL]
```

You nickname is implicit, as you are already logged in under your temporary nickname, the thing you wish to make permanment. Go through any extra steps (for example email confirmation etc). Your nickname should now be registered with YOUR PASSWORD.

5.  Open up another window, this time to the `*nickserv` znc module. Again while browsed onto the same target network. Eg:

```
/msg -freenode *nickserv help
```

6. Save your nickserv password into znc for this network:

```
/msg -freenode *nickserv set [YOUR PASSWORD]
```

^^ Notice the `*` at the front of `*nickserv` this time. Which means we are passing the SET message to znc's own local nickserv module. Once this step is done znv should automatically handle all future logging in / nickserv identification all for you.

ZNC Nickserv module documentation: http://wiki.znc.in/Nickserv

### Per-network Authentication methods

At a minimum you should authenticate your IRC username with nickserv on every network that supports it. In addition to that some fewer networks optionally support better authentication methanisms e.g. by SASL or SSL client certificate. At least SASL is very easy to setup.

#### &bitlbee

* register a new access password with the `register` command
  * navigate to the &bitlbee window and type `help register`

* then have znc login to the local bitlbee server with znc's 'perform' module
  * http://wiki.znc.in/Perform#Bitlbee

#### dalnet

* nickserv
  * Register with nickserv using the general method described ^^ earlier

More info: https://www.dal.net/services/servcmd.php?c=9

* then tell znc to log you with the following command for dalnet
  * http://wiki.znc.in/Nickserv#SetCommand

#### efnet

* none really, only 'ident'. which is hardly anything
  * Just be sure that your znc user's 'ident' field is set the same as your nickname

#### freenode

* nickserv
  * Register with nickserv using the general method described ^^ earlier

* sasl
  * https://www.freenode.net/sasl/sasl-znc.shtml

#### gnome

* nickserv
  * Register with nickserv using the general method described ^^ earlier

Further info: https://wiki.gnome.org/Sysadmin/IRC#Registering_your_nickname

#### ircnet

* none really, only 'ident'. which is hardly anything
  * Just be sure that your znc user's 'ident' field is set the same as your nickname

#### mozilla

* nickserv
  * Register with nickserv using the general method described ^^ earlier

Further info: https://wiki.mozilla.org/IRC#Register_your_nickname

#### oftc

* nickserv
  * Register with nickserv using the general method described ^^ earlier

* SSL client certificate
  * http://www.oftc.net/NickServ/CertFP

Further info: http://www.oftc.net/Services/

#### quakenet

* the 'q' service bot
  * https://www.quakenet.org/help/q/how-to-register-an-account-with-q

* when you first run irssi, a `*q` window should appear, where you can specify to znc your quakenet account's username / password

* login with [znc's 'q' module](http://wiki.znc.in/Q) on quakenet config page of znc's webAdmin

#### undernet

* register a new account with undernet
  * https://cservice.undernet.org/live/newuser.php

* login to the 'x' service bot with znc's 'perform' module
  * http://wiki.znc.in/Perform#Identifying_to_services

Further info: http://help.undernet.org/faq.php?what=cservice#03

### File permissions

There is the main unix account `irssi`. However this container also has unix accounts named `bitlbee` `znc` and `eggdrop` for those other supportive services.

By specifying an alternative uid and gid as a number, this lets you control which folder(s) those specific services have read/write access to. For example, setting these daemons to your own local user account's UID will allow you to access these services IRC `/config` files from outside of the docker session and on the host machine.

The default `uid:gid` of these accounts is named after their TCP port number. Except eggdrop --> `3996`rop, which is leet for `eggd`. As is bitlbee's chosen numeric `6111` leet for `bitl`.

```sh
USER, UID, GID
irssi, 22, 22
znc, 6697, 6697
eggdrop, 3996, 3996
bitlbee, 6111, 6111
```

You can change the unix UIDs and/or GIDs to your liking by setting the following docker env vars:

```sh
irssi_uid=XXX
irssi_gid=YYY

znc_uid=XXX
znc_gid=YYY

eggdrop_uid=XXX
eggdrop_gid=YYY

bitlbee_uid=XXX
bitlbee_gid=YYY
```

### Docker Compose

Sorry there is no example for Docker Compose at this time. But you may do something similar:

```sh
crane.yml:

containers:

  irssi:
    image: dreamcat4/irssi
    run:
      net: none
      volume:
        - /path/to/my/irc/current.config:/config
        - /path/to/my/irc/logs:/logs
      env:
        - irssi_uid=65534
        - irssi_gid=65534
        - pipework_wait=eth0
        - pipework_cmd_eth0=eth0 -i eth0 @CONTAINER_NAME@ 192.168.1.17
      detach: true
```

The `pipework_` variables are used to setup networking with the `dreamcat4/pipework` helper image.


### Alternative

Decent irc web client:

https://kiwiirc.com/client


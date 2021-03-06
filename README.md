# Syndicator

**I have made following changes to the tool:**
- Removed __back-in-time__ dependency, it was causing problems for some reason.
- Updated Syndicator3 to support Python 3.6 and few more fixes

Syndicator is a python script providing a "synchronization indicator" for the file synchronzation software [Unison](http://www.cis.upenn.edu/~bcpierce/unison/download/releases/stable/unison-manual.html) on Ubuntu, i.e. a dynamic icon in the Unity panel that indicates the current synchronization state.  

## Features
Each time you log in, Syndicator will run [Unison](http://www.cis.upenn.edu/~bcpierce/unison/download/releases/stable/unison-manual.html).  If running Unison fails (for example because there is no connection to the server), Syndicator keeps trying at intervals of 1, 2, 4 .... 300 seconds.  The current status is displayed in the indicator panel as follows:

icon | status
-----|--------
![icon during backup](https://rawgithub.com/TentativeConvert/Syndicator/master/icons/backup1.svg)  ![icon during backup](https://rawgithub.com/TentativeConvert/Syndicator/master/icons/backup2.svg) | creating backups
![icon during sync](https://rawgithub.com/TentativeConvert/Syndicator/master/icons/sync1.svg)  ![icon during runtime](https://rawgithub.com/TentativeConvert/Syndicator/master/icons/sync2.svg) | syncing
![icon on success](https://rawgithub.com/TentativeConvert/Syndicator/master/icons/sync-good.svg) | repositories are fully synchronized
![icon on error](https://rawgithub.com/TentativeConvert/Syndicator/master/icons/sync-error.svg) | error (will restart automatically in x seconds)

Additional information is provided via a menu: the current output of Back In Time/Unison, a list of recently synchronized files, and a list of errors.  

![screenshot](documentation/screenshot.png)

Clicking on the first menu entry brings up a window displaying the last few (rather than just the last one) line of output from Back In Time/Unison.

## Prerequisites (overview)
1.  Linux operating system on your client machine (tested only on Ubuntu, see above)
2.  [AppIndicator developer libraries](#appindicator-developer-libraries) for Python
2.  [ssh access to a server](#ssh-access-to-server-via-public-key-authentication), preferably via public key authentication
3.  a working installation of [Unison](#unison) on both client and server (version 2.48.3 or later)
4.  ~~(optional) a working installation of [Back In Time](#back-in-time), or of some other command line backup tool~~

Brief details on how to set up 2-4 are included further below.

## Installation/configuration

Syndicator comes in two versions:

Name | Language |    | System | Desktop | Unison | Back in Time 
-----|----------|---|--------|---|---|---
Syndicator 3 | Python 3 | tested with:  | Ubuntu 18.04 | GNOME | 2.48 | 1.1.24 

Download the folder [syndicator3](https://minhaskamal.github.io/DownGit/#/home?url=https://github.com/tanwirahmad/Syndicator/tree/master/syndicator3).

Open `config.py`and adapt the following lines:
```
sync_command = "unison XPS12-reh -repeat watch"
```
That is, replace the `2` in the first line with the id of the Back In Time profile that you would like Syndicator to use, and replace `XPS12-reh` by the name of the relevant Unison profile.  The Unison profile should include the following options:
```
batch = true
prefer = newer
copyonconflict = true
```
Alternatively, you can include these flags as command line arguments in the value of the `sync_command`, of course. 

Finally, make `main.py` executable (if necessary) with `chmod +x main.py`.
You should now be able to start Syndicator with one of the following commands:

Syndicator 3: call `python -m syndicator3.main` from the *parent folder* of the folder syndicator3.

The icons used by Syndicator are essentially the UbuntuOne icons delivered with Ubuntu 14.04.  You can change these either by replacing the relevant files in the `icons` folder or by editing `config.py`.  For [at least some of the] icons in `/usr/share/icons/` you only need to supply the file name, not the full path.  

## Prerequisites (instructions)
### ssh access to server via public key authentication
Generate a public-private-keypair on your client with:
```
$ ssh-keygen -t rsa -b 4096
```
You will be promted for a passphrase.  Choose it carefully.  Empty passphrases work, but then whoever manages to hack into your computer will automatically gain access to your server.

There should now be one private key (`id_rsa`) and one public key (`id_rsa.pub`) in `~/.ssh`.
Copy the public key to the server:
```
$  scp .ssh/id_rsa.pub username@server.address:/user-directory/.ssh/otherkeys
```
Append this public key to the file `.ssh/authorized_keys` on the server:
```
$  ssh username@server.address
$  cd ~/.ssh
$  cat otherkeys/id_rsa.pub >> authorized_keys
```
If the server is correctly configured, future calls of `ssh username@server.address` should no longer ask for a password.  

### AppIndicator developer libraries
The scripts rely on a Python library called AppIndicator3: one of the first lines in `Indicator.py` is `from gi.repository import AppIndicator3`.  Which packages precisely you need in order for this Python library to be available I do not know how to tell.  On Ubuntu 14.04 with Unity desktop, it seems the necessary Python 2 library is usually installed by default, but some users reported that they needed to install it manually with:
```
$ sudo apt-get install gir1.2-appindicator3-0.1
```
On Ubuntu 18.04, I managed to install the corresponding Python 3 library with:
```
$ sudo apt install libappindicator3-dev 
```

### Unison
Binaries for Unison 2.48.3 built for 64-bit versions of  Ubuntu 14.04  are included in the `unison-binaries` folder in this repository, and these binaries also seem to run fine on Ubuntu 16.10 and Ubuntu 18.04.  So if you are running any of these versions of Ubuntu on your client and server, you can simply copy these binaries to any folder in which the system would usually look for them, e.g. `usr/bin/` or `~/bin/`.  Then read the [manual](http://www.cis.upenn.edu/~bcpierce/unison/download/releases/stable/unison-manual.html) to find out how to configure your profile.

Unison is also available through the official Ubuntu channels and other distributions.  Note however that continuous synchronization of 'watched' folders only works with Unison versions ≥ 2.48.  If the above binaries don't work for you, and if no sufficiently young versions of Unison are available through your system repositories, you might need to compile your own Unison binaries.  For this, you will need an OCaml compiler.  

#### Build Unison locally (instructions for Ubuntu 14.04)
- Install Ocaml from the software center (4.01.0-3ubuntu3).
- Download Unison from http://www.cis.upenn.edu/~bcpierce/unison/ and unpack, e.g. to `~/Downloads/unison-2.48.3`.
- Switch to this folder and run make:
``` 
$ cd ~/Downloads/unison-2.48.3
$ sudo make UISTYLE=text
```
  Test success with `$ ./unison` -- this should display a usage message.  
- Copy the binaries to `~/bin` or whereever you want them, and create a link so that you can start Unison simply by typing `unison` in the command line:
```
$ mkdir ~/bin
$ cp ~/Downloads/unison ~/bin/unison-2.48.3
$ cp ~/Downloads/unison-fsmonitor ~/bin/unison-fsmonitor
$ ln -s unison-2.48.3 unison
```
Calling `unison` should now display the same usage message as before.  

#### Build Unison on the server:    
This should work exactly as above.  *However, you need to make sure you have the same version of Unison on both client on server, and they should be compiled with the same version of OCaml.*  Unison 2.48 compiled with OCaml < 4.02 conflicts with Unison 2.48 compiled with OCaml 4.02!  If you're lucky and the server uses the same architecture as your client you can of course simply copy the binaries. 

If there are several versions of Unison on the server, you need to start Unison with the flag `--add-version-no`.  More precisely, starting `unison-2.48.3` on the client with `--add-version-no` will call `unison-2.48` (without subversion number) on the server, so in addition to setting this flag you will need to create a soft link `unison-2.48` on the server pointing to `unison-2.48.3`.

#### Create a Unison profile:
For Unison to actually do anything, you will need to create a profile `myprofile.prf` in `~/.unison/` -- see the [manual](http://www.cis.upenn.edu/~bcpierce/unison/download/releases/stable/unison-manual.html).

### Back In Time
See https://github.com/bit-team/backintime.

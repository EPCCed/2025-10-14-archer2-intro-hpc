---
title: "Connecting to the remote HPC systemd"
teaching: 25
exercises: 15
---



:::::: questions
  - How do I open a terminal?
  - How do I connect to a remote computer?
  - What is an SSH key?
::::::

:::::: objectives
  - Connect to a remote HPC system.
::::::

# Opening a Terminal

Connecting to an HPC system is most often done through a tool known as "SSH"
(Secure SHell) and usually SSH is run through a terminal. So, to begin using an
HPC system we need to begin by opening a terminal. Different operating systems
have different terminals, none of which are exactly the same in terms of their
features and abilities while working on the operating system. When connected to
the remote system the experience between terminals will be identical as each
will faithfully present the same experience of using that system.

Here is the process for opening a terminal in each operating system.

### Linux

There are many different versions (aka "flavours") of Linux and how to open a
terminal window can change between flavours. Fortunately most Linux users
already know how to open a terminal window since it is a common part of the
workflow for Linux users. If this is something that you do not know how to do
then a quick search on the Internet for "how to open a terminal window in" with
your particular Linux flavour appended to the end should quickly give you the
directions you need.

### Mac

Macs have had a terminal built in since the first version of OS X since it is
built on a UNIX-like operating system, leveraging many parts from BSD (Berkeley
Software Distribution). The terminal can be quickly opened through the use of
the Searchlight tool. Hold down the command key and press the spacebar. In the
search bar that shows up type "terminal", choose the terminal app from the list
of results (it will look like a tiny, black computer screen) and you will be
presented with a terminal window. Alternatively, you can find Terminal under
"Utilities" in the Applications menu.

### Windows

While Windows does have a command-line interface known as the "Command Prompt"
that has its roots in MS-DOS (Microsoft Disk Operating System) it does not have
an SSH tool built into it and so one needs to be installed. There are a variety
of programs that can be used for this; a few common ones we describe here, as
follows:

#### MobaXterm

MobaXterm is a terminal window emulator for Windows and the home edition can be
downloaded for free from
[mobatek.net][mobatek]. If you
follow the link you will note that there are two editions of the home version
available: Portable and Installer. The portable edition puts all MobaXterm
content in a folder on the desktop (or anywhere else you would like it) so that
it is easy to add plug-ins or remove the software. The installer edition adds
MobaXterm to your Windows installation and menu as any other program you might
install. If you are not sure that you will continue to use MobaXterm in the
future, the portable edition is likely the best choice for you.

Download the version that you would like to use and install it as you would any
other software on your Windows installation. Once the software is installed you
can run it by either opening the folder installed with the portable edition and
double-clicking on the executable file named `MobaXterm_Personal_11.1` (your
version number may vary) or, if the installer edition was used, finding the
executable through either the start menu or the Windows search option.

Once the MobaXterm window is open you should see a large button in the middle
of that window with the text "Start Local Terminal". Click this button and you
will have a terminal window at your disposal.

#### Git BASH

Git BASH gives you a terminal like interface in Windows. You can use this to
connect to a remote computer via SSH. It can be downloaded for free from
[here][gitbash].

#### Windows Subsystem for Linux

The Windows Subsystem for Linux also allows you to connect to a remote computer
via SSH. Instructions on installing it can be found
[here][wsl].

## Creating an SSH key

SSH keys are an alternative method for authentication to obtain access to 
remote computing systems. They can also be used for authentication when
transferring files or for accessing version control systems. In this section
you will create a pair of SSH keys, a private key which you keep on your
own computer and a public key which is placed on the remote HPC system
that you will log in to.

### Linux, Mac and Windows Subsystem for Linux

Once you have opened a terminal check for existing SSH keys and filenames
since existing SSH keys are overwritten,

```bash
$ ls ~/.ssh/
```

then generate a new public-private key pair,

```bash
$ ssh-keygen -o -a 100 -t rsa -b 4096 -f ~/.ssh/id_ARCHER2_rsa
```

- `-o` (no default): use the OpenSSH key format,
  rather than PEM.
- `-a` (default is 16): number of rounds of passphrase derivation;
  increase to slow down brute force attacks.
- `-t` (default is [rsa](https://en.wikipedia.org/wiki/RSA_(cryptosystem))):
  specify the "type" or cryptographic algorithm. 
  [ed25519](https://en.wikipedia.org/wiki/EdDSA)
 is faster and shorter than RSA for comparable strength.
- `-f` (default is /home/user/.ssh/id_algorithm): filename to store your keys.
  If you already have SSH keys, make sure you specify a different name:
  `ssh-keygen` will overwrite the default key if you don't specify!

The flag `-b` sets the number of bits in the key.
The default is 2048. EdDSA uses a fixed key length,
so this flag would have no effect.

When prompted, enter a strong password that you will remember. 
Cryptography is only as good as the weakest link, and this will be 
used to connect to a powerful, precious, computational resource.

Take a look in `~/.ssh` (use `ls ~/.ssh`). You should see the two 
new files: your private key (`~/.ssh/key_ARCHER2_rsa`) and 
the public key (`~/.ssh/key_ARCHER2_rsa.pub`). If a key is 
requested by the system administrators, the *public* key is the one
to provide.

:::::::::::: callout

## PRIVATE KEYS ARE PRIVATE

  A private key that is visible to anyone but you should be considered compromised,
  and must be destroyed. This includes having improper permissions on the directory
  it (or a copy) is stored in, traversing any network in the clear, attachment on 
  unencrypted email, and even displaying the key (which is ASCII text) in your 
  terminal window.

  Protect this key as if it unlocks your front door. In many ways, it does.

::::::::::::

:::::::::::: callout

#### Further information
 
For more information on SSH security and some of the
flags set here, an excellent resource is 
[Secure Secure Shell](https://stribika.github.io/2015/01/04/secure-secure-shell.html).

::::::::::::

### Windows

On Windows you can use

 - puttygen, see the Putty [documentation](https://www.chiark.greenend.org.uk/~sgtatham/putty/docs.html)
 - MobaKeyGen, see the MobaXterm [documentation](https://mobaxterm.mobatek.net/documentation.html) 

## Logging onto the system

With all of this in mind, let's connect to a remote HPC system. In this
workshop, we will connect to ARCHER2 --- an HPC system
located at the University of Edinburgh. Although it's unlikely that
every system will be exactly like ARCHER2, it's a very good
example of what you can expect from an HPC installation. To connect to our
example computer, we will use SSH (if you are using PuTTY, see above).

SSH allows us to connect to UNIX computers remotely, and use them as if they
were our own. The general syntax of the connection command follows the format
`ssh -i ~/.ssh/key_for_remote_computer yourUsername@remote.computer.address` 
when using SSH keys and `ssh yourUsername@some.computer.address` if only
password access is available.  Let's attempt to connect to the HPC system
now:

```bash
ssh -i ~/.ssh/key_ARCHER2_ed25519 yourUsername@login.archer2.ac.uk
```

or

```bash
ssh -i ~/.ssh/key_ARCHER2_rsa yourUsername@login.archer2.ac.uk
```

or if SSH keys have not been enabled 

```bash
ssh yourUsername@ARCHER2
```


```output
This node is running Cray's Linux Environment version 1.3.2

#######################################################################################

        @@@@@@@@@
     @@@         @@@            _      ____     ____   _   _   _____   ____    ____
   @@@    @@@@@    @@@         / \    |  _ \   / ___| | | | | | ____| |  _ \  |___ \
  @@@   @@     @@   @@@       / _ \   | |_) | | |     | |_| | |  _|   | |_) |   __) |
  @@   @@  @@@  @@   @@      / ___ \  |  _ <  | |___  |  _  | | |___  |  _ <   / __/
  @@   @@  @@@  @@   @@     /_/   \_\ |_| \_\  \____| |_| |_| |_____| |_| \_\ |_____|
  @@@   @@     @@   @@@
   @@@    @@@@@    @@@       https://www.archer2.ac.uk/support-access/
     @@@         @@@
        @@@@@@@@@

 -         U K R I         -        E P C C        -         H P E   C r a y         -

Hostname:     uan01
Distribution: SLES 15.1 1
CPUS:         256
Memory:       257.4GB
Configured:   2021-04-27

######################################################################################
```

If you've connected successfully, you should see a prompt like the one below.
This prompt is informative, and lets you grasp certain information at a glance.
(If you don't understand what these things are, don't worry! We will cover
things in depth as we explore the system further.)

```bash
userid@ln03:~>
```

## Telling the Difference between the Local Terminal and the Remote Terminal

You may have noticed that the prompt changed when you logged into the remote
system using the terminal (if you logged in using PuTTY this will not apply
because it does not offer a local terminal). This change is important because
it makes it clear on which system the commands you type will be run when you
pass them into the terminal. This change is also a small complication that we
will need to navigate throughout the workshop. Exactly what is reported before
the `$` in the terminal when it is connected to the local system and the remote
system will typically be different for every user. We still need to indicate
which system we are entering commands on though so we will adopt the following
convention:

 - `[local]$` when the command is to be entered on a terminal connected to your
  local computer
 - `userid@ln03:~>` when the command is to be entered on a
  terminal connected to the remote system
 - `$` when it really doesn't matter which system the terminal is connected to.

:::::::::: callout
# Being certain which system your terminal is connected to

If you ever need to be certain which system a terminal you are using is
connected to then use the following command: `$ hostname`.
::::::::::

:::::::::: callout
## Keep two terminal windows open

It is strongly recommended that you have two terminals open, one connected to
the local system and one connected to the remote system, that you can switch
back and forth between. If you only use one terminal window then you will
need to reconnect to the remote system using one of the methods above when
you see a change from `[local]$` to userid@ln03:~> and
disconnect when you see the reverse.
::::::::::


:::::: keypoints

  - To connect to a remote HPC system using SSH and a password,
  run
  
```bash
ssh yourUsername@remote.computer.address
```

  - To connect to a remote HPC system using SSH and an SSH key,
  run 
  
```bash
ssh -i ~/.ssh/key_for_remote_computer yourUsername@remote.computer.address
```
::::::

---
title: "Connecting to the remote HPC system"
teaching: 25
exercises: 15
---



:::::: questions
  - How do I open a terminal?
  - What is an SSH key?
  - How do I connect to a remote computer?
::::::

:::::: objectives
  - Connect to a remote HPC system.
::::::


The first step in using a cluster is to establish a connection from our laptop
to the cluster. When we are sitting at a computer (or standing, or holding it
in our hands or on our wrists), we have come to expect a visual display with
icons, widgets, and perhaps some windows or applications: a graphical user
interface, or GUI. 

Since computer clusters are remote resources that we connect
to over often slow or laggy interfaces (WiFi and VPNs especially), it is more
practical to use a command-line interface, or CLI, in which commands and
results are transmitted via text, only. Anything other than text (images, for
example) are written to disk and opened with a separate program.

If you have ever opened a terminal, i.e. the Windows Command Prompt or macOS Terminal, you have
seen a CLI. If you have already taken The Carpentries' courses on the UNIX
Shell or Version Control, you have used the CLI on your local machine somewhat
extensively. 

## Opening a Terminal

Different operating systems have different terminals, none of which are exactly 
the same in terms of their features and abilities while working on the operating 
system. However, when connected to the remote system the experience between terminals 
will be identical as each will faithfully present the same experience of using that system.

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

#### (1) MobaXterm

MobaXterm is a terminal window emulator for Windows and the home edition can be
downloaded for free from 
[mobatek.net][mobatek].

If you follow the link you will note that there are two editions of the home version
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

#### (2) Git BASH

Git BASH gives you a terminal like interface in Windows. You can use this to
connect to a remote computer via SSH. It can be downloaded for free from
[here][gitbash].

#### (3) Windows Subsystem for Linux

The Windows Subsystem for Linux also allows you to connect to a remote computer
via SSH. Instructions on installing it can be found 
[here][wsl].

::: discussion
## How do I open a terminal?

Everyone should now be able to do this! We have completed the first step to using an
HPC system. 
:::


## Secure SHell protocol (SSH)

We can think of connecting to the remote HPC system as opening a terminal on a *remote*
machine. However, we also need to take some precautions so that other folks on the network can't
see (or change) the commands you're running or the results the remote machine
sends back. 

![Connect to cluster](fig/connect-to-remote.svg){alt="Connecting your laptop to a remote cluster"}

We will use the Secure SHell protocol (or SSH) to open an encrypted
network connection between two machines (your laptop/computer and the remote HPC system), 
allowing you to send & receive text and data without having to worry about prying eyes.

In this section, we will cover creating a pair of SSH keys: a private key which you keep on your
own computer and a public key which is placed on the remote HPC system that you will log in to.

These keys initiate a secure handshake between remote parties. The private vs public nomenclature 
can be confusing as they are both called keys. It is more helpful to think of the public 
key as a "lock" and the private key as the "key". You give the public 'lock' to 
remote parties to encrypt or 'lock' data. This data is then opened with the 'private' 
key which you hold in a secure place.

### Creating an SSH key

Make sure you have a SSH client installed on your laptop. SSH clients are
usually command-line tools, where you provide the remote machine address as the
only required argument. If your SSH client has a graphical front-end, such as 
PuTTY or MobaXterm, you will set these arguments before clicking "connect." From 
the terminal, you'll write something like `ssh userName@hostname`, where the "@" symbol is 
used to separate the two parts of the argument.

### Linux, Mac and Windows Subsystem for Linux

Once you have opened a terminal check for existing SSH keys and filenames. 
Existing SSH keys can be overwritten, so it is good to check this: 

```bash
$ ls ~/.ssh/
```

We can then then generate a new public-private key pair,

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

:::::::::::: challenge

#### Further information
 
For more information on SSH security and some of the
flags set here, an excellent resource is 
[Secure Secure Shell](https://stribika.github.io/2015/01/04/secure-secure-shell.html).

::::::::::::

### Windows

For other options when using Windows, see: 

 - puttygen, see the Putty [documentation](https://www.chiark.greenend.org.uk/~sgtatham/putty/docs.html)
 - MobaKeyGen, see the MobaXterm [documentation](https://mobaxterm.mobatek.net/documentation.html) 


### Uploading the public part of the key to the SAFE 

As part of the [setup](../index.md) you will have created an account on the SAFE. 

The SAFE is an online service management system used by EPCC. Through SAFE, individual 
users can request machine accounts, reset passwords, see available resources and track their 
usage. All users must be registered on SAFE before they can apply for their machine account, so this 
is the service you used to sign-up for an ARCHER2 account. 

Once you have generated your key pair, you need to add the public part to your ARCHER2 account in SAFE:

1. [Login to SAFE](https://safe.epcc.ed.ac.uk)
2. Go to the Menu “Login accounts” and select the ARCHER2 account you want to add the SSH key to
3. On the subsequent Login account details page click the “Add Credential” button
4. Select “SSH public key” as the Credential Type and click “Next”
5. Either copy and paste the public part of your SSH key into the “SSH Public key” box or use the button to select the public key file on your computer.
6. Click “Add” to associate the public SSH key part with your account

The public SSH key part will now be added to your login account on the ARCHER2 system, so it can actually be used as your
secure handshake between remote parties. Remember: you can think of the public 
key as a "lock" and the private key as the "key". You give the public 'lock' to 
remote parties to encrypt or 'lock' data. This data is then opened with the 'private' 
key which you hold in a secure place.

:::::::::::: callout

## PRIVATE KEYS ARE PRIVATE

  A private key that is visible to anyone but you should be considered compromised,
  and must be destroyed. This includes having improper permissions on the directory
  it (or a copy) is stored in, traversing any network in the clear, attachment on 
  unencrypted email, and even displaying the key (which is ASCII text) in your 
  terminal window.

  Protect this key as if it unlocks your front door. In many ways, it does.

::::::::::::




## Logging onto the system

With all of this in mind, let's finally connect to the remote HPC system. In this
workshop, we will connect to ARCHER2 --- an HPC system
located at the University of Edinburgh. Although it's unlikely that
every system will be exactly like ARCHER2, it's a very good
example of what you can expect from an HPC. 


### Configure TOTP passwords

ARCHER2 now uses Time-based One-Time Passwords (TOTP) for multi-factor authentication (MFA). 
One time passwords are a common security measure used by banking, cloud services and apps that 
create a changing time limited code to verify your identity beyond a password and username.

To setup your MFA TOTP you will need an authenticator application on your phone or laptop. 
Follow the steps at [the SAFE documentation](https://epcced.github.io/safe-docs/safe-for-users/#how-to-turn-on-mfa-on-your-machine-account), 
ensuring you create the code for your `ta215` project account.

You will only be prompted at login for your TOTP code once a day.


### Logging in to ARCHER2

We've already used SSH protocol to generate an SSH key-pair, now we will use SSH to connect
(if you are using PuTTY, see above). SSH allows us to connect to UNIX computers remotely, and use them as if they
were our own. The general syntax of the connection command follows the format: 

```bash
ssh -i ~/.ssh/key_for_remote_computer yourUsername@remote.computer.address
```

For the SSH key-pair we generated in the previous section, it will look like this: 

```bash
ssh -i ~/.ssh/key_ARCHER2_rsa yourUsername@login.archer2.ac.uk
```



If your SSH key pair is stored in the default location (usually ~/.ssh/id_rsa) on your local system, 
you may not need to specify the path to the private part of the key wih the -i option to ssh. 
For example: 

```bash
ssh yourUsername@ARCHER2
```

:::::::::::: callout

### First login! 

As an additional security measure, you will also need to use a password from SAFE for your 
first login to ARCHER2 with a new account. When you log into ARCHER2 for the first time with a 
new account, you will be prompted to change your initial password. This is a three step process:

1. When promoted to enter your ldap password: Enter the password which you retrieve from SAFE
2. When prompted to enter your new password: type in a new password
3. When prompted to re-enter the new password: re-enter the new password

Your password has now been changed. You will no longer need this password to log into ARCHER2 
from this point forwards, you will use your SSH key and TOTP code _only_.

::::::::::::


If you've connected successfully, you should see a prompt like the one below.
This prompt is informative, and lets you grasp certain information at a glance.
(If you don't understand what these things are, don't worry! We will cover
things in depth as we explore the system further.)



```output
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

Hostname:     ln02
Distribution: SLES 15.1 1
CPUS:         256
Memory:       515.3GB
Configured:   2025-09-16

######################################################################################
---------------------------------Welcome to ARCHER2-----------------------------------
######################################################################################
```




## Telling the Difference between the Local Terminal and the Remote Terminal


```bash
userid@ln03:~>
```

You may have noticed that the prompt changed when you logged into the remote
system using the terminal (if you logged in using PuTTY this will not apply
because it does not offer a local terminal). 

This change is important because
it makes it clear on which system the commands you type will be run when you
pass them into the terminal. This change is also a small complication that we
will need to navigate throughout the workshop. Exactly what is reported before
the `$` in the terminal when it is connected to the local system and the remote
system will typically be different for every user. 

We will using the following to differentiate:

 - `[local]$` when the command is to be entered on a terminal connected to your
  local computer
 - `userid@ln03:~>` when the command is to be entered on a
  terminal connected to the remote system
 - `$` when it really doesn't matter which system the terminal is connected to.

If you ever need to be certain which system a terminal you are using is
connected to then use the following command: `$ hostname`.

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
To connect to a remote HPC system using SSH, we would run the following command and enter 
an SSH key passphrase and machine password. (For ARCHER2, this is a TOTP) 
  
```bash
ssh -i ~/.ssh/key_for_remote_computer yourUsername@remote.computer.address
```
::::::

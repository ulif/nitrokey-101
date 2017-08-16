# nitrokey-101

Fun with the Nitrokey

The following describes how to do things with Nitrokey devices.


## The Nitrokey Pro


### Setting up a Virtual Machine for Testing

Setup a vagrant box.

You can use the accompanied ``Vagrantfile`` to setup two `vagrant` instances
running a `virtualbox`-based Ubuntu 16.04 'xenial' and a Ubuntu 14.04 'trusty'.

The respective instances are called ``xenial`` and ``trusty`` respectively.

Please note, that by default both instances are provisioned using `ansible`. If
you do not have `ansible` installed, you can do all stuff from `provision.yml`
manually.

To access any USB device in your virtual vagrant guests, you must (on the host,
not the guests) be member of the `vboxusers` group.

    $ sudo add <myusername> vboxusers

Afterwards you must relogin (and relogin the `vagrant` box, reloading is not
sufficient).

As a result you should be able to get:

    $ vagrant ssh
    ubuntu@ubuntu-xenial:~$ lsusb
    Bus 001 Device 002: ID 20a0:4108 Clay Logic
    Bus 001 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub

A device with ID ``20a0:4108`` must appear. The ID represents a `Nitrokey Pro`
device.

All other preparations should be done automatically by the local
`provision.yml` used by `ansible`.


### Setting up the virtualboxes manually

Then, inside the guest, install some packages we will need:

    (ubuntu@ubuntu-xenial):~$ sudo apt-get install libccid scdaemon gnupg2

Finally, copy over the accompanied `udev`-rules:

    (ubuntu@ubuntu-xenial):~$ sudo cp /vagrant/41-nitrokey.rules /etc/udev/rules.d

and back on the host, restart the vagrant machine.

    $ vagrant reload

Now, if you login to your vagrant box and plugin the Nitrokey, you should be
able to retrieve some data using `gpg`:

    (ubuntu@ubuntu-xenial):~$ gpg --card-status
    Application ID ...: D2760011240102010005100038E30000
    Version ..........: 2.1
    Manufacturer .....: ZeitControl
    Serial number ....: 000036E3
    Name of cardholder: [not set]
    Language prefs ...: de
    Sex ..............: unspecified
    URL of public key : [not set]
    Login data .......: [not set]
    Private DO 1 .....: [not set]
    Private DO 2 .....: [not set]
    Signature PIN ....: forced
    Key attributes ...: 2048R 2048R 2048R
    Max. PIN lengths .: 32 32 32
    PIN retry counter : 3 0 3
    Signature counter : 0
    Signature key ....: [none]
    Encryption key....: [none]
    Authentication key: [none]
    General key info..: [none]

If the output looks like the one above, then you are done.


### Generating Entropy in a Vagrant Box

If the generation of passwords takes very long time because of missing entropy,
you can follow the hints at:

  https://major.io/2007/07/01/check-available-entropy-in-linux/#comment-23651

but be aware that keys created on such a box are worthless. For testing and
playing around, however, it is okay and comes down to:

    $ sudo apt-get install rng-tools
    $ sudo bash
    # echo "HRNGDEVICE=/dev/urandom" >> /etc/default/rng-tools
    # exit
    $ sudo service rng-tools restart


## Resetting to Factory Defaults

If you terribly messed up your setup or just forgot your passwords, there is
still a possibility to reset your Nitrokey. The price, of course, is loss of
all data stored on the stick.

The different possibilitites are listed on

    https://www.nitrokey.com/documentation/how-reset-nitrokey

The easiest way for Linux users with a fairly recent `gpg` version (>= 2.1)
installed, is to run:

    $ gpg2 --card-edit

then, selecting ``"admin"`` followed by ``"factory-reset"``.


## Step #1: Change PINs

The Nitrokey Pro uses two PINs. A user PIN (default: `123456`) and an admin PIN
(default: `12345678`). You want to change these as the very first step:

    $ gpg2 --change-pin

and change the user PIN and after that the admin PIN.


## Step #2: Edit your Nitrokey Pro with GPG

Here are some basic steps to get used to the GPG environment.

Go into `gpg2` edit mode to edit attributes of the ``Nitrokey``. This will drop
you into a special shell.

    $ gpg2 --card-edit
    gpg/card>

With ``help`` you can get an overview of what is possible:

    gpg/card> help
    quit           quit this menu
    admin          show admin commands
    help           show this help
    list           list all available data
    fetch          fetch the key specified in the card URL
    passwd         menu to change or unblock the PIN
    verify         verify the PIN and list all data
    unblock        unblock the PIN using a Reset Code

If you enter "admin", there will be more possibilieties:

    gpg/card> admin
    Admin commands are allowed

    gpg/card> help
    quit           quit this menu
    admin          show admin commands
    help           show this help
    list           list all available data
    name           change card holder's name
    url            change URL to retrieve key
    fetch          fetch the key specified in the card URL
    login          change the login name
    lang           change the language preferences
    sex            change card holder's sex
    cafpr          change a CA fingerprint
    forcesig       toggle the signature force PIN flag
    generate       generate new keys
    passwd         menu to change or unblock the PIN
    verify         verify the PIN and list all data
    unblock        unblock the PIN using a Reset Code
    factory-reset  destroy all keys and data

To quit the shell, just enter "quit":

    gpg/card> quit


## Step #3: Create GPG Keys

Plug you Nitrokey Pro into the computer. Then start `gpg2`, go into admin mode
and generate the keys.

    $ gpg2 --card-edit
    gpg/card> admin
    gpg/card> generate

This can last very, very, long. Especially, if you are on a virtual box.

When asked, please create a backup of the key generated, as you will have use
for the public part of the key, which will otherwise not be available.


## Use Nitrokey Pro for SSH Login on Remote Servers

1) Create GPG keys as shown above. Make sure, that the public part of the key
   is already available locally. I.e.:

    $ gpg2 -k

    should list your key generated above, while

    $ gpg2 -K

    should fail.

2) Enable SSH support of gpg-agent in ~/.gnupg/gpg-agent.conf:

    enable-ssh-support

3) In "~/.profile" add

    export SSH_AUTH_SOCK=$HOME/.gnupg/S.gpg-agent.ssh

4) In "~/.bashrc" add

    GPG_TTY=$(/usr/bin/tty)
    SSH_AUTH_SOCK="$HOME/.gnupg/S.gpg-agent.ssh"
    export GPG_TTY SSH_AUTH_SOCK
    echo UPDATESTARTUPTTY | gpg-connect-agent
    # gpg-connect-agent reloadagent /bye

5) Get the "keygrip" of your Nitrokey authentication key:

    $ gpg2 --with-keygrip -k joe
    pub   rsa2048/AA069E8D 2017-05-13 [SC]
          Keygrip = C885218CE63BC64B64359F1A16AB685F831B716B
    uid         [ultimate] Joe Doe <joe@sample.org>
    sub   rsa2048/128FA065 2017-05-13 [A]
          Keygrip = CD4A1911AC15AC20785C8D2A0C1BF174A96CA6EC
    sub   rsa2048/4FD20E45 2017-05-13 [E]
          Keygrip = FBCCB48EA62FCF9E12378B827688AF643C3EEFAE

   Look for the (A)uthentication key and add the respective keygrip in
   ~/.gnupg/sshcontrol:

       # file ~/.gnupg/sshcontrol
       CD4A1911AC15AC20785C8D2A0C1BF174A96CA6EC

6) Extract the key in a form suitable to be used in ``authorized_keys``:

    $ ssh-add -L
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCWiPacBhi8q1kR2SaoQGWP9K+Grwc8UvI5q
    cLXrc6zfg3NHW2qKUb3gzvEEuSXkdhLoIrPQiP+p/lArGTHF6t4TaK9cL+LkaIq/gxNUspDyQ
    YJQaYIWQzVqjbln6rCRtAfuLMX29W7IbGOV1sdeLLw6YvO7iFrs/S1JAR8My3hE495Gb7lNN1
    lGYkXyysXdFOFP+OPfA6GppQE7TuEW/78NNrYY3p7QJg10MMnx1nV7Be3CDJSj6ZcerP7Wg0k
    RLTDOp+5CPfCxLUbapIKnkDEY7651Y1TPPd3xc6VGN4sU+Di5qSpsllQWelijoBS+Dc1CqmdT
    9j881xqyQmKNhYz cardno:000500002FE1

7) Add this key to your remote ``authorized_keys``



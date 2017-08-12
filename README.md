# nitrokey-101

Fun with the Nitrokey

The following describes how to do things with Nitrokey devices.


## The Nitrokey Pro


### Setting up a Virtual Machine for Testing

Setup a vagrant box.

You can use the accompanied ``Vagrantfile`` to setup a `vagrant` instance
running a `virtualbox`-based Ubuntu 16.04 'Xenial'.

To access any USB device in your virtual vagrant guest, you must (on the host,
not the guest) be member of the `vboxusers` group.

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

Then, inside the guest, install some packages we will need:

    (ubuntu@ubuntu-xenial):~$ sudo apt-get install libccid scdaemon

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

    $ gpg --change-pin

and change the user PIN and after that the admin PIN.

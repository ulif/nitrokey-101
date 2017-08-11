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


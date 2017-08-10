# nitrokey-101

Fun with the Nitrokey

The following describes how to do things with Nitrokey devices.


## The Nitrokey Pro


### Setting up a Virtual Machine for Testing

Setup a vagrant box.

You can use the accompanied ``Vagrantfile`` to setup a `vagrant` instance
running `virtualbox`-based Ubuntu-xenial.

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

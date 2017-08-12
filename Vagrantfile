# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "xenial", primary: true do |xenial|
    xenial.vm.box = "ubuntu/xenial64"
  end

  config.vm.define "trusty", autostart: false do |trusty|
    trusty.vm.box = "ubuntu/trusty64"
  end

  config.vm.provider "virtualbox" do |vb|
    vb.customize ['modifyvm', :id, '--usb', 'on']
    vb.customize ['usbfilter', 'add', '0', '--target', :id, '--name',
                  'SmartCard', '--vendorid', '0x20a0', '--productid', '0x4108']
    vb.gui = false
    vb.memory = "1024"
  end

end

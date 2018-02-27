# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/jessie64"
  config.vm.provider :lxc do |lxc|
    lxc.customize 'aa_profile', 'unconfined'
  end
end

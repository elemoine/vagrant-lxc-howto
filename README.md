# Vagrant LXC HowTo

This small Git repo shows how to set up Vagrant LXC on Debian Sid (unstable).

Farewell VirtualBox! (Hopefully)

## Install dependencies

Install LXC and the network bridge tools:

```shell
$ sudo apt install lxc bridge-utils
```

Install the Vagrant LXC plugin:

```shell
$ vagrant plugin install vagrant-lxc
```

## Configure LXC networking

On Debian Sid (and probably other Debian versions) LXC networking is not enabled by default. Some
manual configuration is required.

First, create `/etc/default/lxc-net` with the following content:

```
USE_LXC_BRIDGE="true"
```

Now edit `/etc/lxc/default.conf` and change the default:

```
lxc.network.type = empty
```

to

```
xc.network.type = veth
lxc.network.link = lxcbr0
lxc.network.flags = up
lxc.network.hwaddr = 00:16:3e:xx:xx:xx
```

Restart the `lxc-net` service for the above to be taken into account:

```shell
$ sudo systemctl restart lxc-net
```

## Go!

Now create this minimal `Vagrantfile`:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "debian/jessie64"
  config.vm.provider :lxc do |lxc|
    lxc.customize 'aa_profile', 'unconfined'
  end
end
```

And use `vagrant up` to create and start the LXC container:

```shell
$ vagrant up --provider=lxc
```

When `vagrant up` is finished you should be to SSH into the container:

```shell
$ vagrant ssh
```

Hooray!

## Misc

Stop the container:

```shell
$ vagrant halt
```

Check that the bridge was created:

```shell
$ sudo brctl show lxcbr0
```

Check that the container was created:

```shell
$ sudo lxc-ls
vagrant-lxc-howto_default_1519763511143_76070
$ sudo lxc-info -n vagrant-lxc-howto_default_1519763511143_76070
Name:           vagrant-lxc-howto_default_1519763511143_76070
State:          RUNNING
PID:            14488
IP:             10.0.3.35
CPU use:        1.73 seconds
BlkIO use:      48.00 KiB
Memory use:     24.46 MiB
KMem use:       3.61 MiB
Link:           vethGPA2BD
 TX bytes:      66.78 KiB
 RX bytes:      66.87 KiB
 Total bytes:   133.65 KiB
```

!--
Maintainer:   jeffskinnerbox@yahoo.com / www.jeffskinnerbox.me
Version:      0.0.7
-->


<div align="center">
<img src="http://www.foxbyrd.com/wp-content/uploads/2018/02/file-4.jpg" title="These materials require additional work and are not ready for general use." align="center">
</div>


----

* [Vagrant Box Templates](https://everythingshouldbevirtual.com/virtualization/vagrant-box-templates/)
* [How do I install a Vagrant Linux box that has a GUI (gnome, kde, …)?](https://stackoverflow.com/questions/31845472/how-do-i-install-a-vagrant-linux-box-that-has-a-gui-gnome-kde)
* [Using vagrant to run virtual machines with desktop environment](https://stackoverflow.com/questions/18878117/using-vagrant-to-run-virtual-machines-with-desktop-environment)
* [Virtualbox: install guest additions on Ubuntu 20.04 LTS Focal Fossa](https://linuxconfig.org/virtualbox-install-guest-additions-on-ubuntu-20-04-lts-focal-fossa)



# Vagrant Ubuntu GUI Desktop
Vagrant is great because it allows us to test and run software within a virtual environment,
using an [immutable infrastructure][08],
and meanwhile keep our hosts operating environment clean of
databases, libraries, and programming languages in different flavors and versions.


But what if we want to use Vagrant to run software that needs a GUI?
Specifically, what if your want to run in Ubuntu a X Windows GUI application in your host's window
(aka [X Forwarding][01])?
What if you want to run a full [Ubuntu desktop environment][02]?
An of course I would want to cut & paste across the host and guest systems,
access external devices via the hosts USB ports,
and share files between the host & guest virtual machine.
How can this be done?

The `Vagrantfile` in this repository is just such an implementation
and is a working example for [Ubuntu 16.04 using the Gnome Desktop][06].

Ultimately, I want to creating Ubuntu Desktop [Vagrant base box][07].
The implementation here isn't for a base box yet but it does have the Vagrantfile
from which a base box could be created.
Basically, this is the `Vagrantfile` to create a Vagrant virtual machine
with a full Ubuntu GUI Desktop,
along with X-Forwarding, cut & paste, USB port support,
and disk file sharing.

On top of this, I loaded up the virtual machine with all the basic development environment tools
I like to have on hand for any work I would do.

Sources:

* [How to install a Full Desktop (GUI) on Ubuntu Server](https://www.youtube.com/watch?v=rWyWt3DR9Fs)
* [Adding a GUI to a Debian Vagrant box](https://shanemcd.org/2018/12/16/adding-a-gui-to-a-debian-vagrant-box/)
* [Using vagrant to run virtual machines with desktop environment](https://stackoverflow.com/questions/18878117/using-vagrant-to-run-virtual-machines-with-desktop-environment)

## Warning
I had a wide range of problems
(with folder synchronization, cut & paste, and other services)
This appears to happen when you [upgrade Vagrant or even change boxes][10]
when my version of VirtualBox and its Guest Additions where not at the same level.
Vagrant often doesn't like this and will result in things not working properly.
If features are not working for you, check your VirtualBox and its Guest Additions version level.

You can detect this inconsistency via the following commands:

```bash
# run on host: check on the status of virtualbox guest additions
$ vagrant vbguest --status
Got different reports about installed GuestAdditions version:
Virtualbox on your host claims:   6.0.14
VBoxService inside the vm claims: 5.1.38
Going on, assuming VBoxService is correct...
[default] GuestAdditions versions on your host (6.0.14) and guest (5.1.38) do not match.

# also got the following error at times
$ vagrant vbguest --status
[default] A Virtualbox Guest Additions installation was found but no tools to rebuild or start them.
```

Another potential way to spot inconsistencies is the following:

```bash
# run on host: more direct way to check virtualbox version
$ virtualbox --help | head -n 1 | awk '{print $NF}'
v6.0.14_Ubuntu

# run on host: more direct way to check virtualbox guest additions version
$ modinfo vboxguest | grep ^version | awk '{ print $2 }'
5.1.38_Ubuntu
```

As you can see, there is a version level difference.
To fix this, you must bring the VirtualBox Guest Addition up to the same version level as VirtualBox.
You can accomplish this with the following:

```bash
# run on host: update virtualbox guest addition
vagrant vbguest

# run on host: check on the status of virtualbox guest additions
$ vagrant vbguest --status
[default] GuestAdditions 6.0.14 running --- OK.
```

Rebuild your VM (i.e. `vagrant destroy`, followed by `vagrant up`).
With this, hopefully you get a working VM.


----


# Key Supporting Features
The section below covers some the key features within the Vagrantfile
used to create this Ubuntu Desktop virtual machine.

### GNOME Desktop
I was hoping to install the basic Gnome desktop environment, also known as the Vanilla desktop,
you could do the following, but I [repeatedly ran into "purple screen" problem][12]:

```bash
# install vanilla GNOME desktop
sudo apt-get install gnome-session

# use the gdm3 login screen of Gnome desktop
sudo update-alternatives --set gdm3.css /usr/share/gnome-shell/theme/Yaru/gnome-shell.css
```

So instead, I install the full GNOME desktop but did not use the [`tasksel`][11] command.
In stead, I installed the desktop using `apt-get` and avoid the "purple screen"
for guest virtual machine with ubuntu 18.04 but not ubuntu 19.10:

```bash
# install full version of GNOME desktop
sudo apt-get install -y ubuntu-desktop
```

Sources

* [Ubuntu Linux install Gnome desktop on server](https://www.cyberciti.biz/faq/ubuntu-linux-install-gnome-desktop-on-server/)
* [How to install Gnome on Ubuntu 20.04 LTS Focal Fossa](https://linuxconfig.org/how-to-install-gnome-on-ubuntu-20-04-lts-focal-fossa)
* [How to install Vanilla Gnome Desktop on Ubuntu](https://vitux.com/how-to-install-vanilla-gnome-desktop-on-ubuntu/)

### Vagrant Ubuntu X Forwarding
The quickest and simplest thing to do to get a X Windows graphics program running
is to use your Vagrant host as your [X Server][03] and let the Vagrant guest run the [X Client][04].
Thanks to [X Forwarding][01], we can forward the output of graphical programs running
on the guest virtual machine to your host machine if you're running X11 on the host.
You simply need to add `config.ssh.forward_x11 = true` to your `Vagrantfile`.

>**NOTE:** Before enabling X11 forwarding on the Vagrantfile,
>you may need to install the utility [`xauth`][05].
>This program is usually used to extract authorization records from one machine
>and merge them in on another.
>Generally, `xauth` is installed, but if needed, use `sudo apt-get install xauth`.

To use X-Forwarding,
you first need to activate it from within your `Vagrantfile`, like this:

```ruby
Vagrant.configure(2) do |config|
  ...
  # X forwarding support, using port 2222
  config.ssh.forward_agent = true        # if true, agent forwarding over SSH connections is enabled
  config.ssh.forward_x11 = true          # if true, X11 forwarding over SSH connections is enabled
end
```

Use `vagrant up` and then check if X Forwarding has been activated.
During your initial `vagrant ssh`,
you may get the message  `/home/vagrant/.Xauthority` does not exist.
This is the "MIT magic cookie" error.
This is okay since we’re running `xauth` for the first time and
the `xauth` program will create `.Xauthority` file for you.

```bash
# on the vagrant host, bring up the vagrant guest
$ vagrant ssh
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-179-generic x86_64)
  .
  .
  .
/usr/bin/xauth:  file /home/vagrant/.Xauthority does not exist

# on the vagrant host, confirm the ssh configuration on the guest
$ vagrant ssh-config
Host default
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/jeff/src/vagrant-machines/jetson-dev/.vagrant/machines/default/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL
  ForwardAgent yes
  ForwardX11 yes
```

So we can now use port 2222 to connect to the guest machine,
using the `ssh` keys listed by `vagrant ssh-config`,
and run a X11 program on the Vagrant guest but displayed on the Vagrant host,
as shown below:

```bash
# make sure your in the relavent directory
cd ~/src/vagrant-machines/jetson-dev

# login to the guest VM via ssh with x forwarding
ssh vagrant@localhost -X -p 2222 -i ./.vagrant/machines/default/virtualbox/private_key

# on the vagrant guest, execute a test program
xeyes
```

Sources:

* [Run graphical programs within Vagrantboxes](https://coderwall.com/p/ozhfva/run-graphical-programs-within-vagrantboxes)
* [X-forwarding to run GUI program in Vagrant box](https://code-maven.com/xforwarding-from-vagrant-box)
* [How to enable and use SSH X11 Forwarding on Vagrant Instances](https://computingforgeeks.com/how-to-enable-and-use-ssh-x11-forwarding-on-vagrant-instances/)

### Enabling Cut & Paste / Clipboard
Your going to want to enable cut & paste between the host and guest machines.
This can be easily be accomplished via the following:

```ruby
config.vm.provider "virtualbox" do |vb|
   .
   .
   .
  # clipboard, drag & drop, notifications support
  vb.customize ["modifyvm", :id, "--clipboard", "bidirectional"]           # clipboard shared between host / guest
  vb.customize ["modifyvm", :id, "--draganddrop", "bidirectional"]         # drag & drop between host / guest
   .
   .
   .
end
```

### Startup GUI Window Size
I find the default startup GUI window size poor and want to customize to my taste.
In my case, I would like it to be 3/4 of the size of my display monitor.

To get my monitor size, I used this:

```bash
# getting screen resolution
$ xdpyinfo | grep dimensions
  dimensions:    1920x1200 pixels (508x317 millimeters)
```

With this information, set the size of the GUI in the Vagrantfile:

```ruby
config.vm.provider "virtualbox" do |vb|
   .
   .
   .
  # startup gui screen resolution (3/4 monitor size)
  vb.customize ['setextradata', :id, 'GUI/LastGuestSizeHint','1440,900']
   .
   .
   .
end
```

Source:

* [Fixing a guest screen resolution in VirtualBox](https://superuser.com/questions/301464/fixing-a-guest-screen-resolution-in-virtualbox)

### Connect to USB Device Through Vagrant
Sometimes you need to connect a board or device to the
Vagrant guest virtual machine via the hosts USB port.
USB devices won't work in your guest until you can see them
listed via the command `lsusb` run on the guest machine.
To get them listed, you'll need to do two things:

1. Make sure you have the [VirtualBox Extension Pack is installed][09]
so you can use the utility `VBoxManage`.
2. Plug into a USB port the device you wish to use with the guest machine
and run the utility `VBoxManage` as shown below:

```bash
# get a listing of the usb devices
$ VBoxManage list usbhost
Host USB Devices:
   .
   .
   .
UUID:               bd1b84d2-3737-4c6e-aedd-7d48706d5754
VendorId:           0x046d (046D)
ProductId:          0x0994 (0994)
Revision:           0.7 (0007)
Port:               3
USB version/speed:  2/High
Manufacturer:       Logitech, Inc.
Product:            QuickCam Orbit/Sphere AF
SerialNumber:       C0CE6210
Address:            sysfs:/sys/devices/pci0000:00/0000:00:14.0/usb3/3-1/3-1.1/3-1.1.4//device:/dev/vboxusb/003/051
Current State:      Captured
   .
   .
   .
```

```ruby
# virtual hardware configuration
config.vm.provider "virtualbox" do |vb|
   .
   .
   .
    # enable USB and add a filter based on the desired device manufacturer / product
    vb.customize ["modifyvm", :id, "--usbehci", "on"]       # usb 2.0 (EHCI) controller
    vb.customize ["modifyvm", :id, "--usbxhci", "on"]       # usb 3.0 (xHCI) controller
    vb.customize ['usbfilter', 'add', '0', '--target', :id,
        '--name', 'QuickCam Orbit/Sphere AF',
        '--vendorid', '0x046d',
        '--productid', '0x0994']
   .
   .
   .
end
```

With the above in your Vagrantfile,
starting your virtual machine see if you can detect the device:

```bash
# start the guess virtual machine
vagrant up

# on the guest, check for the device
$ lsusb
Bus 001 Device 002: ID 046d:0994 Logitech, Inc. QuickCam Orbit/Sphere AF
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub

# use cheese to access camera (didn't work ... old camera)
cheese -d /dev/bus/usb/001/001
```

Sources:

* [How To Pass Through USB Devices To Guests On An Ubuntu 8.10 Host](https://www.howtoforge.com/virtualbox-2-how-to-pass-through-usb-devices-to-guests-on-an-ubuntu-8.10-host)
* [Connect USB From Virtual Machine Using Vagrant and Virtual Box](https://sonnguyen.ws/connect-usb-from-virtual-machine-using-vagrant-and-virtualbox/)
* [Connect a USB device through Vagrant](http://code-chronicle.blogspot.com/2014/08/connect-usb-device-through-vagrant.html)

### Synchronizing Folders/Files Across Guest & Host
Vagrant supports synchronizing folders/files between the host and guest machines.
This will allow you to continue working on your project's files on your host machine,
but also use the resources in the guest machine to edit, compile, or execute your project.

There are different types of file synchronization that Vagrant can choose from,
including the `rsync` type, which is one-way sync from host to guest.
To make my folders sync in two-way, I had to add an explicit definition in my Vagrantfile,
that is `type: "virtualbox"`:

```ruby
Vagrant.configure(2) do |config|
  ...
  # share host and guest folder /files
  config.vm.synced_folder "shared-data/", "/home/vagrant/shared-data", create: true,  type: "virtualbox"
end
```

>**NOTE:** By default, Vagrant will share your host's project directory
>(i.e. The directory containing the `Vagrantfile`) to `/vagrant` on the guest virtual machine.)

>**NOTE:** Vagrant can't share symbolically linked files.

Sources:

* [Synced Folders](https://www.vagrantup.com/docs/synced-folders/)
* [How to enable two-way folder sync in Vagrant with VirtualBox?](https://stackoverflow.com/questions/34448717/how-to-enable-two-way-folder-sync-in-vagrant-with-virtualbox)

### Set Desired Disk Size
The [`vagrant-disksize` plugin][13] can help you configure the disk storage your application needs.
Default disk storage is 10G.
You can install the plugin with `vagrant plugin install vagrant-disksize`.

To request the disk size of your choose, use the following in your Vagrantfile:

```ruby
Vagrant.configure('2') do |config|
  config.vm.box = "ubuntu/xenial64"          # ubuntu 16.04 guest vm
  config.disksize.size = "30GB"              # set the desired disk sized
end
```

Sources:

* [How to increase disk size on a Vagrant VM using VirtualBox?](https://medium.com/@thucnc/how-to-increase-disk-size-on-a-vagrant-vm-using-virtualbox-c3d24acee3f4)
* [Resize Hard Disk of a Vagrant box](https://nmrony.info/change-disk-size-of-a-vagrant-box/)
* [vagrant-disksize](https://github.com/sprotheroe/vagrant-disksize)


### Private Network
```ruby
Vagrant.configure(2) do |config|
  ...
 # Create a private network, which allows host-only access to the machine
  #config.vm.network "private_network", type: "dhcp"           # DHCP assigned ip address
  config.vm.network "private_network", ip: "192.168.10.222"   # static ip addess
end
```

### Custom Login and Development Environment
Within the Vagrantfile's provisioning,
I have include steps to create my login's working environment,
X Windows, C, and Python development environment.
Others will surely want to include their tools instead of mine here.
Never the less, the Vagrantfile shows you how I choose to personalize my working environment.


-----


# Tools Used to Create Base Box

## Packer (Not Needed)
The creation of a Vagrant box starts with the creation of VM using virtualization tool like VirtualBox.
This is often a manual process, using a GUI or CLI,
and is very different for all the virtualization tools on the market (e.g. VitrualBox, VMWare, etc.).
Unfortunately, this doesn't fit the modern paradigm of [infrastructure as code][14].

To overcome this, the creates of Vagrant, [HasiCorp][15], offer a tool called [Packer][16].
[Why Use Packer][20]?
Packer embraces modern configuration management automates the creation of any type of machine image.
Packer is an open source tool for creating identical 'machine images'
for multiple virtualization tools from a single source configuration.
Packer runs on every major operating system, and creates machine images for multiple platforms in parallel.
Packer does not replace [configuration management][17] tools like Ansible, Chef, or Puppet.
In fact, when building images,
Packer is able to use configuration management tools to install software onto the image.
Packer lets you build Virtual Machine images for different providers from one JSON file.
You can use the same file and commands to build an image on AWS, Digital Ocean VirtualBox and Vagrant.
This makes it possible to use exactly the same system for development which you then create in production.


----


# Building Vagrant Ubuntu GUI Desktop Base Box
Vagrant base boxes are prepackaged environments that are the foundation of Vagrant
and you typically find them at place like [Vagrant Cloud][18].
In most cases, this is usually just a stripped and naked operating system such as Ubuntu.
My mission here is to take just such a base box, add what is need to make it my
Ubuntu GUI Desktop envirnment, and covert this into a new base box for my use.

Sources:

* [Creating a Base Box](https://www.vagrantup.com/docs/virtualbox/boxes.html)
* [How to Create a Vagrant Base Box from an Existing One](https://scotch.io/tutorials/how-to-create-a-vagrant-base-box-from-an-existing-one)
* [Create a Vagrant Base Box (VirtualBox)](https://oracle-base.com/articles/vm/create-a-vagrant-base-box-virtualbox)
* [Building Custom Vagrant box](https://medium.com/@gajbhiyedeepanshu/building-custom-vagrant-box-e6a846b6baca)

>**NOTE:** The task of our base box is made much easier since we are using Vagrant to start with.
>The creation of a Vagrant box starts with the creation of VM using virtualization tool like VirtualBox.
>This is often a manual process, using a GUI or CLI,
>and is very different for all the virtualization tools on the market (e.g. VitrualBox, VMWare, etc.).
>Unfortunately, this doesn't fit the modern paradigm of [infrastructure as code][14].
>
>To overcome this, the creates of Vagrant, [HasiCorp][15], offer a tool called [Packer][16].
>[Why Use Packer][20]?
>Packer embraces modern configuration management automates the creation of any type of machine image.
>Packer is an open source tool for creating identical 'machine images'
>for multiple virtualization tools from a single source configuration.
>Packer runs on every major operating system, and creates machine images for multiple platforms in parallel.
>Packer does not replace [configuration management][17] tools like Ansible, Chef, or Puppet.
>In fact, when building images,
>Packer is able to use configuration management tools to install software onto the image.
>Packer lets you build Virtual Machine images for different providers from one JSON file.
>You can use the same file and commands to build an image on AWS, Digital Ocean VirtualBox and Vagrant.
>This makes it possible to use exactly the same system for development which you then create in production.

## Step 1: Build You Box Using Vagrant
First step is to create your box destine to become your new base box via Vagrant.
The Vagrantfile used to create it is in this repository.

```ruby
# build the box
vagrant up
```

We now SSH into the box and do any customizing required of it.
We're also going to clean up disk space on the VM
so when we package it into a new Vagrant box, it's as clean as possible.

```bash
# do any required customization

# remove APT cache
sudo apt-get clean

# "zero out" the drive (this is for Ubuntu)
sudo dd if=/dev/zero of=/EMPTY bs=1M
sudo rm -f /EMPTY

# clear the bash history
cat /dev/null > ~/.bash_history && history -c

# shutdown the vm
sudo shutdown -h now
```

## Step 2: Repackage the VM into a New Vagrant Base Box
We are going to repackage the server we just created into a new Vagrant Base Box.

```bash
# create the new box
vagrant package --output ubuntu-desktop.box

# install the vagrant box in your local repository
vagrant box add --name ubuntu-desktop ./ubuntu-desktop.box

# check to see the box is in the local repository
vagrant box list

# remove the built box now that its in the repository
rm ubuntu-desktop.box

# your vm is nolong needed
vagrant destroy
```

Now you have the box and you can use it like any other box
by referencing it in a `Vagrantfile` for a new build.

If you wise to remove the box from the local repository,
use the command `vagrant box remove ubuntu-desktop`.

## Step 3: Test the Build
Now lets test if the newly created Vagrant box in fact works.
You can login into the VM using “vagrant” as user name and “vagrant” as a password,
but first we need to initialize our test environment:

```bash
# create your test environment
mkcd ~/tmp/test-ubuntu-desktop

# initialize the vagrant environment
vagrant init ubuntu-desktop

# bring up the vm (first issues will take long time, in typical Microsoft fashion)
vagrant up
```



[01]:http://www.tldp.org/HOWTO/XDMCP-HOWTO/ssh.html
[02]:https://linuxconfig.org/8-best-ubuntu-desktop-environments-18-04-bionic-beaver-linux
[03]:http://www.linfo.org/x_server.html
[04]:http://www.linfo.org/x_client.html
[05]:http://users.stat.umn.edu/~geyer/secure.html
[06]:https://ubuntu.com/download/desktop
[07]:https://www.vagrantup.com/docs/boxes/base.html
[08]:https://rollout.io/blog/immutable-deployments/
[09]:https://askubuntu.com/questions/661414/how-to-install-virtualbox-extension-pack
[10]:https://kvz.io/vagrant-tip-keep-virtualbox-guest-additions-in-sync.html
[11]:http://manpages.ubuntu.com/manpages/focal/man8/tasksel.8.html
[12]:https://askubuntu.com/questions/1114069/ubuntu-64-bit-stuck-on-the-purple-loading-screen-on-vm
[13]:https://github.com/sprotheroe/vagrant-disksize
[14]:https://en.wikipedia.org/wiki/Infrastructure_as_code
[15]:https://www.hashicorp.com/
[16]:https://www.packer.io/
[17]:https://www.ansible.com/use-cases/configuration-management
[18]:https://app.vagrantup.com/boxes/search
[19]:https://packer.io/downloads.html
[20]:https://www.packer.io/intro/why.html

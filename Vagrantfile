# -*- mode: ruby -*-
# vi: set ft=ruby :

# Maintainer:   jeffskinnerbox@yahoo.com / www.jeffskinnerbox.me
# Version:      0.0.5


# Vagrantfile API/syntax version.  The "2" is the Vagrant configuration version.
VAGRANTFILE_API_VERSION = "2"

# All Vagrant configuration is done below.
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Every Vagrant development environment requires a box
  # You can search for boxes at https://vagrantcloud.com/search
  #config.vm.box = "ubuntu/xenial64"          # ubuntu 16.04 guest vm
  #config.vm.box = "ubuntu/eoan64"            # ubuntu 19.10 guest vm
  config.vm.box = "ubuntu/bionic64"          # ubuntu 18.04 guest vm
  config.vm.hostname = "ubuntu-desktop.vm"   # set hostname

  # set the disk size to be allocated
  config.disksize.size = "15GB"

  # X forwarding support, using port 2222
  config.ssh.forward_agent = true        # if true, agent forwarding over SSH connections is enabled
  config.ssh.forward_x11 = true          # if true, X11 forwarding over SSH connections is enabled

  # set auto_update to false, if you do NOT want to check the correct
  # guest additions version when booting this machine
  if Vagrant.has_plugin?("vagrant-vbguest") then
        config.vbguest.auto_update = false
  end

  # time in seconds that vagrant will wait for the machine to boot and be accessible, default 300 sec
  config.vm.boot_timeout = 300

  # If set to 'false', boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = true

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  #config.vm.network "forwarded_port", guest: 80, host: 8080       # port for web server

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  #config.vm.network "forwarded_port", guest: 8888, host: 8888, host_ip: "127.0.0.1" # port for jupyter notebook

 # Create a private network, which allows host-only access to the machine
  #config.vm.network "private_network", type: "dhcp"           # DHCP assigned ip address
  config.vm.network "private_network", ip: "192.168.10.222"   # static ip addess


  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on your network.
  #$def_net = `ip route | grep -E "^default" | awk '{printf "%s", $5; exit 0}'`   # default network interface
  #config.vm.network "public_network", bridge: "#$def_net", ip: "192.168.1.218"   # static ip addess
  config.vm.network "public_network"                                             # DHCP assigned ip address

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  #config.vm.synced_folder "shared-data/", "/home/vagrant/shared-data", create: true, type: "virtualbox", :mount_options => ["dmode=777", "fmode=666"]

  # customize the configuration of VM - see https://www.virtualbox.org/manual/ch08.html
  # virtual hardware configuration
  config.vm.provider "virtualbox" do |vb|
    vb.gui = true                                                            # true = GUI, false = headless
    vb.customize ["modifyvm", :id, "--cpus", 2]                              # number of cpu used
    vb.customize ["modifyvm", :id, "--memory", 4096]                         # memory used
    vb.customize ["modifyvm", :id, "--vram", 128]                            # add video ram for gui

    # clipboard, drag & drop, notifications support
    vb.customize ["modifyvm", :id, "--clipboard", "bidirectional"]           # clipboard shared between host / guest
    vb.customize ["modifyvm", :id, "--draganddrop", "bidirectional"]         # drag & drop between host / guest
    #
    # disable notification mouse/keyboard capture
    vb.customize ["setextradata", "global", "GUI/SuppressMessages", "all"]

    # startup gui screen resolution (3/4 monitor size)
    vb.customize ['setextradata', :id, 'GUI/LastGuestSizeHint','1440,900']

    # enable USB and add a filter based on the desired device manufacturer / product
    #vb.customize ["modifyvm", :id, "--usb", "on"]           # usb 1.1 (CHCI) controller
    vb.customize ["modifyvm", :id, "--usbehci", "on"]       # usb 2.0 (EHCI) controller
    vb.customize ["modifyvm", :id, "--usbxhci", "on"]       # usb 3.0 (xHCI) controller
    vb.customize ['usbfilter', 'add', '0', '--target', :id,
        '--name', 'QuickCam Orbit/Sphere AF',
        '--vendorid', '0x046d',
        '--productid', '0x0994']
    vb.customize ['usbfilter', 'add', '0', '--target', :id,
        '--name', 'Adafruit Console Cable',
        '--vendorid', '0x10c4',
        '--productid', '0xea60']
  end



  # ----------------------------------------------------------------------------
  # Linux System Environment
  # ----------------------------------------------------------------------------

  # --------------------- update linux packages (as root) ----------------------
  config.vm.provision "shell", name: "update linux packages (as root)", run: "always",
     inline: "apt-get -y update && apt-get -y dist-upgrade"

  # ------------------ create your system environment (as root) ----------------
  config.vm.provision "shell", name: "create your system environment (as root)", run: "once", inline: <<-SHELL
    # install NTP and set your time zone
    apt-get -y install ntp
    timedatectl set-timezone America/New_York

    # load firewall tool and utilities used by your bash shell
    apt-get -y install ufw

    # software version control tools (need to load your tools)
    apt-get -y install git
SHELL

  # ------------ create your user GUI desktop environment (as root) ------------
  config.vm.provision "shell", name: "create your user desktop environment (as root)", run: "once", inline: <<-SHELL
    # install virtualbox guest tools giving you healthy screen resolution, integrated mouse, etc.
    apt-get -y install build-essential virtualbox-guest-utils virtualbox-guest-x11 virtualbox-guest-dkms

#    # install full desktop (login: vagrant / password: vagrant)
    #add-apt-repository universe
    #add-apt-repository multiverse
    #apt-get -y install ubuntu-desktop
    ##apt-get install unity

    # install full version of GNOME desktop (login: vagrant / password: vagrant)
    apt-get -y install ubuntu-desktop

    # make sure to do 'shutdown -r now' as the last step, so this takes effect
SHELL



  # ----------------------------------------------------------------------------
  # Create Your Login Environment
  # ----------------------------------------------------------------------------

  # ---------------- create your working environment (as vagrant) --------------
  config.vm.provision "shell", name: "create your working environment (as vagrant)", run: "once", inline: <<-SHELL
    # become the vagrant user
    su --login vagrant --shell /bin/bash <<'EOF'

    # setup your bash environment
    rm ~/.bashrc ~/.bash_logout
    git clone https://github.com/jeffskinnerbox/.bash.git ~/.bash
    ln -s ~/.bash/inputrc ~/.inputrc
    ln -s ~/.bash/bashrc ~/.bashrc
    ln -s ~/.bash/bash_login ~/.bash_login
    ln -s ~/.bash/bash_logout ~/.bash_logout
    ln -s ~/.bash/bash_profile ~/.bash_profile
    ln -s ~/.bash/dircolors.old ~/.dircolors

    # source/load the changes into your profile
    source ~/.bashrc

    # setup your vim environment
    git clone http://github.com/jeffskinnerbox/.vim.git ~/.vim
    ln -s ~/.vim/vimrc ~/.vimrc
    mkdir ~/.vim/backup
    mkdir ~/.vim/tmp
    cd ~/.vim
    git submodule init
    git submodule update
    cd ~

    # setup your x windows environment
    git clone http://github.com/jeffskinnerbox/.X.git ~/.X
    ln -s ~/.X/xbindkeysrc ~/.xbindkeysrc
    ln -s ~/.X/Xresources ~/.Xresources
    ln -s ~/.X/xsessionrc ~/.xsessionrc

    # rebuilding $HOME/.Xauthority to avoid MIT magic cookie error
    touch ~/.Xauthority
    xauth generate :0 . trusted
    xauth add ${HOST}:0 . `xxd -l 16 -p /dev/urandom`
EOF
SHELL



  # ----------------------------------------------------------------------------
  # X Windows and C Development Environment
  # ----------------------------------------------------------------------------

  # --------------- install your development libraries (as root) ---------------
  config.vm.provision "shell", name: "install your development libraries (as root)", run: "once", inline: <<-SHELL
    # install needed compilers
    apt-get -y install build-essential cmake unzip pkg-config
    apt-get -y install gcc-6 g++-6

    # install X windows libraries and OpenGL libraries
    apt-get -y install libxmu-dev libxi-dev libglu1-mesa libglu1-mesa-dev

    # install optimization algorithums libraries
    apt-get -y install libopenblas-dev libatlas-base-dev liblapack-dev gfortran

    # HDF5 library for working with large datasets
    apt-get -y install libhdf5-serial-dev

    # install image and video libraries
    apt-get -y install libjpeg-dev libpng-dev libtiff-dev
    apt-get -y install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
    apt-get -y install libxvidcore-dev libx264-dev
SHELL

  # ----------------- install your development tools (as root) -----------------
  config.vm.provision "shell", name: "install your development tools (as root)", run: "once", inline: <<-SHELL
    # general development tools
    apt-get -y install gnome-terminal jq markdown vim screen

    # software version control tools
    apt-get -y install git git-lfs

    # secure hash algorithms (SHA) tools, specifically SHA256
    apt-get -y install hashalot

    # install some X Window utilities
    apt-get -y install x11-apps x11-xserver-utils xterm wmctrl

    # tools for viewing and manipulating image & video files
    apt-get -y install imagemagick feh mplayer

    # load your favorate browser
    echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list
    wget -P /home/vagrant/.ssh/ https://dl.google.com/linux/linux_signing_key.pub
    apt-key add /home/vagrant/.ssh/linux_signing_key.pub
    apt-get -y update
    apt-get -y install google-chrome-stable
SHELL



  # ----------------------------------------------------------------------------
  # Python Development Environment
  # ----------------------------------------------------------------------------

  # ------------- install your python tools and libraries (as root) ------------
  config.vm.provision "shell", name: "install your python tools and libraries (as root)", run: "once", inline: <<-SHELL
    # python 3 development libraries including TK and GTK GUI support
    apt-get -y install python3-dev python3-tk python-imaging-tk
    apt-get -y install libgtk-3-dev libboost-all-dev
    apt-get -y install build-essential cmake
SHELL

  # ----------- install your python tools and libraries (as vagrant) -----------
  config.vm.provision "shell", name: "create your python working environment (as vagrant)", run: "once", inline: <<-SHELL
    # become the vagrant user and load profile
    su --login vagrant --shell /bin/bash <<'EOF'

    # by default, non-interactive bash executions do not load ~/.bashrc
    source ~/.bashrc

    # install pip
    wget https://bootstrap.pypa.io/get-pip.py
    python3 get-pip.py
    rm get-pip.py
    export PATH="$HOME/.local/bin:$PATH"                # also in .bashrc but needed here

    # install virtual environment tools
    pip3 install virtualenv virtualenvwrapper

    # setup environment for virtualenv and virtualenvwrapper
    mkdir ~/.virtualenvs
    export WORKON_HOME=\$HOME/.virtualenvs              # also in .bashrc but needed here
    export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3    # also in .bashrc but needed here
    source $HOME/.local/bin/virtualenvwrapper.sh        # also in .bashrc but needed here

    # do pyenv install and setup environment
    git clone https://github.com/pyenv/pyenv.git ~/.pyenv
    echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc     # already in my .bashrc but here just in case
    echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc  # already in my .bashrc but here just in case

    # do pyenv intitalize
    $HOME/.pyenv/bin/pyenv install 3.8.3                # install python 3.8.3 (ubuntu 16.04 comes with 3.5.0)
    $HOME/.pyenv/bin/pyenv rehash                       # to assure the pyenv shims are updated
EOF
SHELL



  # ----------------------------------------------------------------------------
  # Restart Linux OS
  # ----------------------------------------------------------------------------

  # ------------- reboot to make sure everything is set (as root) --------------
  config.vm.provision "shell", name: "reboot to make sure everything is setup (as root)", run: "once", inline: <<-SHELL
    shutdown -r now
SHELL

end

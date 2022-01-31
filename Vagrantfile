# -*- mode: ruby -*-
# vi: set ft=ruby :

VM_MEMORY=8192
VM_CORES=2

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "bento/ubuntu-20.04"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  config.vm.provider :vmware_fusion do |v, override|
    v.vmx['memsize'] = VM_MEMORY
    v.vmx['numvcpus'] = VM_CORES
  end

  config.vm.provider :virtualbox do |v, override|
    v.memory = VM_MEMORY
    v.cpus = VM_CORES
    v.gui = true

    required_plugins = %w( vagrant-vbguest )
    required_plugins.each do |plugin|
      system "vagrant plugin install #{plugin}" unless Vagrant.has_plugin? plugin
    end
  end

  config.vm.provider :parallels do |v, override|
    v.memory = VM_MEMORY
    v.cpus = VM_CORES

    v.update_guest_tools = true

    # required_plugins = %w( vagrant-vbguest )
    # required_plugins.each do |plugin|
    #   system "vagrant plugin install #{plugin}" unless Vagrant.has_plugin? plugin
    # end
  end

  config.vm.provision 'shell' do |s|
    s.inline = 'echo Setting up machine name'

    config.vm.provider :vmware_fusion do |v, override|
      v.vmx['displayname'] = "Xv6 Dev"
    end

    config.vm.provider :virtualbox do |v, override|
      v.name = "Xv6 Dev"
    end

    config.vm.provider "parallels" do |prl|
      prl.name = "Xv6 Dev"
    end
  end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL

  $script = <<-SHELL
  export LANG=C.UTF-8
  export LC_ALL=C
  export DEBIAN_FRONTEND=noninteractive
  apt-get -q update
  apt-get purge -q -y snapd lxcfs lxd ubuntu-core-launcher snap-confine
  apt-get -q -y install build-essential libncurses5-dev zlib1g-dev \
          libssl-dev liblzo2-2 liblzo2-dev \
          git make cmake autoconf automake libtool pkg-config \
          unzip ca-certificates curl wget \
          apt-transport-https software-properties-common
  apt-get -q -y install --no-install-recommends python3 python3-dev \
          python3-pip python3-setuptools
  apt-get -q -y install python-minimal
  apt-get -q -y install --no-install-recommends tmux mg vim silversearcher-ag
  apt-get -q -y install --no-install-recommends netcat rsync \
          gzip bzip2 zip unzip fakeroot parted gdisk e2fsprogs
  apt-get -q -y install --no-install-recommends \
          gcc-aarch64-linux-gnu debootstrap \
          qemu-user-static qemu-system-aarch64 binfmt-support \
          libguestfs-tools schroot gawk
  apt-get -q -y install --no-install-recommends \
          pkg-config-aarch64-linux-gnu
  apt-get -q -y install --no-install-recommends sqlite3 libsqlite3-dev
  apt-get -q -y install texinfo

  # for RISC-V / Xv6
  # see: https://pdos.csail.mit.edu/6.828/2021/tools.html
  sudo apt-get -q -y install --no-install-recommends \
    git build-essential gdb-multiarch \
    qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu

  mkdir -p /home/vagrant
  chown -R vagrant:vagrant /home/vagrant/

  apt-get -q -y autoremove
  apt-get -q -y clean
  update-locale LC_ALL=C

  # upgrade cmake to 3.16.1
  wget -q -O cmake-linux.sh https://github.com/Kitware/CMake/releases/download/v3.16.1/cmake-3.16.1-Linux-x86_64.sh
  sudo sh cmake-linux.sh -- --skip-license --prefix=/usr
  rm cmake-linux.sh

  # fetch latest xv6 and book
  # see: https://pdos.csail.mit.edu/6.828/2021/xv6.html
  cd /vagrant
  git clone git://github.com/mit-pdos/xv6-riscv.git
  git clone git://github.com/mit-pdos/xv6-riscv-book.git

  echo
  echo "Provisioning COMPLETE."
  echo
SHELL

  # config.vm.provision 'shell', privileged: true, inline:
  config.vm.provision "shell", inline: $script, privileged: true
end

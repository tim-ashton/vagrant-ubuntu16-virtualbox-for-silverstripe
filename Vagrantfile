# -*- mode: ruby -*-
# vi: set ft=ruby :

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
  config.vm.box = "ubuntu/xenial64"

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
  config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  config.vm.hostname = "ss.test"
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "./", "/vagrant_data", id: "vagrant-root",
    :owner => "vagrant",
    :group => "www-data",
    :mount_options => ["dmode=775", "fmode=664"]

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    #   # Display the VirtualBox GUI when booting the machine
      # vb.gui = true
    #
    #   # Customize the amount of memory on the VM:
    vb.memory = "1024"
    vb.name = "s.test"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    add-apt-repository ppa:ondrej/php
    apt-get update 
    apt-get upgrade
	  apt-get install -y git apache2 curl
    apt-get install php7.2 libapache2-mod-php7.2 php7.2-cli php7.2-curl php7.2-tidy php7.2-common php7.2-mysql php7.2-pdo php7.2-intl php7.2-gd php7.2-xml php7.2-mbstring php7.2-zip -y 
    apt-get install zip unzip -y

    debconf-set-selections <<< 'mysql-server mysql-server/root_password password sstest'
    debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password sstest'
    apt-get install mysql-server -y
    
    #restart apache - not sure if this is really needed.
    service apache2 restart
	
    if ! [ -L /var/www ]; then
      rm -rf /var/www
      ln -fs /vagrant_data/public /var/www

      a2enmod rewrite

      sed -i 's:/var/www/html:/vagrant_data/public:' /etc/apache2/sites-enabled/000-default.conf
      echo "<Directory /vagrant_data/public/>\nOptions Indexes FollowSymLinks\nAllowOverride All\nRequire all granted\n</Directory>\n" >> /etc/apache2/apache2.conf
    fi
	
    if [ ! -f /var/log/databasesetup ];
    then
      echo "CREATE USER 'ssmysqluser'@'localhost' IDENTIFIED BY 'ssmysqlusrpass'" | mysql -uroot -psstest
      echo "CREATE DATABASE ssmydb" | mysql -uroot -psstest
      echo "GRANT ALL ON ssmydb.* TO 'ssmysqluser'@'localhost'" | mysql -uroot -psstest
      echo "flush privileges" | mysql -uroot -psstest
    fi

    # php config
    echo "date.timezone = \"Australia/Brisbane\"" >> /etc/php/7.2/cli/php.ini
    phpenmod -v 7.2 curl tidy gd pdo_mysql mbstring exif mysqli
    
    #install composer
    curl -Ss https://getcomposer.org/installer | php > /dev/null
    mv composer.phar /usr/bin/composer

    chown -R vagrant /home/vagrant/.composer/cache/repo/https---repo.packagist.org
    chown -R vagrant /home/vagrant/.composer/cache/files/


    # install deployer as per ss documentation
    composer require deployer/dist --dev
    php vendor/bin/dep

    #restart apache
    service apache2 restart
  SHELL
end

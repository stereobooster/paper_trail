# -*- mode: ruby -*-
# vi: set ft=ruby :
# Usage:
#   vagrant up && vagrant ssh
#   cd /vagrant && wwtd -u before_script
Vagrant.configure('2') do |config|
  config.vm.box      = 'ubuntu/trusty64'
  config.vm.hostname = 'rails-dev-box'

  config.vm.network :forwarded_port, guest: 3000, host: 3000

  # this requires MacOS or Linux see https://github.com/rails/rails-dev-box#nfs
  config.vm.network :private_network, ip: '192.168.50.4'
  config.vm.synced_folder '.', '/vagrant', type: 'nfs'

  config.vm.provider :virtualbox do |v|
    v.memory = 1024
    v.cpus = 2
  end

  config.vm.provision :shell, keep_color: true, inline: <<-SHELL
    function install {
      echo installing $1
      shift
      apt-get -y install "$@" >/dev/null 2>&1
    }

    if ! hash postgres 2>/dev/null; then
      echo 'deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main' >> /etc/apt/sources.list.d/pgdg.list
      wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -
    fi

    echo updating package information
    # apt-add-repository -y ppa:brightbox/ruby-ng >/dev/null 2>&1
    add-apt-repository ppa:webupd8team/java >/dev/null 2>&1
    apt-get -y update >/dev/null 2>&1

    install 'development tools' build-essential python-software-properties libreadline-dev zlib

    if ! hash git 2>/dev/null; then
      install Git git
    fi

    if ! hash java 2>/dev/null; then
      install Java8 oracle-java8-installer
      # install Java default-jdk
    fi

    if ! hash sqlite3 2>/dev/null; then
      install SQLite sqlite3 libsqlite3-dev
    fi

    if ! hash postgres 2>/dev/null; then
      install PostgreSQL postgresql-9.4 postgresql-contrib libpq-dev
      sudo -u postgres createuser --superuser vagrant
      sudo -u postgres psql -c 'create database vagrant;'
    fi

    if ! hash mysql 2>/dev/null; then
      debconf-set-selections <<< 'mysql-server mysql-server/root_password password root'
      debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password root'
      install MySQL mysql-server libmysqlclient-dev
      mysql -uroot -proot -e "CREATE USER 'travis'@'localhost'; UPDATE user SET Password='' WHERE user='travis';GRANT ALL PRIVILEGES ON *.* TO 'travis'@'localhost' WITH GRANT OPTION;" mysql
      mysql -uroot -proot -e "CREATE USER 'vagrant'@'localhost'; UPDATE user SET Password='' WHERE user='vagrant';GRANT ALL PRIVILEGES ON *.* TO 'vagrant'@'localhost' WITH GRANT OPTION;" mysql
    fi
  SHELL

  config.vm.provision :shell, privileged: false, keep_color: true, inline: <<-SHELL
    function install_ruby {
      if [ ! -d "$HOME/.rbenv/versions/$1" ]; then
        echo installing ruby $1

        rbenv install $1 >/dev/null 2>&1 3>&1
      else
        echo ruby $1 installed
      fi

      rbenv shell $1
      rbenv rehash

      echo "  " updating gems
      gem update --system >/dev/null 2>&1

      echo "  " installing Bundler
      gem install bundler >/dev/null 2>&1
    }

    if [ ! -d "$HOME/.rbenv" ]; then
      echo installing rbenv
      git clone git://github.com/sstephenson/rbenv.git $HOME/.rbenv >/dev/null 2>&1
      echo 'export PATH="$HOME/.rbenv/bin:\$PATH"' >> $HOME/.profile
      echo 'eval "$(rbenv init -)"' >> $HOME/.profile

      echo installing ruby-build
      git clone git://github.com/sstephenson/ruby-build.git $HOME/.rbenv/plugins/ruby-build >/dev/null 2>&1
      echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:\$PATH"' >> $HOME/.profile

    else
      echo updating rbenv
      cd $HOME/.rbenv
      git pull >/dev/null 2>&1

      echo updating ruby-build
      cd $HOME/.rbenv/plugins/ruby-build
      git pull >/dev/null 2>&1

      cd $HOME
    fi

    source $HOME/.profile

    # no rdoc for installed gems
    echo 'gem: --no-ri --no-rdoc' >> $HOME/.gemrc

    install_ruby 1.9.3-p551

    install_ruby jruby-1.7.19

    install_ruby 2.0.0-p648

    rbenv global 2.0.0-p648
    rbenv rehash

    echo installing wwtd
    gem install wwtd >/dev/null 2>&1
  SHELL
end

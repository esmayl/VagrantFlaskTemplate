# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  # use default box
  config.vm.box = "ubuntu/xenial64"

  # forward port guest machine:5000 -> host machine:5000
  # port 5000 is default for flask web app
  config.vm.network "forwarded_port", guest: 8000, host: 8000

  config.vm.provision "file", source: "~/.ssh/vvv_ssh_key", destination: "~/.ssh/vvv_ssh_key"

  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update -y
    sudo apt-get upgrade -y
    sudo apt-get install -y libsm6 libxext6 libxrender-dev # bug fix for opencv-python error libSM.so
    sudo apt-get install -y nginx-core \
    python3-pip \
    python3-dev \
    build-essential \
    libssl-dev \
    libffi-dev \
    python3-setuptools \
    python3-venv
  SHELL

  # pip packages
  config.vm.provision "shell", reset:true,inline: <<-SHELL
    python3 -m pip install --upgrade pip
    pip3 install wheel
    pip3 install uwsgi
    pip3 install flask
    pip3 install opencv-python
    pip3 install uwsgi
    pip3 install markupsafe
  SHELL

  # ssh setup
  config.ssh.forward_agent = true

  config.vm.provision "shell", reset:true, inline: <<-SHELL
    sudo apt-get install -y git
    chmod 700 ~/.ssh
    ssh-keyscan -H github.com >> ~/.ssh/known_hosts
    sudo rm -R /var/www/html/
    git clone git@github.com:esmayl/FirstFlaskApp.git /var/www/html
    cd /var/www/html
    git checkout origin/Development
    sudo ln -s /var/www/html/FirstFlaskApp /etc/nginx/sites-enabled/FirstFlaskApp
    sudo chown www-data:www-data /var/www/html
    sudo chown www-data:www-data /var/www/html/* -R
    sudo rm /etc/nginx/sites-enabled/default
    sudo usermod -g www-data vagrant
    sudo chmod 770 . -R
    sudo ufw allow 8000
    sudo systemctl restart nginx
  SHELL

  #
  # # start uwsgi srver
  config.vm.provision "shell", reset:true,privileged: false, inline: <<-SHELL
     cd /var/www/html
     uwsgi --ini ./uwsgi.ini
  SHELL
end

# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Базовый образ
  # config.vm.box = "archlinux/archlinux"
  config.vm.box = "./images/Arch-Linux-x86_64-virtualbox-20220418.0.box"

  # Название хоста
  config.vm.hostname = "Doplom"
  config.vm.define "Doplom"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Открываем порт 9000 только для локальных запросов
  config.vm.network "forwarded_port", guest: 3128, host: 3128, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 9000, host: 9000, host_ip: "127.0.0.1"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Выдаём больше памяти
  config.vm.provider "virtualbox" do |vb|
     vb.memory = "2024"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Обновление системы
   config.vm.provision "shell", inline: <<-SHELL
      pacman -Suy  gcc-go postgresql squid openssl --noconfirm
      yes | pacman -Scc
   SHELL

   # Синхронизация каталога репозитория с виртуальной машиной
   config.vm.synced_folder ".", "/app"
   #Сборка прогарммы
   config.vm.provision "shell", inline: <<-SHELL
     cd /app/
     go mod download
     go mod tidy
     go build -compiler=gccgo main.go
   SHELL

   # Инициализация базы данных
   config.vm.provision "shell", inline: <<-SHELL
     sudo -u postgres initdb -E UTF8 -D /var/lib/postgres/data
     systemctl start postgresql.service 
     sudo -u postgres psql --command "CREATE USER test_user WITH PASSWORD 'test';"
     sudo -u postgres createdb -O test_user test_bd
     systemctl enable postgresql.service --now
   SHELL

   # Создадим юнит сервера
   config.vm.provision "shell", inline: <<-SHELL
     echo "[Unit]
Description=Управляющий сервер doplom
[Service]
WorkingDirectory=/app
Type=simple
ExecStart=/app/main
[Install]
WantedBy=multi-user.target" > /etc/systemd/system/doplom.service
     systemctl daemon-reload
     systemctl enable doplom.service --now
   SHELL


end
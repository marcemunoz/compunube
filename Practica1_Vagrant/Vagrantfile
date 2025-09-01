# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.no_install = true
    config.vbguest.auto_update = false
    config.vbguest.no_remote = true
  end

  # ----------------- CLIENTE -----------------
  config.vm.define "clienteUbu" do |clienteUbu|
    clienteUbu.vm.box = "bento/ubuntu-18.04"
    clienteUbu.vm.network "private_network", ip: "192.168.100.11"
    clienteUbu.vm.hostname = "clienteUbu"
    clienteUbu.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
    end

    clienteUbu.vm.provision "shell", inline: <<-SHELL
      sudo apt update -y
      sudo apt install -y snapd
      sudo snap install lxd

      sudo usermod -aG lxd vagrant
      newgrp lxd

      lxd init --auto
      sleep 5

      # Crear contenedores web2, web3, web4
      lxc launch ubuntu:18.04 web2
      sleep 10
      lxc exec web2 -- apt update -y
      lxc exec web2 -- apt install -y apache2
      lxc exec web2 -- mkdir -p /var/www/html
      lxc exec web2 -- tee /var/www/html/index.html <<< "<h1>Hello from web2</h1>"
      lxc config device add web2 http proxy listen=tcp:0.0.0.0:80 connect=tcp:127.0.0.1:80

      lxc launch ubuntu:18.04 web3
      sleep 10
      lxc exec web3 -- apt update -y
      lxc exec web3 -- apt install -y apache2
      lxc exec web3 -- mkdir -p /var/www/html
      lxc exec web3 -- tee /var/www/html/index.html <<< "<h1>Hello from web3 (backup)</h1>"
      lxc config device add web3 http proxy listen=tcp:0.0.0.0:81 connect=tcp:127.0.0.1:80

      lxc launch ubuntu:18.04 web4
      sleep 10
      lxc exec web4 -- apt update -y
      lxc exec web4 -- apt install -y apache2
      lxc exec web4 -- mkdir -p /var/www/html
      lxc exec web4 -- tee /var/www/html/index.html <<< "<h1>Hello from web4 (backup)</h1>"
      lxc config device add web4 http proxy listen=tcp:0.0.0.0:82 connect=tcp:127.0.0.1:80
    SHELL
  end

  # ----------------- SERVIDOR -----------------
  config.vm.define "servidorUbu" do |servidorUbu|
    servidorUbu.vm.box = "bento/ubuntu-18.04"
    servidorUbu.vm.network "private_network", ip: "192.168.100.12"
    servidorUbu.vm.hostname = "servidorUbu"
    servidorUbu.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
    end

    servidorUbu.vm.provision "shell", inline: <<-SHELL
      sudo apt update -y
      sudo apt install -y snapd
      sudo snap install lxd

      sudo usermod -aG lxd vagrant
      newgrp lxd

      lxd init --auto
      sleep 5

      # Crear contenedor web1
      lxc launch ubuntu:18.04 web1
      sleep 10
      lxc exec web1 -- apt update -y
      lxc exec web1 -- apt install -y apache2
      lxc exec web1 -- mkdir -p /var/www/html
      lxc exec web1 -- tee /var/www/html/index.html <<< "<h1>Hello from web1</h1>"
      lxc config device add web1 http proxy listen=tcp:0.0.0.0:80 connect=tcp:127.0.0.1:80

      # Crear contenedor haproxy
      lxc launch ubuntu:18.04 haproxy
      sleep 10
      lxc exec haproxy -- apt update -y
      lxc exec haproxy -- apt install -y haproxy

      # Configuración mínima de HAProxy apuntando a las IP de las máquinas
      lxc exec haproxy -- bash -c "cat > /etc/haproxy/haproxy.cfg <<'EOF'
global
    log /dev/log local0
    log /dev/log local1 notice
    maxconn 2048
    daemon

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5s
    timeout client  50s
    timeout server  50s

# FRONTEND: donde escucha HAProxy
frontend http
    bind *:80
    default_backend web-backend

# BACKEND: los servidores web reales
backend web-backend
    balance roundrobin

    # Panel de estadísticas
    stats enable
    stats uri /haproxy?stats
    stats auth admin:admin

    # Tus servidores web
    server web1 192.168.100.12:80 check
    server web2 192.168.100.11:8080 check
    server web3 192.168.100.11:8081 check backup
    server web4 192.168.100.11:8082 check backup

     # Página personalizada si no hay servidores disponibles
    errorfile 503 /etc/haproxy/errors/503.http
    
EOF"

      lxc exec haproxy -- systemctl restart haproxy
    SHELL
  end
end

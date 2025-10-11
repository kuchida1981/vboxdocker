# -*- mode: ruby -*-
# vi: set ft=ruby :

docker_guest_port = ENV["VBOXDCR_GUESTPORT"] || "2376"
docker_host_port  = ENV["VBOXDCR_HOSTPORT"] || "12376"
shared_dir        = ENV["VBOXDCR_SHAREDDIR"] || ENV["HOME"]
name              = ENV["VBOXDCR_NAME"] || "default"
vm_disksize       = ENV["VBOXDCR_VM_DISKSIZE"] || "20GB"
vm_memory         = ENV["VBOXDCR_VM_MEMORY"] || "1024"
vm_cpus           = ENV["VBOXDCR_VM_CPUS"] || "2"
network           = ENV["VBOXDCR_NETWORK"] || "docker"

docker_guest_host = "tcp://0.0.0.0:#{docker_guest_port}"

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"
  config.vm.disk :disk, size: vm_disksize, primary: true

  config.vm.define name do |name|
  end

  config.vm.provider "virtualbox" do |vb|
    vb.memory = vm_memory.to_i
    vb.cpus = vm_cpus.to_i
    vb.customize ["setextradata", :id, "VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled", 1]
  end

  config.vm.network "forwarded_port", guest: docker_guest_port, host: docker_host_port
  config.vm.network "private_network", type: "dhcp", name: network

  config.vm.synced_folder shared_dir, shared_dir

  config.vm.provision "shell", inline: <<-SHELL
  resize2fs /dev/sda1
  echo DOCKER_HOST=#{docker_guest_host} >> /etc/environment
  export DOCKER_HOST=#{docker_guest_host}

  mkdir -p /etc/systemd/system/docker.service.d
  cat <<EOF > /etc/systemd/system/docker.service.d/override.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H #{docker_guest_host}
EOF

  apt-get update && apt-get install -y curl systemd-timesyncd
  curl -fsSL https://get.docker.com | sh
  adduser vagrant docker
  SHELL
end

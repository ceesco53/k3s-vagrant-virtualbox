# -*- mode: ruby -*-
# vi: set ft=ruby :

# HA cluster with bridged networking on vbox
k3s_version = "v1.28.9-rc1+k3s1" #make sure compatible with your workloads (SCDF isn't 1.29 certified)
box = "ubuntu/jammy64"
iface = "enp0s8" #eth1 on alpine, enp0s8 for ubuntu jammy

leader1_ip = "192.168.0.3" #leader1

leaders = { "leader2" => "192.168.0.4" ,
            "leader3" => "192.168.0.5" }

workers = { "worker1" => "192.168.0.6",
           "worker2" => "192.168.0.7",
           "worker3" => "192.168.0.8",
           "worker4" => "192.168.0.9" }

# Extra parameters in INSTALL_K3S_EXEC variable because of
# K3s picking up the wrong interface when starting server and agent
# https://github.com/alexellis/k3sup/issues/306

Vagrant.configure("2") do |config|
  config.vm.box = box

  # start with a canary vm (leader1)
  config.vm.define "leader1", primary: true do |leader1|
    leader1.vm.network "public_network", ip: leader1_ip, netmask:"255.255.254.0"
    leader1.vm.synced_folder "./shared", "/vagrant_shared"
    leader1.vm.hostname = "leader1"
    leader1.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = "2"
    end
    leader1.vm.provision "shell", inline: <<-SHELL
      sudo -i
      apk add curl
      export INSTALL_K3S_VERSION="#{k3s_version}"
      export INSTALL_K3S_EXEC="--bind-address=#{leader1_ip} --node-external-ip=#{leader1_ip} --flannel-iface=#{iface}"
      curl -sfL https://get.k3s.io | sh -s - server --cluster-init
      echo "Sleeping for 5 seconds to wait for k3s to start"
      sleep 5
      cp /var/lib/rancher/k3s/server/token /vagrant_shared
      cp /etc/rancher/k3s/k3s.yaml /vagrant_shared
    SHELL
  end

  #let's add in the additional leader vms for HA control plane
  leaders.each do |leader_name, leader_ip|
    config.vm.define leader_name do |leader|
      leader.vm.network "public_network", ip: leader_ip, netmask:"255.255.254.0"
      leader.vm.synced_folder "./shared", "/vagrant_shared"
      leader.vm.hostname = leader_name
      leader.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
        vb.cpus = "2"
      end
      leader.vm.provision "shell", inline: <<-SHELL
        sudo -i
        apk add curl
        export INSTALL_K3S_VERSION="#{k3s_version}"
        export K3S_TOKEN_FILE=/vagrant_shared/token
        export K3S_URL=https://#{leader1_ip}:6443
        curl -sfL https://get.k3s.io | sh -s - server --bind-address=#{leader_ip} --node-external-ip=#{leader_ip} --server https://#{leader1_ip}:6443 --flannel-iface=#{iface}
        echo "Sleeping for 5 seconds to wait for k3s to start"
        sleep 5
      SHELL
    end
  end

  #now let's get the data plane up
  workers.each do |worker_name, worker_ip|
    config.vm.define worker_name do |worker|
      worker.vm.network "public_network", ip: worker_ip, netmask:"255.255.254.0"
      worker.vm.synced_folder "./shared", "/vagrant_shared"
      worker.vm.hostname = worker_name
      worker.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
        vb.cpus = "2"
      end
      worker.vm.provision "shell", inline: <<-SHELL
        sudo -i
        apk add curl
        export INSTALL_K3S_VERSION="#{k3s_version}"
        export K3S_TOKEN_FILE=/vagrant_shared/token
        export K3S_URL=https://#{leader1_ip}:6443
        export INSTALL_K3S_EXEC="--flannel-iface=#{iface}"
        curl -sfL https://get.k3s.io | sh -
      SHELL
    end
  end
end
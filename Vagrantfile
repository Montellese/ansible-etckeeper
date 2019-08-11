# -*- mode: ruby -*-
# vi: set ft=ruby ts=2 sw=2 tw=0 et :

boxes = [
  # {
  #   :name => "ubuntu-1604",
  #   :box => "ubuntu/xenial64",
  #   :ip => '10.0.0.13',
  #   :cpu => "50",
  #   :ram => "256"
  # },
  {
    :name => "ubuntu-1804",
    :box => "ubuntu/bionic64",
    :ip => '10.0.0.14',
    :cpu => "50",
    :ram => "512"
  },
  # {
  #   :name => "debian-8",
  #   :box => "generic/debian8",
  #   :ip => '10.0.0.16',
  #   :cpu => "50",
  #   :ram => "256"
  # },
  # {
  #   :name => "debian-9",
  #   :box => "generic/debian9",
  #   :ip => '10.0.0.17',
  #   :cpu => "50",
  #   :ram => "512"
  # },
  {
    :name => "debian-10",
    :box => "generic/debian10",
    :ip => '10.0.0.18',
    :cpu => "50",
    :ram => "512"
  },
]

role = File.basename(File.expand_path(File.dirname(__FILE__)))

Vagrant.configure("2") do |config|
  boxes.each do |box|
    config.vm.define box[:name] do |vms|
      vms.vm.box = box[:box]
      vms.vm.hostname = "#{role}-#{box[:name]}"

      vms.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--cpuexecutioncap", box[:cpu]]
        v.customize ["modifyvm", :id, "--memory", box[:ram]]
      end

      vms.vm.network :private_network, ip: box[:ip]

      vms.vm.provision :ansible do |ansible|
        ansible.playbook = "tests/vagrant.yml"
        ansible.verbose = "vv"
        ansible.compatibility_mode = "2.0"
      end
    end
  end
end

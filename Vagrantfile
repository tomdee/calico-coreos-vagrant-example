# -*- mode: ruby -*-
# # vi: set ft=ruby :

# The version of calico to install
calico_docker_ver = "v0.5.2"

# Size of the cluster created by Vagrant
num_instances=2

# Change basename of the VM
instance_name_prefix="calico"

# The IP address of the first server
primary_ip = "172.17.8.101"

# Official CoreOS channel from which updates should be downloaded
update_channel='alpha'

Vagrant.configure("2") do |config|
  # always use Vagrants insecure key
  config.ssh.insert_key = false

  config.vm.box = "coreos-%s" % update_channel
  config.vm.box_version = ">= 308.0.1"
  config.vm.box_url = "http://%s.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json" % update_channel

  config.vm.provider :virtualbox do |v|
    # On VirtualBox, we don't have guest additions or a functional vboxsf
    # in CoreOS, so tell Vagrant that so it can be smarter.
    v.check_guest_additions = false
    v.functional_vboxsf     = false
  end

  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  # Set up each box
  (1..num_instances).each do |i|
    vm_name = "%s-%02d" % [instance_name_prefix, i]
    config.vm.define vm_name do |host|
      host.vm.hostname = vm_name

      ip = "172.17.8.#{i+100}"
      host.vm.network :private_network, ip: ip

      #config.vm.provision "docker", images: ["calico/node:#{calico_docker_ver}", "busybox:latest"]

      # Use a different cloud-init on the first server.
      if i == 1
        config.vm.provision :file, :source => "user-data-first", :destination => "/tmp/vagrantfile-user-data"
        config.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true
      else
        config.vm.provision :file, :source => "user-data-others", :destination => "/tmp/vagrantfile-user-data"
        config.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true
      end

      config.vm.post_up_message = "Vagrant has finished but cloud-init might still be executing.
      Check the progress using systemctl status -f"
    end
  end
end
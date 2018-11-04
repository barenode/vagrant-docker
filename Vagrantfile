# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|                                          
  
  # box 
  config.vm.box = "/dev/vagrant/c72/package.box"    
  
  #synced folders
  config.vm.synced_folder '.', '/opt/dev', type: 'nfs'


  #defaul VM settings
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", 18120] 
    vb.customize ["modifyvm", :id, "--cpus", 8] 
  end
  
    # asciidoctor
   config.vm.define :docker do |docker|
     docker.vm.hostname = "docker.barenode.org"
     docker.vm.network :private_network, ip: "192.168.72.101"
   end 
  
$script = <<SCRIPT

#!/bin/bash

#docker
curl -fsSL https://get.docker.com/ | sh
sudo systemctl start docker
systemctl enable docker.service

#preflight kubernetes
systemctl stop firewalld
swapoff -a

docker pull registry
systemctl restart docker
#docker run -dit -p 5000:5000 --name registry registry

sed -i 's%dockerd%dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock --insecure-registry 192.168.72.101:5000%' /usr/lib/systemd/system/docker.service

systemctl daemon-reload
systemctl restart docker
docker run -dit -p 5000:5000 --name registry registry
 
docker pull jboss/wildfly
docker build -t 192.168.72.101:5000/wildfly /opt/dev/docker/wildfly/.
docker push 192.168.72.101:5000/wildfly

SCRIPT
  config.vm.provision :shell do |shell|
    shell.inline = $script
  end     
end
                        
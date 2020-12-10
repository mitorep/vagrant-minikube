INSTANCES=1

Vagrant.configure("2") do |config|
  config.vm.box = "centos/8"
  config.disksize.size = '30GB'
  config.vm.synced_folder '.', '/vagrant', disabled: true
  config.ssh.insert_key = false
  (0..INSTANCES-1).each do |i|
    config.vm.define "centos#{i}" do |node|

      node.vm.network "private_network", ip: "192.100.10.#{i+10}"
      node.vm.hostname = "centos#{i}.mito.local"
      #node.vm.synced_folder "./provision/", "/provision/", create: true

      node.vm.provider "virtualbox" do |vb|
        vb.memory = 6144
        vb.cpus = 4
      end

      node.vm.provision "shell", inline: <<-SHELL
        sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
        sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config
        service sshd restart

        dnf install -y cloud-utils-growpart 
        growpart /dev/sda 1
        xfs_growfs /dev/sda1

        yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
        sudo yum install -y docker-ce docker-ce-cli containerd.io conntrack
        systemctl start docker
        systemctl enable docker
        usermod -aG docker vagrant

        setenforce 0
        sed -i 's/enforcing/disabled/g' /etc/selinux/config

        curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
        chmod +x ./kubectl
        mv ./kubectl /usr/local/bin/kubectl
        kubectl version --client

        curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-latest.x86_64.rpm
        rpm -ivh minikube-latest.x86_64.rpm
      SHELL

    end 
  end
end
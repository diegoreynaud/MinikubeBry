#IP 192.168.56.101

$install_docker = <<-SCRIPT    
  apt install -y docker.io
SCRIPT

$install_minikube = <<-SCRIPT    
  wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && \
  cp minikube-linux-amd64 /usr/local/bin/minikube && \
  chmod 755 /usr/local/bin/minikube && \
  usermod -aG docker vagrant && newgrp docker  
SCRIPT

$install_kubectl = <<-SCRIPT    
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - && \
  echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list && \
  apt-get update -y && \
  apt-get install kubectl -y
SCRIPT

$install_net_tools = <<-SCRIPT    
  apt install net-tools -y
SCRIPT

$install_helm = <<-SCRIPT    
  curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null && \
  apt-get install apt-transport-https --yes && \
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list && \
  apt-get update && \
  apt-get install helm
SCRIPT

$install_mkcert = <<-SCRIPT
  wget https://github.com/FiloSottile/mkcert/releases/download/v1.4.3/mkcert-v1.4.3-linux-amd64 && \
  mv mkcert-v1.4.3-linux-amd64 mkcert && \
  chmod +x mkcert && \
  mv mkcert /usr/local/bin/ && \
  mkcert -install && \
  mkcert -cert-file whoami.crt -key-file whoami.key minikube.local
SCRIPT

$install_cert_manager = <<-SCRIPT
  su -c \"kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.5.4/cert-manager.crds.yaml\" -s /bin/sh vagrant && \
  su -c \"kubectl create namespace cert-manager\" -s /bin/sh vagrant && \
  su -c \"helm repo add jetstack https://charts.jetstack.io\" -s /bin/sh vagrant && \
  su -c \"helm repo update\" -s /bin/sh vagrant && \
  su -c \"helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.5.4\" -s /bin/sh vagrant
SCRIPT


$kubectl_apply = <<-SCRIPT
  su -c \"kubectl apply -f /home/vagrant/k8s/letsencrypt-issuer.yaml\" -s /bin/sh vagrant && \
  su -c \"kubectl apply -f /home/vagrant/k8s/whoami-deployment.yaml\" -s /bin/sh vagrant && \
  su -c \"kubectl apply -f /home/vagrant/k8s/whoami-service.yaml\" -s /bin/sh vagrant && \
  su -c \"kubectl apply -f /home/vagrant/k8s/whoami-ingress.yaml\" -s /bin/sh vagrant  
SCRIPT

Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/focal64"

    config.vm.define "minikubeauto" do |minikubeauto|
        
        minikubeauto.vm.provider "virtualbox" do |vb|
            vb.cpus = 4
            vb.memory = 4096
            vb.name = "minikubeauto"
        end
        
        #network
        minikubeauto.vm.network "private_network", ip: "192.168.56.101"

        #Install
        minikubeauto.vm.provision "shell", inline: "echo UPDATE AND UPGRADE"
        minikubeauto.vm.provision "shell", inline: "apt-get update -y && apt-get upgrade -y"
        minikubeauto.vm.provision "shell", inline: "apt-get install curl wget apt-transport-https -y"
        
        minikubeauto.vm.provision "shell", inline: "echo INSTALL NET_TOOLS"
        minikubeauto.vm.provision "shell", inline: $install_net_tools

        minikubeauto.vm.provision "shell", inline: "echo INSTALL DOCKER"
        minikubeauto.vm.provision "shell", inline: $install_docker      

        #setup geral
        minikubeauto.vm.synced_folder "./configs", "/configs"
        minikubeauto.vm.provision "shell", inline: "cat /configs/docker_key.pub >> .ssh/authorized_keys"

        minikubeauto.vm.provision "shell", inline: "echo INSTALL HELM"
        minikubeauto.vm.provision "shell", inline: $install_helm

        minikubeauto.vm.provision "shell", inline: "echo INSTALL MINIKUBE"
        minikubeauto.vm.provision "shell", inline: $install_minikube
        
        minikubeauto.vm.provision "shell", inline: "echo INSTALL KUBECTL"
        minikubeauto.vm.provision "shell", inline: $install_kubectl

        minikubeauto.vm.provision "shell", inline: "echo STARTING MINIKUBE"
        minikubeauto.vm.provision "shell", inline: "su -c \"minikube start\" -s /bin/sh vagrant" 
        minikubeauto.vm.provision "shell", inline: "su -c \"minikube addons enable ingress\" -s /bin/sh vagrant"
                
        #setup k8s
        minikubeauto.vm.provision "shell", inline: "echo SETUP K8s"
        minikubeauto.vm.synced_folder "./k8s", "/home/vagrant/k8s"
        minikubeauto.vm.provision "shell", inline: "cat ./k8s/dominio >> /etc/hosts"
        minikubeauto.vm.provision "shell", inline: $install_cert_manager
        minikubeauto.vm.provision "shell", inline: $install_mkcert
        minikubeauto.vm.provision "shell", inline: $kubectl_apply

    end
end
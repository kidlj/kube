# -*- mode: ruby -*-
# vi: set ft=ruby :

servers = [
    {
        :name => "master",
        :type => "master",
        :box => "bento/ubuntu-16.04",
        :eth1 => "192.168.200.10",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "node1",
        :type => "node",
        :box => "bento/ubuntu-16.04",
        :eth1 => "192.168.200.11",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "node2",
        :type => "node",
        :box => "bento/ubuntu-16.04",
        :eth1 => "192.168.200.12",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "node3",
        :type => "node",
        :box => "bento/ubuntu-16.04",
        :eth1 => "192.168.200.13",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "node4",
        :type => "node",
        :box => "bento/ubuntu-16.04",
        :eth1 => "192.168.200.14",
        :mem => "2048",
        :cpu => "2"
    }
]

# This script to install k8s using kubeadm will get executed after a box is provisioned
$configureBox = <<-SCRIPT

# ensure kube-proxy ipvs mode
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4

cat <<EOF > /etc/modules-load.d/ipvs.conf
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack_ipv4
EOF

# install docker v17.03
# reason for not using docker provision is that it always installs latest version of the docker, but kubeadm requires 17.03 or older
apt-get update
apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable"
apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')

# run docker commands as vagrant user (sudo not required)
usermod -aG docker vagrant

# install kubeadm
apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
# install kubernetes 1.12.3
KUBE_VERSION=1.12.3-00
apt-get install -y kubelet=$KUBE_VERSION kubectl=$KUBE_VERSION kubeadm=$KUBE_VERSION
apt-mark hold kubelet kubeadm kubectl

# kubelet requires swap off
swapoff -a

# keep swap off after reboot
sed -i '/ swap / s/^(.*)$/#\1/g' /etc/fstab

# ip of this box
IP_ADDR=$1
# set node-ip
sudo sed -i "/^[^#]*KUBELET_EXTRA_ARGS=/c\KUBELET_EXTRA_ARGS=--node-ip=$IP_ADDR" /etc/default/kubelet
sudo systemctl restart kubelet
SCRIPT

$configureMaster = <<-SCRIPT
echo "This is master"

# ip of this box
IP_ADDR=$1

# install k8s master
HOST_NAME=$(hostname -s)
kubeadm init --apiserver-advertise-address=$IP_ADDR --apiserver-cert-extra-sans=$IP_ADDR  --node-name $HOST_NAME --pod-network-cidr=192.168.0.0/16

#copying credentials to regular user - vagrant
sudo --user=vagrant mkdir -p /home/vagrant/.kube
cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config

# install Calico pod network addon
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml

kubeadm token create --print-join-command >> /etc/kubeadm_join_cmd.sh
chmod +x /etc/kubeadm_join_cmd.sh

# required for setting up password less ssh between guest VMs
sudo sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
sudo service sshd restart

SCRIPT

$configureNode = <<-SCRIPT
echo "This is worker"
apt-get install -y sshpass
sshpass -p "vagrant" scp -o StrictHostKeyChecking=no vagrant@192.168.200.10:/etc/kubeadm_join_cmd.sh .
sh ./kubeadm_join_cmd.sh
SCRIPT

Vagrant.configure("2") do |config|

    servers.each do |opts|
        config.vm.define opts[:name] do |config|
            config.vm.box = opts[:box]
            config.vm.hostname = opts[:name]
            config.vm.network :private_network, ip: opts[:eth1]

            config.vm.provider "virtualbox" do |v|
                v.gui = false
                v.name = opts[:name]
                v.memory = opts[:mem]
                v.cpus = opts[:cpu]
            end

            # we cannot use this because we can't install the docker version we want - https://github.com/hashicorp/vagrant/issues/4871
            #config.vm.provision "docker"

            config.vm.provision "shell", inline: $configureBox, args: opts[:eth1]

            if opts[:type] == "master"
                config.vm.provision "shell", inline: $configureMaster, args: opts[:eth1]
            else
                config.vm.provision "shell", inline: $configureNode
            end

        end

    end

end 

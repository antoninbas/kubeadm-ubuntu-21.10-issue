VAGRANTFILE_API_VERSION = "2"

K8S_POD_NETWORK_V4_CIDR = "10.10.0.0/16"
K8S_POD_NETWORK_CIDR = K8S_POD_NETWORK_V4_CIDR

K8S_SERVICE_NETWORK_V4_CIDR = "10.96.0.0/12"
K8S_SERVICE_NETWORK_CIDR = K8S_SERVICE_NETWORK_V4_CIDR

NODE_NETWORK_V4_PREFIX = "192.168.77."

MEMORY = 2048

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/impish64"

  config.vm.provider "virtualbox" do |v|
    v.memory = MEMORY
    # 2 CPUS required to initialize K8s cluster with "kubeadm init"
    v.cpus = 2
  end

  groups = {
    "controlplane" => ["k8s-node-control-plane"],
  }

  config.vm.define "k8s-node-control-plane" do |node|
    node.vm.hostname = "k8s-node-control-plane"
    node_ipv4 = NODE_NETWORK_V4_PREFIX + "100"
    # The network will be configured using the Ansible playbook
    # Despite setting auto_config to false, it seems that it is necessary to
    # provide an IP address, even though it won't be used.
    # See https://github.com/hashicorp/vagrant/issues/7583
    node.vm.network "private_network", ip: node_ipv4, auto_config: false
    node_ip = node_ipv4

    node.vm.provision :ansible  do |ansible|
      ansible.playbook = "playbook/k8s.yml"
      ansible.groups = groups
      ansible.extra_vars = {
        # Ubuntu bionic does not ship with python2
        ansible_python_interpreter:"/usr/bin/python3",
        node_ip: node_ip,
        node_ipv4: node_ipv4,
        node_name: "k8s-node-control-plane",
        k8s_pod_network_cidr: K8S_POD_NETWORK_CIDR,
        k8s_service_network_cidr: K8S_SERVICE_NETWORK_CIDR,
        k8s_api_server_ip: node_ipv4,
      }
    end
  end
end

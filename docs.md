# Containerd
* Install necessary packages
  <br>
   ```
    sudo apt-get update
    sudo apt-get install ca-certificates curl gnupg
    
 * Add the official GPG key  
   <br>
   ```  
   sudo install -m 0755 -d /etc/apt/keyrings
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
   sudo chmod a+r /etc/apt/keyrings/docker.gpg

 * Add the key to repository
   <br>  
   ```
    echo \
    "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

 * Update the apt package
    <br>  
    ```
    sudo apt-get update
      
* Installing Containerd
    <br> 
   ```
    sudo apt-get install containerd.io 

* Configuring the systemd cgroup driver
  
   To use the systemd cgroup driver in /etc/containerd/config.toml with runc, set:
   <br>
   ```
   [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
       SystemdCgroup = true
* Restart the containerd
  <br> 
  ```
  sudo systemctl restart containerd      

# kubernetes Cluster v1.26.2
* Forwarding IPv4 and letting iptables see bridged traffic
  <br> 
  ```
  cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
  overlay
  br_netfilter
  EOF

  sudo modprobe overlay
  sudo modprobe br_netfilter

  # sysctl params required by setup, params persist across reboots
  cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
  net.bridge.bridge-nf-call-iptables  = 1
  net.bridge.bridge-nf-call-ip6tables = 1
  net.ipv4.ip_forward                 = 1
  EOF
  # Apply sysctl params without reboot
  sudo sysctl --system
* Verify that the br_netfilter, overlay modules are loaded
  <br>
  ```
  lsmod | grep br_netfilter
  lsmod | grep overlay
* Verify that the net.bridge.bridge-nf-call-iptables, net.bridge.bridge-nf-call-ip6tables, and net.ipv4.ip_forward system variables are set to 1 in your sysctl config
  <br>
  ```
  sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
* Disable Swap
  <br>
  ```
  sudo swapoff -a
  sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

* install packages needed to use the Kubernetes
  <br>
  ```
  sudo apt-get update
  sudo apt-get install -y apt-transport-https ca-certificates curl

* Google Cloud public signing key
  <br>
  ```
  sudo curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -  
  
* Add the Kubernetes apt repository  
  <br>
  ```
  sudo apt-add-repository  "deb https://apt.kubernetes.io/ kubernetes-xenial main"

* Update apt package index, install kubelet, kubeadm and kubectl, and pin their version
  <br>
  ```
  sudo apt-get update
  sudo apt-get install -y kubelet=1.26.2-00 kubeadm=1.26.2-00 kubectl=1.26.2-00
  sudo apt-mark hold kubelet kubeadm kubectl
* Initializing your control-plane node
  <br>
  ```
  kubeadm init --pod-network-cidr=10.244.0.0/16
  ```
  ```
  Your Kubernetes control-plane has initialized successfully!

  To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

  You should now deploy a Pod network to the cluster.
  Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  /docs/concepts/cluster-administration/addons/

  You can now join any number of machines by running the following on each node
  as root:

  kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>

## Installing Addons 
 * Weave Net
   <br> 
   ```
   kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml   
   ```
   If you do set the --cluster-cidr option on kube-proxy, make sure it matches the IPALLOC_RANGE given to Weave Net:
   <br>
   ```
   kubectl edit  ds -n kube-system weave-net
   ```
   <br>
   

        containers:
         - name: weave
           env:
            - name: IPALLOC_RANGE
              value: 10.0.0.0/16

   
  
 
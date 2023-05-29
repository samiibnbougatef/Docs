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
* 
  
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
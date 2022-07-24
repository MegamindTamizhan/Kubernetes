# Install Kubernetes Cluster on Ubuntu 21.04

**Prerequisites:**

 1. Minimum 2 CPU’s with 4Gb Memory is required.

   `$ cat /proc/cpuinfo | grep -i processor (to check cpu)`
   `$ free -h (to check memory)`

 2. Make an entry of each host in /etc/hosts file for name resolution on all kubernetes nodes as below or configure it on DNS if you have DNS server.
```
    $ cat /etc/hosts
    127.0.0.1 localhost
    172.16.15.11 master
    172.16.15.12 nodeone
    172.16.15.13 nodesecond
```
 3. Make sure kubernetes master and worker nodes are reachable between each other.
 
 4. Kubernetes doesn’t support **“Swap”**. Disable Swap on all nodes using below command and also to make it permanent comment out the swap entry in `/etc/fstab` file.

    `$ sudo swapoff -a`

 5. Internet must be enabled on all nodes, because required packages for kubernetes cluster will be downloaded from official repository.

    Steps involved to Install Kubernetes Cluster on Ubuntu,

    On All Nodes:
        1. Enable Kubernetes repository on master and all worker nodes
        2. Install all required packages on master and all worker nodes
        3. Start and Enable docker service on master and all worker nodes

    On Master Node:
        1. Initializing and setting up the kubernetes cluster only on Master node
        2. Copy /etc/kubernetes/admin.conf and Change Ownership only on Master node
        3. Install Network add-on to enable the communication between the pods only on Master node

    On Worker Nodes:
        1. Join all worker nodes with kubernetes master node

Before begin, we must update the Ubuntu Repositories and install basic tools like apt-transport-https, curl

    $ sudo apt-get update && sudo apt-get install -y apt-transport-https curl

Once completed, move on to the next step.

 1. Enable Kubernetes repository on master and all worker nodes
    - Add the Kubernetes signing key on all nodes.

    $ wget https://packages.cloud.google.com/apt/doc/apt-key.gpg
    $ sudo mv apt-key.gpg /etc/apt/trusted.gpg.d/

    - Add kubernetes repository on all nodes.

    $ sudo apt-add-repository 'deb http://apt.kubernetes.io/ kubernetes-xenial main'

 2. Install all required packages on master and all worker nodes
    - Use apt-get command to install kubelet, kubeadm, kubectl, docker.io packages.

    $ sudo apt-get update && sudo apt-get install -y kubelet=1.20.0-00 kubeadm=1.20.0-00 kubectl=1.20.0-00 docker.io

 3. Start and Enable docker service on master and all worker nodes

    $ sudo systemctl start docker && sudo systemctl enable docker

 4. Initializing and setting up the kubernetes cluster only on Master node
    - Use “kubeadm” command to initialize the kubernetes cluster along with “apiserver-advertise-address” and “–pod-network-cidr” options. It is used to specify the IP address for kubernetes cluster communication and range of networks for the pods.

    $ sudo kubeadm init --apiserver-advertise-address=172.16.15.11 --pod-network-cidr=10.0.0.0/16

>>>> Your Kubernetes control-plane has initialized successfully!

 5.  (a= run as a regular user or b= run as a root user)

    a. To start using your cluster, you need to run the following as a regular user:

    $ mkdir -p $HOME/.kube
    $ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    $ sudo chown $(id -u):$(id -g) $HOME/.kube/config

    b. Alternatively, if you are the root user, you can run:

    $ export KUBECONFIG=/etc/kubernetes/admin.conf

    You should now deploy a pod network to the cluster.

    Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
    https://kubernetes.io/docs/concepts/cluster-administration/addons/

 6. Install Network add-on to enable the communication between the pods only on Master node
    We have lot of network add-on available to enable the network communication  with different functionality, Here I have used flannel network provider. Flannel is an overlay network provider that can be used with Kubernetes. You can refer more add-on from https://kubernetes.io/docs/concepts/cluster-administration/addons/

    $ sudo kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

    Then you can join any number of worker nodes by running the following on each as root:

    $ sudo kubeadm join 172.16.15.11:6443 --token amz0ii.1kpvdcfjyd5we7xa \
        --discovery-token-ca-cert-hash sha256:0b5631cdb96e4973c997cf49e19f8d4168b8c341e8b7c6f615c628d3e

    Kubernetes cluster initialization is completed, Copy the join token highlighted in yellow color from the “kubeadm init” command output and store it somewhere, it is required while joining the worker nodes.

 7. Copy /etc/kubernetes/admin.conf and Change Ownership only on Master node
    Once kubernetes cluster is initialized, copy “/etc/kubernetes/admin.conf” and change ownership. You could see this same instructions in the output of “kubeadm init” command.

 8. Use “kubectl get nodes” command to ensure the kubernetes master node status is ready. Wait for few minutes until the status of the kubernetes master turn ready state.

    $ kubectl get nodes

 9. You could see the status of indiviual node using below command 

    $ kubectl describe node nodesecond

 10. Once worker nodes are joined with kubernetes master, then verify the list of nodes within the kubernetes cluster. Wait for few minutes until the status of the kubernetes nodes turn ready state.

    $ kubectl get nodes

That's it!

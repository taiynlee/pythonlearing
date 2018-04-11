# Preparation on All Machines
## Change to root
`sudo passwd root`

`sudo su -`

## Disable Swap
`swapoff -a`

```
vi /etc/fstab 
**uncomment the last line about swap**
```

## (1/4) Installing kubeadm on your hosts

## Installing Docker on All Machines
```
apt-get update
apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository \
   "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable"
apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')
```

## Installing kubeadm, kubelet and kubectl on All Machines
```
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

## Configure cgroup driver used by kubelet on Master Node
Make sure that the cgroup driver used by kubelet is the same as the one used by Docker. Verify that your Docker cgroup driver matches the kubelet config:
```
docker info | grep -i cgroup
cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
Manual Add line
`Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"`
To /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
Then restart kubelet:
```
systemctl daemon-reload
systemctl restart kubelet
```

## (2/4) Initializing your master
`kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=<master-ip-address>`

`export KUBECONFIG=/etc/kubernetes/admin.conf`

## (3/4) Installing a pod network on Master Node
Calico

`kubectl apply -f https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/kubeadm/1.7/calico.yaml`

Master Isolation

`kubectl taint nodes --all node-role.kubernetes.io/master-`

## (4/4) Joining cluster from Worker Node

`sudo su -`
```
kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```
# kubeadm
# Setup Three Nodes K8s Cluster with Kubeadm

## Prepare Three Nodes

Create three nodes Centos7 machines through `Vagrant`

```bash
vagrant up
```

Then check all three nodes have installed `kubeadm`, `kubelet` and `kubectl`, and Docker is running.

```bash
➜  kubeadm git:(master) ✗ vagrant status
Current machine states:

k8s-master-centos         running (virtualbox)
k8s-node1-centos          running (virtualbox)
k8s-node2-centos          running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
➜  kubeadm git:(master) ✗ vagrant ssh k8s-master-centos
Last login: Sun Feb 23 19:41:09 2020 from 10.0.2.2
[vagrant@k8s-master-centos ~]$ which kubeadm
/usr/bin/kubeadm
[vagrant@k8s-master-centos ~]$ which kubectl
/usr/bin/kubectl
[vagrant@k8s-master-centos ~]$ which kubelet
/usr/bin/kubelet
[vagrant@k8s-master-centos ~]$ sudo docker version
Client:
 Version:         1.13.1
 API version:     1.26
 Package version: docker-1.13.1-108.git4ef4b30.el7.centos.x86_64
 Go version:      go1.10.3
 Git commit:      4ef4b30/1.13.1
 Built:           Tue Jan 21 17:16:25 2020
 OS/Arch:         linux/amd64

Server:
 Version:         1.13.1
 API version:     1.26 (minimum version 1.12)
 Package version: docker-1.13.1-108.git4ef4b30.el7.centos.x86_64
 Go version:      go1.10.3
 Git commit:      4ef4b30/1.13.1
 Built:           Tue Jan 21 17:16:25 2020
 OS/Arch:         linux/amd64
 Experimental:    false

```

## Configuring Kubernetes Master node

### Start kubelet in three nodes

First, removing the `$KUBELET_NETWORK_ARGS` in `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` in all three nodes and start Kubelet

```bash
sudo vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
sudo systemctl enable kubelet && sudo systemctl start kubelet
```

### kubeadm init on master node

```bash
sudo kubeadm init
sudo kubeadm reset
sudo kubeadm init --pod-network-cidr 172.100.0.0/16 --apiserver-advertise-address 192.168.205.120
```

Output

```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.205.120:6443 --token 8j0n2y.sudhq4e4hehggjl6 \
    --discovery-token-ca-cert-hash sha256:e971554ed8fb754254f19327b8aa7f73931fded8860aa59b4830bc117939367e
```

Please follow the output. and after all is done, we can get all pods running

On master node:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

install network addon

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

Check if all pod are running. then it's OK.

```bash
[vagrant@k8s-master-centos ~]$ kubectl get pod --all-namespaces
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
kube-system   coredns-6955765f44-99nbr                    1/1     Running   0          2m49s
kube-system   coredns-6955765f44-nn5sl                    1/1     Running   0          2m49s
kube-system   etcd-k8s-master-centos                      1/1     Running   0          3m1s
kube-system   kube-apiserver-k8s-master-centos            1/1     Running   0          3m1s
kube-system   kube-controller-manager-k8s-master-centos   1/1     Running   0          3m1s
kube-system   kube-proxy-stzs2                            1/1     Running   0          2m48s
kube-system   kube-scheduler-k8s-master-centos            1/1     Running   0          3m1s
kube-system   weave-net-b9m42                             2/2     Running   0          43s
```

## Join worker node

Please use sudo join

```bash
sudo kubeadm join 192.168.205.120:6443 --token 8j0n2y.sudhq4e4hehggjl6 --discovery-token-ca-cert-hash sha256:e971554ed8fb754254f19327b8aa7f73931fded8860aa59b4830bc117939367e
```

After that, we can get three nodes ouput on master node

```bash
[vagrant@k8s-master-centos ~]$ kubectl get node
NAME                STATUS   ROLES    AGE     VERSION
k8s-master-centos   Ready    master   8m19s   v1.17.3
k8s-node1-centos    Ready    <none>   2m2s    v1.17.3
k8s-node2-centos    Ready    <none>   66s     v1.17.3
```

all pod are ok include flannel

```bash
[vagrant@k8s-master-centos ~]$ kubectl get pod --all-namespaces
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
kube-system   coredns-6955765f44-99nbr                    1/1     Running   0          14m
kube-system   coredns-6955765f44-nn5sl                    1/1     Running   0          14m
kube-system   etcd-k8s-master-centos                      1/1     Running   0          15m
kube-system   kube-apiserver-k8s-master-centos            1/1     Running   0          15m
kube-system   kube-controller-manager-k8s-master-centos   1/1     Running   0          15m
kube-system   kube-proxy-skfgf                            1/1     Running   0          8m51s
kube-system   kube-proxy-stzs2                            1/1     Running   0          14m
kube-system   kube-proxy-wpzqv                            1/1     Running   0          7m55s
kube-system   kube-scheduler-k8s-master-centos            1/1     Running   0          15m
kube-system   weave-net-5bx8c                             2/2     Running   1          7m55s
kube-system   weave-net-b9m42                             2/2     Running   0          12m
kube-system   weave-net-gr6sv                             2/2     Running   0          8m51s
```

## Reference

[https://blog.tekspace.io/setup-kubernetes-cluster-on-centos-7/](https://blog.tekspace.io/setup-kubernetes-cluster-on-centos-7/
)

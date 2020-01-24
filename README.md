# kubernetes sandbox

cross platform(freebsd,lin,win,mac..etc)

~~~~
>vagrant up

vagrant global-status
id       name            provider   state    directory
-----------------------------------------------------------------------------------------------------------
c34c93c  k8s-master01    virtualbox running  C:/multimachine/kubernetes-sandbox-remote
adb4ffe  worker01        virtualbox running  C:/multimachine/kubernetes-sandbox-remote
2e21187  worker02        virtualbox running  C:/multimachine/kubernetes-sandbox-remote
b39b49d  remotecontrol01 virtualbox running  C:/multimachine/kubernetes-sandbox-remote

>vagrant ssh remotecontrol01
sudo ansible-playbook -i /vagrant/kube-cluster/hosts /vagrant/kube-cluster/1_initial.yml
sudo ansible-playbook -i /vagrant/kube-cluster/hosts /vagrant/kube-cluster/2_kube-dependencies.yml
sudo ansible-playbook -i /vagrant/kube-cluster/hosts /vagrant/kube-cluster/3_masters.yml
sudo ansible-playbook -i /vagrant/kube-cluster/hosts /vagrant/kube-cluster/4_workers.yml

vagrant ssh k8s-master01
$ kubectl get nodes
NAME           STATUS   ROLES    AGE   VERSION
k8s-master01   Ready    master   20m   v1.16.0
worker01       Ready    <none>   12m   v1.16.0
worker02       Ready    <none>   12m   v1.16.0

$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-55754f75c-kckwb   1/1     Running   0          19m
kube-system   calico-node-6qhgc                         1/1     Running   7          19m
kube-system   calico-node-8648z                         1/1     Running   0          11m
kube-system   calico-node-m4thf                         1/1     Running   0          11m
kube-system   coredns-5644d7b6d9-2844c                  1/1     Running   0          19m
kube-system   coredns-5644d7b6d9-2dwp2                  1/1     Running   0          19m
kube-system   etcd-k8s-master01                         1/1     Running   0          18m
kube-system   kube-apiserver-k8s-master01               1/1     Running   0          19m
kube-system   kube-controller-manager-k8s-master01      1/1     Running   0          18m
kube-system   kube-proxy-2cwcx                          1/1     Running   0          11m
kube-system   kube-proxy-7xbfz                          1/1     Running   0          11m
kube-system   kube-proxy-dfgxk                          1/1     Running   0          19m
kube-system   kube-scheduler-k8s-master01               1/1     Running   0          18m

vagrant@k8s-master01:~$ docker version
Client: Docker Engine - Community
 Version:           19.03.4
 API version:       1.40
 Go version:        go1.12.10
 Git commit:        9013bf583a
 Built:             Fri Oct 18 15:53:51 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.4
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.10
  Git commit:       9013bf583a
  Built:            Fri Oct 18 15:52:23 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.10
  GitCommit:        b34a5c8af56e510852c35414db4c1f4fa6172339
 runc:
  Version:          1.0.0-rc8+dev
  GitCommit:        3e425f80a8c931f88e6d94a8c831b9d5aa481657
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
~~~~

upgrade kubernetes
~~~~
vagrant ssh k8s-master01
$ kubectl get nodes
NAME           STATUS   ROLES    AGE     VERSION
k8s-master01   Ready    master   10m     v1.17.0
worker01       Ready    <none>   6m54s   v1.17.0
worker02       Ready    <none>   6m58s   v1.17.0


$ apt-cache madison kubelet | more
 kubelet |  1.16.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages

 vagrant@k8s-master01:~$ apt-cache madison kubelet | more
    kubelet |  1.17.2-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubelet |  1.17.1-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubelet |  1.17.0-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubelet |  1.16.6-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubelet |  1.16.5-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages

#kubernetes-sandbox-remote-calico\kube-cluster\2_kube-dependencies.yml
kubernetes_version : "=1.17.0-00"
validated_dockerv: "=5:19.03.4~3-0~ubuntu-xenial"

https://kubernetes.io/docs/setup/release/notes/

~~~~
upgrade calico
~~~~
Installing a pod network add-on
kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml

$ kubectl get pods -n kube-system
$ kubectl -n kube-system logs coredns-5644d7b6d9-cw9bg

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

~~~~

upgrade docker
~~~~  

vagrant@k8s-master01:~$ apt-cache madison docker-ce
 docker-ce | 5:19.03.5~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 5:19.03.4~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 5:19.03.3~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 5:19.03.2~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 5:19.03.1~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages


Update the latest validated version of Docker to 19.03 (#84476, @neolit123)
https://kubernetes.io/docs/setup/release/notes/

On each of your machines, install Docker. Version 19.03.4 is recommended, but 1.13.1, 17.03, 17.06, 17.09, 18.06 and 18.09 are known to work as well. Keep track of the latest verified Docker version in the Kubernetes release notes.
https://kubernetes.io/docs/setup/production-environment/container-runtimes/

[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/

~~~~

Controlling your cluster from machines other than the control-plane node
~~~~
vagrant@k8s-master01:~$ hostnamectl | grep "Operating System"
  Operating System: Ubuntu 19.04
vagrant@worker01:~$ hostnamectl | grep "Operating System"
    Operating System: Ubuntu 16.04.5 LTS
vagrant@worker02:~$ hostnamectl | grep "Operating System"
      Operating System: Ubuntu 18.10


[vagrant@remotecontrol01 ~]$ cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)


@k8s-master01:/etc/kubernetes/admin.conf .
copy the administrator kubeconfig file from your control-plane node to your workstation
scp vagrant@k8s-master01:/~admin.conf .

Install kubectl
[vagrant@remotecontrol01 ~]$ cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
[vagrant@remotecontrol01 ~]$ sudo yum install -y kubectl
[vagrant@remotecontrol01 ~]$ kubectl --kubeconfig ./admin.conf get nodes
NAME           STATUS   ROLES    AGE   VERSION
k8s-master01   Ready    master   38m   v1.15.2
worker01       Ready    <none>   31m   v1.15.2
worker02       Ready    <none>   30m   v1.15.2

~~~~

Init Containers

~~~~
vagrant@k8s-master:~$ kubectl apply -f /vagrant/myapp.yaml
pod/myapp-pod created

vagrant@k8s-master:~$ kubectl get -f /vagrant/myapp.yaml
NAME        READY   STATUS     RESTARTS   AGE
myapp-pod   0/1     Init:0/2   0          12m

# for more details
vagrant@k8s-master:~$ kubectl describe -f /vagrant/myapp.yaml
# Inspect the first init container
vagrant@k8s-master:~$ kubectl logs myapp-pod -c init-myservice
# Inspect the second init container
vagrant@k8s-master:~$ kubectl logs myapp-pod -c init-mydb


vagrant@k8s-master01:~$ kubectl apply -f /vagrant/services.yaml
service/myservice created
service/mydb created
vagrant@k8s-master01:~$ kubectl get -f /vagrant/myapp.yaml
NAME        READY   STATUS    RESTARTS   AGE
myapp-pod   1/1     Running   0          5m52s
vagrant@k8s-master01:~$ kubectl logs myapp-pod -c init-mydb
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      mydb
Address 1: 10.96.186.168 mydb.default.svc.cluster.local
vagrant@k8s-master01:~$ kubectl logs myapp-pod -c init-myservice | more
nslookup: can't resolve 'myservice'
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local


Init Containers
https://kubernetes.io/docs/concepts/workloads/pods/init-containers/
~~~~

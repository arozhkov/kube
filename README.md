
# Terraform
```
  hostname = admin-0
  network.0.address = 147.75.100.97
  network.2.address = 10.80.16.129

  hostname = worker-0
  network.0.address = 147.75.100.131
  network.2.address = 10.80.55.129
    
  hostname = worker-1
  network.0.address = 147.75.100.135
  network.2.address = 10.80.83.1
```

# /etc.hosts
```
10.80.16.129 admin-0
10.80.55.129 worker-0
10.80.83.1 worker-1
```

# Ansible
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://yum.kubernetes.io/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

setenforce 0
yum install -y docker kubelet kubeadm kubectl kubernetes-cni
systemctl enable docker && systemctl start docker
systemctl enable kubelet && systemctl start kubelet
```


# Admin
```
[root@admin-0 ~]# kubeadm init --api-advertise-addresses=10.80.16.129
<master/tokens> generated token: "cac934.ada71a1a128172d3"
<master/pki> created keys and certificates in "/etc/kubernetes/pki"
<util/kubeconfig> created "/etc/kubernetes/kubelet.conf"
<util/kubeconfig> created "/etc/kubernetes/admin.conf"
<master/apiclient> created API client configuration
<master/apiclient> created API client, waiting for the control plane to become ready
<master/apiclient> all control plane components are healthy after 12.890454 seconds
<master/apiclient> waiting for at least one node to register and become ready
<master/apiclient> first node is ready after 4.004322 seconds
<master/discovery> created essential addon: kube-discovery, waiting for it to become ready
<master/discovery> kube-discovery is ready after 2.504190 seconds
<master/addons> created essential addon: kube-proxy
<master/addons> created essential addon: kube-dns

Kubernetes master initialised successfully!

You can now join any number of machines by running the following on each node:

kubeadm join --token cac934.ada71a1a128172d3 10.80.16.129
```

# Node join
```
kubeadm join --token cac934.ada71a1a128172d3 10.80.16.129
```

# CNI
```
[root@admin-0 ~]# kubectl apply -f https://git.io/weave-kube
```

# Ready
```
[root@admin-0 ~]# kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                              READY     STATUS    RESTARTS   AGE       IP             NODE
kube-system   etcd-admin-0                      1/1       Running   0          14m       10.80.16.129   admin-0
kube-system   kube-apiserver-admin-0            1/1       Running   0          14m       10.80.16.129   admin-0
kube-system   kube-controller-manager-admin-0   1/1       Running   0          14m       10.80.16.129   admin-0
kube-system   kube-discovery-982812725-sq6ws    1/1       Running   0          14m       10.80.16.129   admin-0
kube-system   kube-dns-2247936740-19402         2/3       Running   0          14m       10.46.0.1      admin-0
kube-system   kube-proxy-amd64-dtp9v            1/1       Running   0          14m       10.80.16.129   admin-0
kube-system   kube-proxy-amd64-k1z9d            1/1       Running   0          13m       10.80.83.1     worker-1
kube-system   kube-proxy-amd64-ve39y            1/1       Running   0          13m       10.80.55.129   worker-0
kube-system   kube-scheduler-admin-0            1/1       Running   0          14m       10.80.16.129   admin-0
kube-system   weave-net-tayfy                   2/2       Running   0          1m        10.80.16.129   admin-0
kube-system   weave-net-vo21o                   2/2       Running   0          1m        10.80.83.1     worker-1
kube-system   weave-net-y487i                   2/2       Running   0          1m        10.80.55.129   worker-0
```


# Demo
```
kubectl create namespace sock-shop
kubectl apply -n sock-shop -f "https://github.com/microservices-demo/microservices-demo/blob/master/deploy/kubernetes/complete-demo.yaml?raw=true"

http://147.75.100.97:30574
```


# Worker 0
```
[root@worker-0 ~]# ifconfig
datapath: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1410
        inet6 fe80::d835:96ff:fed2:a517  prefixlen 64  scopeid 0x20<link>
        ether da:35:96:d2:a5:17  txqueuelen 0  (Ethernet)
        RX packets 50  bytes 3756 (3.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8  bytes 648 (648.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 0.0.0.0
        ether 02:42:51:e5:bc:56  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether 0c:c4:7a:b5:85:bc  txqueuelen 1000  (Ethernet)
        RX packets 186659  bytes 241720798 (230.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 40318  bytes 6279299 (5.9 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device memory 0xdf120000-df13ffff  

eth1: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 0c:c4:7a:b5:85:bc  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device memory 0xdf100000-df11ffff  

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 380  bytes 41824 (40.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 380  bytes 41824 (40.8 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

team0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 147.75.100.131  netmask 255.255.255.254  broadcast 147.75.100.131
        inet6 2604:1380:2000:2100::3  prefixlen 127  scopeid 0x0<global>
        inet6 fe80::ec4:7aff:feb5:85bc  prefixlen 64  scopeid 0x20<link>
        ether 0c:c4:7a:b5:85:bc  txqueuelen 0  (Ethernet)
        RX packets 38850  bytes 229354864 (218.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 39760  bytes 6242471 (5.9 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

team0:0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.80.55.129  netmask 255.255.255.254  broadcast 10.80.55.129
        ether 0c:c4:7a:b5:85:bc  txqueuelen 0  (Ethernet)

vethwe-bridge: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1410
        inet6 fe80::50bf:e1ff:fe6d:9245  prefixlen 64  scopeid 0x20<link>
        ether 52:bf:e1:6d:92:45  txqueuelen 1000  (Ethernet)
        RX packets 50  bytes 3804 (3.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 17  bytes 1338 (1.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vethwe-datapath: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1410
        inet6 fe80::1459:e7ff:fe5e:a32b  prefixlen 64  scopeid 0x20<link>
        ether 16:59:e7:5e:a3:2b  txqueuelen 1000  (Ethernet)
        RX packets 380  bytes 41824 (40.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 380  bytes 41824 (40.8 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

weave: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1410
        inet 10.40.0.0  netmask 255.240.0.0  broadcast 0.0.0.0
        inet6 fe80::48d5:8fff:fed5:acc1  prefixlen 64  scopeid 0x20<link>
        ether 4a:d5:8f:d5:ac:c1  txqueuelen 0  (Ethernet)
        RX packets 48  bytes 2964 (2.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 9  bytes 690 (690.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0



[root@worker-0 ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         147.75.100.130  0.0.0.0         UG    0      0        0 team0
10.0.0.0        10.80.55.128    255.0.0.0       UG    0      0        0 team0
10.32.0.0       0.0.0.0         255.240.0.0     U     0      0        0 weave
10.80.55.128    0.0.0.0         255.255.255.254 U     0      0        0 team0
10.80.55.128    0.0.0.0         255.255.255.254 U     350    0        0 team0
147.75.100.130  0.0.0.0         255.255.255.254 U     0      0        0 team0
147.75.100.130  0.0.0.0         255.255.255.254 U     350    0        0 team0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
```


# Nginx 
```
kubectl run nginx --image=nginx --port=80 --replicas=2
kubectl expose deployment nginx --type NodePort --name=frontend



[root@admin-0 ~]# kubectl describe pods nginx-3449338310-0ir9g
...

[root@admin-0 ~]# kubectl get pods -o wide
NAME                     READY     STATUS    RESTARTS   AGE       IP          NODE
nginx-3449338310-0ir9g   1/1       Running   0          1m        10.32.0.2   worker-1
nginx-3449338310-e9t75   1/1       Running   0          1m        10.40.0.1   worker-0


[root@admin-0 ~]# kubectl describe services frontend
Name:			frontend
Namespace:		default
Labels:			run=nginx
Selector:		run=nginx
Type:			NodePort
IP:			100.68.4.188
Port:			<unset>	80/TCP
NodePort:		<unset>	31483/TCP
Endpoints:		10.32.0.2:80,10.40.0.1:80
Session Affinity:	None


http://147.75.100.97:31483/
```

# Log into container
```
kubectl exec nginx-3449338310-0ir9g -c nginx -i -t -- bash -il

cat /etc/resolv.conf
ping frontend
timeout 2 bash -c "</dev/tcp/frontend/80"; echo $?  
```

# From Admin server
```
kubectl describe svc frontend
kubectl scale deployment nginx --replicas=0

kubectl describe svc frontend
kubectl scale deployment nginx --replicas=4

kubectl run curl --image=radial/busyboxplus:curl -i --tty
```


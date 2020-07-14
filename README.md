# Install PSO on your Kubernetes (K8's) Cluster

This guild will help you install and test PSO in your K8's cluster

## Prerequisites

- A working K8's cluster with no firewall's blocking ports 
- your api key and mgmt url for Pure Array (nfs mount point for flashblade)


### Clone the Helm Chart
Youll need the repo to get the latest values.yaml for your deployment

```
git clone https://github.com/purestorage/helm-charts.git
```

### Install the latest Helm
You need to download and install the latest version

```
wget https://get.helm.sh/helm-v3.2.0-rc.1-linux-amd64.tar.gz
tar -xvf helm-v3.2.0-rc.1-linux-amd64.tar.gz
cd linux-amd64
sudo cp helm /usr/local/bin
```

### Add the Pure Helm Charts and update your repo

```
helm repo add pure https://purestorage.github.io/helm-charts

helm repo update
```

validate you have the charts 

```
helm search repo pure-csi
NAME         	CHART VERSION	APP VERSION	DESCRIPTION
pure/pure-csi	1.2.0        	1.2.0      	A Helm chart for Pure Service Orchestrator CSI ...
```


### Install PSO 
add your Pure values to the values file

```
arrays:

   FlashArrays:
     - MgmtEndPoint: "10.226.224.120"
       APIToken: "a2bc1b63-a4eb-35ddxxxxxxxxxxxx"
   FlashBlades:
     - MgmtEndPoint: "10.226.224.182"
       APIToken: "T-62824b9d-edd7-41xxxxxxxxxxxx"
       NfsEndPoint: "10.226.224.247"
```

Create a namespace for Pure

```
kubectl create namespace flashy
```

### Install PSO with Helm

```
helm install pure-storage-driver pure/pure-csi -n flashy -f values.yaml
[root@k8svteam01 /]# helm install pure-storage-driver pure/pure-csi -n flashy -f values.yaml
NAME: pure-storage-driver
LAST DEPLOYED: Mon Jul 13 23:25:09 2020
NAMESPACE: flashy
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### Smoke Test
Lets perform a smoke test to validate functionality

```
git clone https://github.com/opslounge/PSO.git
cd PSO/wordpress
[root@k8svteam01 wordpress]# ./make
secret/mysql-pass-7tt4f27774 created
service/wordpress-mysql created
service/wordpress created
deployment.apps/wordpress-mysql created
deployment.apps/wordpress created
persistentvolumeclaim/mysql-pv-claim created
persistentvolumeclaim/wp-pv-claim created
```
You can validate the install by checking on the following

pods are up
```
kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
wordpress-5894fc5d86-z4tcr        1/1     Running   2          24m
wordpress-mysql-56ff98b6b-2grw4   1/1     Running   0          24m
```
validate the pvc is created and bound
NOTE you can copy the pvc string into the array to validate it was created on the array
```
kubectl get pvc
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pv-claim   Bound    pvc-97ad4531-f747-46f5-98d0-382cf56b66f6   20Gi       RWO            pure-file      13s
wp-pv-claim      Bound    pvc-76692b6b-b8e1-4c02-a657-c351180870a9   20Gi       RWO            pure-file      12s
```

Log into a pod to see the mounted volume/filesystem
```
kubectl exec -it wordpress-5894fc5d86-dgrtm /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl kubectl exec [POD] -- [COMMAND] instead.
root@wordpress-5894fc5d86-dgrtm:/var/www/html# df -h
Filesystem                                                    Size  Used Avail Use% Mounted on
overlay                                                        47G  3.1G   44G   7% /
tmpfs                                                          64M     0   64M   0% /dev
tmpfs                                                         7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/mapper/centos-root                                        47G  3.1G   44G   7% /etc/hosts
shm                                                            64M     0   64M   0% /dev/shm
10.226.224.247:/k8s-pvc-da7d62df-4401-4fd9-8283-0aad3c25cf49   20G   19M   20G   1% /var/www/html
tmpfs                                                         7.8G   12K  7.8G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                                                         7.8G     0  7.8G   0% /proc/acpi
tmpfs                                                         7.8G     0  7.8G   0% /proc/scsi
tmpfs                                                         7.8G     0  7.8G   0% /sys/firmware
```

You should also be able to log into the web app and setup wordpress
You will need to identify what node the app is on and what port is being redirected to. 

In the below example we are on node 10.226.226.18

```
[root@k8svteam01 wordpress]# kubectl describe pod wordpress-5894fc5d86-dgrtm
Name:         wordpress-5894fc5d86-dgrtm
Namespace:    default
Priority:     0
Node:         k8svteam03/10.226.226.18
```
Note that node is forwarding to port 31741

```
[root@k8svteam01 wordpress]# kubectl get svc
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes        ClusterIP      10.96.0.1       <none>        443/TCP        67m
wordpress         LoadBalancer   10.96.155.198   <pending>     80:31741/TCP   6m22s
wordpress-mysql   ClusterIP      None            <none>        3306/TCP       6m22s
```

You can open a browser to point to that url and you should see a wordpress setup page.

```
http://10.226.226.18:31741
```

### Remove PSO
You can remove using the following command 
```
helm del pure-storage-driver -n flashy


Authors

* **Andy Parsons** - - [opslounge](https://github.com/opslounge)

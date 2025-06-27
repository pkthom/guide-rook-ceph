# Guide rook ceph

![image](https://github.com/user-attachments/assets/7acc6c81-fedc-4265-bb80-75d3ad223976)

### 1.Adding Raw Disks to Worker Nodes for Ceph

Create EBS Volumes

1. Go to the EC2 Dashboard.
2. In the left-hand menu, choose **Volumes** under **Elastic Block Store**.
3. Click **Create Volume**.
4. Configure the volume:

   * **Size**: Choose a suitable size (e.g., 100 GiB).
   * **Availability Zone**: Select the *same AZ* as the worker node you want to attach this volume to.
   * **Volume Type**: General Purpose SSD (gp3) or other as needed.
5. Click **Create Volume**.

Attach the Volume to Your EC2 Instance

1. After the volume is created, select it in the **Volumes** list.
2. Click **Actions** → **Attach Volume**.
3. In the dialog:

   * **Instance**: Select the worker node instance you want to attach it to.
   * **Device name**: Enter `/dev/sdd`

     * ⚠️ Note: `/dev/sdb` and `/dev/sdc` may not be allowed or may be reserved. Use `/dev/sdd` or later (e.g., `/dev/sde`, `/dev/sdf`) instead.
4. Click **Attach Volume**.

Repeat the steps above for each worker node that will run Ceph OSDs.

```
lsblk -i
```
Example output:
```
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0          7:0    0 27.2M  1 loop /snap/amazon-ssm-agent/11320
loop1          7:1    0 73.9M  1 loop /snap/core22/1981
loop2          7:2    0 50.9M  1 loop /snap/snapd/24505
nvme0n1      259:0    0   30G  0 disk
|-nvme0n1p1  259:1    0   29G  0 part /
|-nvme0n1p14 259:2    0    4M  0 part
|-nvme0n1p15 259:3    0  106M  0 part /boot/efi
`-nvme0n1p16 259:4    0  913M  0 part /boot
nvme1n1      259:5    0   10G  0 disk
```
=> You will see `nvme1n1`


### 2.Install Ceph Operator 

https://rook.io/docs/rook/latest-release/Helm-Charts/operator-chart/

Install
```
helm repo add rook-release https://charts.rook.io/release
helm repo update
helm install --create-namespace --namespace rook-ceph rook-ceph rook-release/rook-ceph -f rook-ceph.yaml
```
`rook-ceph-operator` pod should be running. 
```
kubectl get pods -n rook-ceph
```

### 3.Create a Cephcluster

```
kubectl apply -f cephcluster.yaml
```
⚠️ Replace <NODE_NAME_1>, <NODE_NAME_2>, and <NODE_NAME_3> with your actual Kubernetes node hostnames.

Check the pods:
```
kubectl get pods -n rook-ceph
```
<details><summary>Example output:</summary>

```
NAME                                                              READY   STATUS      RESTARTS   AGE
csi-rbdplugin-6mckc                                               2/2     Running     0          157m
csi-rbdplugin-c5r5b                                               2/2     Running     0          157m
csi-rbdplugin-c9mzj                                               2/2     Running     0          157m
csi-rbdplugin-provisioner-c6585b54f-hqxmm                         5/5     Running     0          157m
csi-rbdplugin-provisioner-c6585b54f-s5tmx                         5/5     Running     0          157m
csi-rbdplugin-t4vvq                                               2/2     Running     0          157m
csi-rbdplugin-trlwl                                               2/2     Running     0          157m
csi-rbdplugin-zp6mj                                               2/2     Running     0          157m
rook-ceph-crashcollector-03778dc7d705851af8c89297c11025c9-4kg6p   1/1     Running     0          149m
rook-ceph-crashcollector-bfc0fe1b36ae0088ea4c7dc2202902bd-v8htb   1/1     Running     0          149m
rook-ceph-crashcollector-d72fc5a18872a06baa3b660d1a4aac38-jzphm   1/1     Running     0          150m
rook-ceph-exporter-03778dc7d705851af8c89297c11025c9-57dfbf6d4p8   1/1     Running     0          149m
rook-ceph-exporter-bfc0fe1b36ae0088ea4c7dc2202902bd-74b4c8jwfnk   1/1     Running     0          149m
rook-ceph-exporter-d72fc5a18872a06baa3b660d1a4aac38-6654b4csbmh   1/1     Running     0          150m
rook-ceph-mgr-a-677475967f-n7gnf                                  1/1     Running     0          150m
rook-ceph-mon-a-c49f7c486-mmphz                                   1/1     Running     0          157m
rook-ceph-mon-b-6f665c875f-zjqxb                                  1/1     Running     0          151m
rook-ceph-mon-c-6d797cd9cf-rn4gq                                  1/1     Running     0          150m
rook-ceph-operator-6bd6ff8dfb-964g5                               1/1     Running     0          158m
rook-ceph-osd-0-5965446966-xhsp2                                  1/1     Running     0          149m
rook-ceph-osd-1-778b955856-7j2dk                                  1/1     Running     0          149m
rook-ceph-osd-2-769d75bf9-mdz95                                   1/1     Running     0          149m
rook-ceph-osd-prepare-03778dc7d705851af8c89297c11025c9-5q6h6      0/1     Completed   0          149m
rook-ceph-osd-prepare-bfc0fe1b36ae0088ea4c7dc2202902bd-w2v45      0/1     Completed   0          149m
rook-ceph-osd-prepare-d72fc5a18872a06baa3b660d1a4aac38-s6v8l      0/1     Completed   0          149m
rook-ceph-tools-7b75b967db-dz6mn                                  1/1     Running     0          138m
```
  
</details>

Check the secrets:
```
kubectl get secret -n rook-ceph | grep rook-csi
```
Check inside the secrets:
```
kubectl get secret rook-csi-rbd-provisioner -n rook-ceph -o yaml
kubectl get secret rook-csi-rbd-node -n rook-ceph -o yaml
```
Check the cephcluster status:
```
kubectl get cephcluster -n rook-ceph
```
Example output:
```
NAME        DATADIRHOSTPATH   MONCOUNT   AGE     PHASE   MESSAGE                        HEALTH      EXTERNAL   FSID
rook-ceph   /var/lib/rook     3          9m21s   Ready   Cluster created successfully   HEALTH_OK              abc
```
=> Make sure the cluster is `Ready` and `HEALTH_OK`

### 4.Create a Cephblockpool

```
kubectl apply -f cephblockpool.yaml
```
Check the cephblockpool:
```
kubectl get cephblockpool -n rook-ceph
```

### 5.Create a Storageclass

```
kubectl apply -f storageclass.yaml
```
```
kubectl get storageclass
```

### 6.Install Toolbox

```
kubectl apply -f https://raw.githubusercontent.com/rook/rook/v1.17.5/deploy/examples/toolbox.yaml
```
```
kubectl get pods -n rook-ceph | grep tool
```

### Node Clean Up 

```
kubectl delete cephblockpool replicapool -n rook-ceph
kubectl delete storageclass rook-ceph-block -n rook-ceph
kubectl delete cephcluster rook-ceph -n rook-ceph
helm uninstall rook-ceph -n rook-ceph

sudo rm -rf /var/lib/rook 
wipefs --all /dev/nvme1n1
sgdisk --zap-all /dev/nvme1n1
dd if=/dev/zero of=/dev/nvme1n1 bs=1M count=100
```

# Install kube-prometheus-stack on Ceph

### 1.Install a NGINX Ingress Controller

https://github.com/pkthom/guide-aws-rke2-ha?tab=readme-ov-file#install-nginx-ingress-controller


### 2.Install kube-prometheus-stack 

https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack

Install:
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install kube-prom-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace -f kube-prom-stack.yaml
```
Check the pods:
```
kubectl get pods -n monitoring
```
<details><summary>Example output:</summary>

```
NAME                                                     READY   STATUS    RESTARTS   AGE
alertmanager-kube-prom-stack-kube-prome-alertmanager-0   2/2     Running   0          8m53s
kube-prom-stack-grafana-86c68bd77d-mdtb2                 3/3     Running   0          8m59s
kube-prom-stack-kube-prome-operator-5f4d478f8b-lgjf8     1/1     Running   0          8m59s
kube-prom-stack-kube-state-metrics-854fb45cfc-dzzfz      1/1     Running   0          8m59s
kube-prom-stack-prometheus-node-exporter-7jcrm           1/1     Running   0          8m59s
kube-prom-stack-prometheus-node-exporter-mmf7n           1/1     Running   0          8m59s
kube-prom-stack-prometheus-node-exporter-n2zbh           1/1     Running   0          8m59s
kube-prom-stack-prometheus-node-exporter-p675c           1/1     Running   0          8m59s
kube-prom-stack-prometheus-node-exporter-xjxvv           1/1     Running   0          8m59s
kube-prom-stack-prometheus-node-exporter-xqj9c           1/1     Running   0          8m59s
prometheus-kube-prom-stack-kube-prome-prometheus-0       2/2     Running   0          8m53s
```
  
</details>

Check the pvc:
```
kubectl get pvc -n monitoring
```
Check the pv:
```
kubectl get pv
```
=> 3 pvc and pv for each grafana, alertmanager, prometheus should be created.

Check the ingress:
```
kubectl get ingress -n monitoring
```
=> 3 ingress for each grafana, alertmanager, prometheus should be created.

### 3.Access Grafana 

[http://test-grafana.com](http://test-grafana.com)

[http://test-alertmanager.com](http://test-alertmanager.com)

[http://test-prometheus.com](http://test-prometheus.com)

Get grafana password
```
kubectl get secret --namespace monitoring kube-prom-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

### Uninstall
```
helm uninstall kube-prom-stack -n monitoring
kubectl delete pvc -n monitoring --all
```

# Create a Test Pod on Ceph

### 1.Create a test Nginx on Ceph
```
kubectl create namespace test
helm install test-nginx ./test-nginx -n test
```
※ I added `test-nginx/templates/pvc.yaml` and necessary settings in `test-nginx/values.yaml`

Check the resources:
```
kubectl get pods -n test
kubectl get ingress -n test
kubectl get pvc -n test
```
[http://chart-example.local/](http://chart-example.local/)

### 2.Create a test busybox on Ceph

Create a pvc:
```
kubectl apply -f busybox-pvc.yaml
kubectl get pvc -n test
```
Create a pod:
```
kubectl apply -f busybox-pod.yaml
kubectl get pods -n test
```

# Limit IOPS + Throughput per Ceph RBD image  

### 1.Create a namespace
```
kubectl create namespace fio
```

### 2.Create a pvc

Create a pvc:
```
kubectl apply -f fio-pvc.yaml
kubectl get pvc -n fio
```
A PVC creates a RBDimage.

List Ceph RBD images:

Go inside Toolbox
```
kubectl exec -it rook-ceph-tools-xxx -n rook-ceph -- sh
```
Run this inside the Toolbox pod:
```
rbd ls -p replicapool
```
Check which Ceph RBD image is for `fio-pvc`
```
kubectl get pv <fio-pvc-name> -o jsonpath='{.spec.csi.volumeHandle}'
```
Example output:
```
0001-0009-rook-ceph-0000000000000002-96c552cd-c0bd-4409-909f-3e0ad5a731e0%
```
=> `csi-vol-96c552cd-c0bd-4409-909f-3e0ad5a731e0` is `fio-pvc`


### 3.Create a pod
```
kubectl apply -f fio-pod.yaml
kubectl get pods -n fio
```

### 4.Monitor metrics 

Go inside the pod:
```
kubectl exec -it fio-test -n fio -- sh
```
Run this inside the pod:
```
fio --name=test --rw=write --bs=1M --size=1G --filename=/data/testfile --direct=1 --ioengine=libaio --iodepth=8 --runtime=300 --time_based
```
⚠️ The fio pod mounts the `fio-pvc` at `/data`, so make sure to write under the `/data` directory.

Check the metrics

Run this inside the Toolbox pod:
```
rbd perf image iotop --pool replicapool
```
```
 >WRITES OPS    READS OPS  WRITE BYTES   READ BYTES    WRITE LAT     READ LAT IMAGE
        70/s          0/s    124 MiB/s        0 B/s     96.49 ms      0.00 ns csi-vol-96c552cd-c0bd-4409-909f-3e0ad5a731e0
```
```
  write: IOPS=122, BW=123MiB/s (129MB/s)(4988MiB/40608msec); 0 zone resets
```

### 5.Limit throughput and Monitor Metrics 

Limit throughput to 10MiB
```
sh-5.1$ rbd config image set replicapool/csi-vol-96c552cd-c0bd-4409-909f-3e0ad5a731e0 rbd_qos_bps_limit 10485760
sh-5.1$ rbd config image get replicapool/csi-vol-96c552cd-c0bd-4409-909f-3e0ad5a731e0 rbd_qos_bps_limit
10485760
```
```
 >WRITES OPS    READS OPS  WRITE BYTES   READ BYTES    WRITE LAT     READ LAT IMAGE
        46/s          0/s     10 MiB/s        0 B/s	 9.81 ms      0.00 ns csi-vol-96c552cd-c0bd-4409-909f-3e0ad5a731e0
```
```
  write: IOPS=10, BW=10.3MiB/s (10.8MB/s)(279MiB/27055msec); 0 zone resets
```

### 6.Limit IOPS and Monitor Metrics

Remove the throughput restriction
```
sh-5.1$ rbd config image remove replicapool/csi-vol-96c552cd-c0bd-4409-909f-3e0ad5a731e0 rbd_qos_bps_limit
sh-5.1$ rbd config image get replicapool/csi-vol-96c552cd-c0bd-4409-909f-3e0ad5a731e0 rbd_qos_bps_limit
rbd: rbd_qos_bps_limit is not set
sh-5.1$
```
Limit IOPS to 10
```
sh-5.1$ rbd config image set replicapool/csi-vol-96c552cd-c0bd-4409-909f-3e0ad5a731e0 rbd_qos_iops_limit 10
sh-5.1$ rbd config image get replicapool/csi-vol-96c552cd-c0bd-4409-909f-3e0ad5a731e0 rbd_qos_iops_limit
10
```
```
 >WRITES OPS    READS OPS  WRITE BYTES   READ BYTES    WRITE LAT     READ LAT IMAGE
        10/s          0/s    1.2 MiB/s        0 B/s	 7.15 ms      0.00 ns csi-vol-96c552cd-c0bd-4409-909f-3e0ad5a731e0
```

# Send Ceph Metrics to Prometheus

### 1.Check port for `rook-ceph-mgr`

Run this inside the Toolbox pod:
```
ceph mgr services
```
Example output:
```
{

    "dashboard": "http://10.42.2.8:7000/",
    "prometheus": "http://10.42.2.8:9283/"
}
```
```
kubectl get svc -n rook-ceph
```
Example output:
```
NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
rook-ceph-mgr             ClusterIP   10.43.218.135   <none>        9283/TCP            93m
```
=> `rook-ceph-mgr` pod is exposed via port `9283`

### 2.Check prometheus release name

```
kubectl get prometheus -n monitoring -o yaml
```
Example output:
```
release: kube-prom-stack
```

### 3.Create servicemonitor for `rook-ceph-mgr`

```
kubectl apply -f rook-ceph-mgr-servicemonitor.yaml
kubectl get servicemonitor -n rook-ceph
```

### 4.target is created in Prometheus

[http://test-prometheus.com/targets?search=rook-ceph-mgr](http://test-prometheus.com/targets?search=rook-ceph-mgr)

=> target `rook-ceph-mgr` is created.

You can now get Ceph metrics 

<img width="1904" alt="image" src="https://github.com/user-attachments/assets/2aecfdc0-47cb-4eee-a6be-b5c319118bcd" />



# local-k8s

Step 1: create kubeadm config file
```
apiVersion: kubeadm.k8s.io/v1beta1
kind: InitConfiguration
api:
  advertiseAddress: #privateip#
networking:
  podSubnet: "10.32.0.0/12" # default Weave Net IP range
apiServerExtraArgs:
  service-node-port-range: 80-32767
  feature-gates: "PersistentLocalVolumes=true,VolumeScheduling=true,MountPropagation=true"
controllerManagerExtraArgs:
  feature-gates: "PersistentLocalVolumes=true,VolumeScheduling=true,MountPropagation=true"
schedulerExtraArgs:
  feature-gates: "PersistentLocalVolumes=true,VolumeScheduling=true,MountPropagation=true"
```

Step 2: Create Kubeadm cluster

```
sudo kubeadm init --config kubeadm-config.yaml
```

Step 3: Install CNI

```
kubectl apply -f https://docs.projectcalico.org/v3.9/manifests/calico.yaml
```

Step 4: Taint if a single node [Optional]

```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

Step 5: Create local storageclass

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-ssd
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

Step 6: Create local persistent volume

```
sudo mkdir -p /mnt/data
apiVersion: v1
kind: PersistentVolume
metadata:
  name: podinfo-vol-0
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
          - key: role
            operator: In
            values:
              - local-ssd
  storageClassName: local-ssd
  local:
    path: /mnt/data
```

Step 7: Install helm

```
wget https://get.helm.sh/helm-v3.0.3-linux-amd64.tar.gz
tar -zxvf helm-v3.0.3-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/
sudo mv linux-amd64/helm /usr/local/bin/
rm -rf linux-amd64/helm
```

Step 8: Install consul
```
git clone --single-branch --branch v0.8.1 https://github.com/hashicorp/consul-helm.git
helm install hashicorp ./consul-helm --values consul-helm/values.yaml
```



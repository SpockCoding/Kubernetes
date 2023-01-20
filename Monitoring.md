### How to install Prometheus on Kubernetes Cluster

* criar o namespace monitoring (ou monitoramento)

```bash=
kubectl create namespace monitoring
```
![image](https://user-images.githubusercontent.com/97816800/213588584-fc9ff7d7-e6fe-4503-84e8-d90a680452a3.png)

* se quiser criar via artefato 

```bash=
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
  ``
* criar o persistent volume

* instalar primeiro o NFS

```bash=
sudo apt install nfs-kernel-server
``

* crie o diretório que será usado no Prometheus

```bash=
sudo mkdir -p /mnt/nfs/promdata
```

* mude o dono do diretório, no caso, para nenhum

```bash=
sudo chown nobody:nogroup /mnt/nfs/promdata
```
* mude as permissões do diretório

```bash=
sudo chmod 777 /mnt/nfs/promdata
```
* crie um arquivo  pv-pvc.yaml (vim pv-pvc.yml)

```bash=
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs-data
  namespace: monitoring
  labels:
    type: nfs
    app: prometheus-deployment
spec:
  storageClassName: managed-nfs
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 192.168.49.1
    path: "/mnt/nfs/promdata"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs-data
  namespace: monitoring
  labels:
    app: prometheus-deployment
spec:
  storageClassName: managed-nfs
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 500Mi
```

* aplique

```bash
kubectl apply -f pv-pvc.yaml
```
![image](https://user-images.githubusercontent.com/97816800/213589289-d6b091f2-47bd-4193-941e-cb58565226e4.png)

* cri um cluster role (vim cluster-role.yml)

```bash=
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs: ["get", "list", "watch"]
```
* aplique

```bash=
kubectl apply -f cluster-role.yaml
```

















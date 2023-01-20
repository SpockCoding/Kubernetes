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

* crie um cluster role (vim cluster-role.yml)

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

![image](https://user-images.githubusercontent.com/97816800/213589581-3c701850-a206-4e8a-a46c-ec48ac2b9007.png)

* crie uma service account (vim service-account.yml)

```bash=
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: monitoring
```

* aplique

```bash=
kubectl apply -f service-account.yaml
```

![image](https://user-images.githubusercontent.com/97816800/213589774-860382ce-322c-469c-a6ea-d9822d8e6b2d.png)

* crie um cluster role binding (vim cluster-role-binding.yml)

```bash=
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: monitoring
```

* aplique

```bash=
kubectl apply -f cluster-role-binding.yaml
``

![image](https://user-images.githubusercontent.com/97816800/213589924-850e35f4-aabd-4115-af56-5f445aab2c73.png)






































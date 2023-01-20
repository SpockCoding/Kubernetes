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















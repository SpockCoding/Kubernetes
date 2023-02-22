## Tutorial sem muita enrolação, Kubernetes no Centos 7, com a aplicação para fins didáticos. 

### Será um Cluster com 1 Master e 2 Workers

|    Hostname     |      IP      |    Function   |    vCPU   |  Memory  |   Disk   |
|-----------------|--------------|---------------|-----------|----------|----------|
| Master  | 192.168.1.30 | Control Plane |     2     |   4GB    |   20GB   |
|     Worker-1    | 192.168.1.60 |    Worker     |     2     |   4GB    |   30GB   |
|     Worker-2    | 192.168.1.70 |    Worker     |     2     |   4GB    |   30GB   |

### Configure o Hostname e o arquivo Hosts

```bash=
hostnamectl set-hostname master.local
hostnamectl set-hostname worker-1.local
hostnamectl set-hostname worker-2.local
```
```bash=
cat <<EOF>> /etc/hosts
192.168.1.30 master.local   master
192.168.1.60 worker-1.local worker-1
192.168.1.70 worker-2.local worker-2
EOF
``` 

### Desabilite O SELinux e Ajuste as regras de Firewall e faça um Update no Iptables

```bash=
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

reboot
```
```bash=
sudo firewall-cmd --permanent --add-port=6443/tcp
sudo firewall-cmd --permanent --add-port=2379-2380/tcp
sudo firewall-cmd --permanent --add-port=10250/tcp
sudo firewall-cmd --permanent --add-port=10251/tcp
sudo firewall-cmd --permanent --add-port=10252/tcp
sudo firewall-cmd --permanent --add-port=10255/tcp
sudo firewall-cmd --reload
```
```bash=
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system
```

### Desabilite a memória Swap

```bash=
sudo sed -i '/swap/d' /etc/fstab
sudo swapoff -a
systemctl daemon-reload
```



### Adicione os repositórios necessários

```bash=
sudo yum upgrade -y

sudo yum install -y yum-utils

sudo yum install -y dnf

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

```bash=
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

### Agora vamos na vera, Docker, Containerd, Kubernetes

```bash=
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

* Esta parte é essencial pq se não comentar esta linha irá dar um erro dizendo "container runtime is not working"

```bash=
vim /etc/containerd/config.toml

#disabled_plugins = ["cri"]
```
```bash=
sudo systemctl enable containerd
sudo systemctl start containerd
sudo systemctl enable docker
sudo systemctl start docker
```

```bash=
sudo yum install -y kubelet kubeadm kubectl
```
```bash=
sudo systemctl enable kubelet
sudo systemctl start kubelet
```

### Por firm, vamos subir o cluster, primeiro o nó Master depois os nós workers.

```bash=
sudo kubeadm init --control-plane-endpoint="192.168.1.30:6443" --pod-network-cidr="10.244.0.0/16"
```

* Tudo estando certo irá aparecer conforme abaixo e as partes grifadas serão utilizadas posteriormente. Então copie e guarde.

![image](https://user-images.githubusercontent.com/97816800/220653405-9ab67f27-5a74-4a01-9f35-fc5f94103378.png)

### Preparação do Kubernetes para receber o node, ajustes de config. do HOME das configurações e a instalação do CNI (Container Network Interface) Weave

```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
```bash=
export kubever=$(kubectl version | base64 | tr -d '\n')
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"
```

### Adicionar os nós Workers ao Cluster

* O token que copiamos para adicionar os Workers Nodes ao cluster iremos utilizar agora. 

```bash=
kubeadm join 192.168.1.30:6443 --token 5yhjnt.g3j0fnacrh3wspov \
        --discovery-token-ca-cert-hash sha256:d3ebb4bdd99009977e7d35ea6ab98ef5b608bab522e8bb2736105849d7ae44b7
```

### Testando

```bash=
kubectl get nodes -o wide
```

![image](https://user-images.githubusercontent.com/97816800/220674115-6db31aa8-e7b5-43fc-8c2c-69bcc0d0137e.png)

### Agora vamos pra perfumaria, ajustar o Vim para 2 espaços ao apertar Tab (importante para criar .yaml), criar Alias para o Kubectl, e o Autocomplete do Kubernetes. 

* Vim

```bash=
vim ~/.vimrc

set number relativenumber
set tabstop=2
set softtabstop=2
set expandtab
set shiftwidth=2
```

* Variável "do" para abreviar o "-o yaml --dry-run=client" quando for criar um arquivo .yaml de um POD
```bash=
export do="-o yaml --dry-run=client"
```

* Exemplo prático:
```bash=
kubectl run  nginx --image=nginx $do > pod.yaml
```
![image](https://user-images.githubusercontent.com/97816800/220683157-3600b556-1b7e-49a6-aead-07a445940f22.png)

* Autocomplete
```bash=
dnf install -y bash-completion
```

```bash=
echo "source <(kubectl completion bash)" >> ~/.bashrc
source ~/.bashrc
```

* Alias
```bash=
echo "alias k=kubectl" >> ~/.bashrc
echo "complete -o default -F __start_kubectl k" >> ~/.bashrc
```

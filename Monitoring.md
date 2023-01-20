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
![image](https://user-images.githubusercontent.com/97816800/213589957-8c1c1e90-b760-4d7e-9f25-11f78bfb28c8.png)

* crie um config map (vim configmap.yml)

```bash=
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval:     15s
      evaluation_interval: 15s
    alerting:
      alertmanagers:
      - static_configs:
        - targets:
    rule_files:
      # - "example-file.yml"
    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
        - targets: ['localhost:9090']
    scrape_configs:
      - job_name: 'kubelet'
        kubernetes_sd_configs:
        - role: node
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
          insecure_skip_verify: true  # Required with Minikube.
      - job_name: 'cadvisor'
        kubernetes_sd_configs:
        - role: node
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
          insecure_skip_verify: true  # Required with Minikube.
        metrics_path: /metrics/cadvisor
      - job_name: 'k8apiserver'
        kubernetes_sd_configs:
        - role: endpoints
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
          insecure_skip_verify: true  # Required if using Minikube.
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
          action: keep
          regex: default;kubernetes;https
      - job_name: 'k8services'
        kubernetes_sd_configs:
        - role: endpoints
        relabel_configs:
        - source_labels:
            - __meta_kubernetes_namespace
            - __meta_kubernetes_service_name
          action: drop
          regex: default;kubernetes
        - source_labels:
            - __meta_kubernetes_namespace
          regex: default
          action: keep
        - source_labels: [__meta_kubernetes_service_name]
          target_label: job
         - job_name: 'k8pods'
        kubernetes_sd_configs:
        - role: pod
        relabel_configs:
        - source_labels: [__meta_kubernetes_pod_container_port_name]
          regex: metrics
          action: keep
        - source_labels: [__meta_kubernetes_pod_container_name]
          target_label: job
 ```
 
 * aplique

```bash=
kubectl apply -f configmap.yaml
```

![image](https://user-images.githubusercontent.com/97816800/213590127-094643bf-7071-4cf6-a858-431f178d2fc9.png)

* crie um deploymento pro prometheus (vim deployment.yml)

```bash=
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
  labels:
    app: prometheus
spec:
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9090"
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus
        args:
          - '--storage.tsdb.retention=6h'
          - '--storage.tsdb.path=/prometheus'
          - '--config.file=/etc/prometheus/prometheus.yml'
        ports:
        - name: web
          containerPort: 9090
        volumeMounts:
        - name: prometheus-config-volume
          mountPath: /etc/prometheus
        - name: prometheus-storage-volume
          mountPath: /prometheus
      restartPolicy: Always
      volumes:
      - name: prometheus-config-volume
        configMap:
            defaultMode: 420
            name: prometheus-config
      - name: prometheus-storage-volume
        persistentVolumeClaim:
            claimName: pvc-nfs-data
  ```
  
  * aplique

```bash=
kubectl apply -f deployment.yaml
```

![image](https://user-images.githubusercontent.com/97816800/213590311-d0ae54a0-e1c2-46cc-a03c-8d28e4da6fff.png)

* crie um serviço que irá expor o prometheus (vim service.yml)

```bash=
apiVersion: v1
kind: Service
metadata:
    name: prometheus-service
    namespace: monitoring
    annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port:   '9090'
spec:
    selector:
        app: prometheus
    type: NodePort
    ports:
    - port: 8080
      targetPort: 9090
      nodePort: 30909
```

* aplique

```bash=
kubectl apply -f service.yaml
```

![image](https://user-images.githubusercontent.com/97816800/213590429-53504e98-a81b-4ed8-83bd-fbef82e6f0b2.png)


* verifique tudo o que fizemos

```bash=
kubectl get all -n monitoring
```

![image](https://user-images.githubusercontent.com/97816800/213590616-d243dca5-bda6-4283-a528-61e4c056f943.png)






































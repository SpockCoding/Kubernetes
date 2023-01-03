### K8S Cluster's Structure


|    Hostname     |      IP      |    Function   |    vCPU   |  Memory  |   Disk   |
|-----------------|--------------|---------------|-----------|----------|----------|
|     Haproxy     | 192.168.1.20 |    HAproxy    |     2     |   4GB    |   20GB   |
| ControlPlane-1  | 192.168.1.30 | Control Plane |     2     |   4GB    |   20GB   |
| ControlPlane-2  | 192.168.1.40 | Control Plane |     2     |   4GB    |   20GB   |
| ControlPlane-3  | 192.168.1.50 | Control Plane |     2     |   4GB    |   20GB   |
|     Worker-1    | 192.168.1.60 |    Worker     |     2     |   4GB    |   20GB   |
|     Worker-2    | 192.168.1.70 |    Worker     |     2     |   4GB    |   20GB   |

### Domain

*  Domain Name: Enterprise.local

### Monitoring 

* [Prometheus](https://prometheus.io) 
* [Grafana](https://grafana.com)

### Data Store 

* [LongHorn](https://longhorn.io)

### Cluster Managing

* [Rancher](https://www.rancher.com)

### Load Balancer 

* [MetalLB](https://metallb.universe.tf)

### Application Manager

* [Helm](https://helm.sh)

### CI/ CD

* [GitHub Actions](https://github.com/features/actions), [GitLab CI](https://docs.gitlab.com/ee/ci/), [CircleCI](https://circleci.com), [ArgoCD](https://argoproj.github.io/cd/)

### IaC
* [Terraform](https://argoproj.github.io/cd/)

*“Eis que estou convosco todos os dias, até o fim dos tempos.” (Mt 28,20)*

*"I am with you always, even unto the end of the world. Amen." (Mt28,20)*

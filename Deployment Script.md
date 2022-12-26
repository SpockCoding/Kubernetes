########## K8S Cluster's Structure ###############


|      Host       |      IP      |    Function   |    vCPU   |  Memory  |   Disk   |
|-----------------|--------------|---------------|-----------|----------|----------|
| Controlplane-1  | 192.168.1.30 | Control Plane |     2     |   4GB    |   20GB   |
| Controlplane-2  | 192.168.1.40 | Control Plane |     2     |   4GB    |   20GB   |
|      Node-1     | 192.168.1.50 |    Worker     |     2     |   4GB    |   20GB   |
|      Node-2     | 192.168.1.60 |    Worker     |     2     |   4GB    |   20GB   |

##################################################

################### Monitoring ###################

1 - Prometheus
2 - Grafana

################## Data Store ####################

1 - LongHorn

############### Cluster Managing #################

1 - Rancher

################# Load Balancer ##################

1 - MetalLB

################ Application Manager #############

1 - Helm

to be continued
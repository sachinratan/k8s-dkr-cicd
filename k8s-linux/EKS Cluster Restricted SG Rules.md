## EKS cluster and node security group rule for restricted traffic

### Cluster Security Group:
```
- Inbound: Allow TCP port 443 (HTTPS) from the worker node security group
- Outbound: Allow TCP port 10250 (kubelet) to the worker node security group
```
### Worker Node Security Group:
```
- Inbound: Allow TCP port 10250 (kubelet) from the cluster security group
- Outbound: Allow TCP port 443 (HTTPS) to the cluster security group
- Outbound: Allow TCP port 443 (HTTPS) from the AWS VPC endpoint security group (optional)
- Outbound: Allow TCP port 443 (HTTPS) from the AWS managed prefix list for S3 endpoint (optional - Might be needed when ECR images fail to unpack)
```
### AWS VPC Endpoint Security Group:
```
- Inbound: Allow TCP port 443 (HTTPS) from the VPC CIDR/worker node security group
- Outbound: Allow TCP port 443 (HTTPS) from the VPC CIDR/worker node security group
```

### To get the worker node to join the cluster with minimal security group inbound and outbound rules configurations
- Cluster Security Group:
```
- Inbound: Allow TCP port 443 (HTTPS) from the worker node security group
- Outbound: Allow TCP port 10250 (kubelet) to the worker node security group
```
- Worker Node Security Group:
```
- Inbound: Allow TCP port 10250 (kubelet) from the cluster security group
- Outbound: Allow TCP port 443 (HTTPS) to the cluster security group
```

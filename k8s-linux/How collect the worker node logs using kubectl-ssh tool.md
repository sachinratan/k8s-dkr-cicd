## The guide will help you to collect the worker node logs using kubectl-ssh tool:

#### Install the kubectl-ssh executable on terminal.
```
$ curl -O https://raw.githubusercontent.com/luksa/kubectl-plugins/master/kubectl-ssh chmod +x kubectl-sshmv kubectl-ssh /usr/local/bin/
```
#### Login to the worker node instance.
```
$ kubectl ssh node <nodeName>
```

Example events:
```
$ kubectl ssh node ip-192-xx-xx-55.ec2.internal
Created pod/ssh-node-k9zvm
Waiting for container to start...
If you don't see a command prompt, try pressing enter.
[root@ip-192-xx-xx-55 /]# ls
```
- Note: Please make a note of the pod name that will be needed in step 4 for copying the logs on the local system.

#### Download the EKS log collector script on worker node shell.
```
$ curl -O https://raw.githubusercontent.com/awslabs/amazon-eks-ami/master/log-collector-script/linux/eks-log-collector.shsudo bash eks-log-collector.sh
```

#### Use following command to copy the logs from kubectl-ssh tool pod to local system directory.
```
$ kubectl cp default/<POD Name>:host/var/log/eks_i-xxxxxxxxxxxxxxxxx_YYYY-MM-DD_HHMM-UTC_0.7.5.tar.gz <Local Path>/eks_i-xxxxxxxxxxxxxxxxx_YYYY-MM-DD_HHMM-UTC_0.7.5.tar.gz
```

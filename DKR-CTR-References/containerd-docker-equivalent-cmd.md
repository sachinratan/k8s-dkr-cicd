# This is guide provides the containerd docker equivalent commands.

#### To pull images from private repository with authentications:
```
$ sudo ctr image pull -u AWS:`aws ecr get-login-password --region us-east-1` <AWS_ACC_NO>.dkr.ecr.us-east-1.amazonaws.com/xacctest:latest
```

#### To pull the public images:
```
$ sudo ctr images pull docker.io/library/python:3
```
OR
```
ctr image pull docker.io/library/nginx:latest
```
#### To list the images
```
$ sudo ctr -n k8s.io images ls
```
OR
```
ctr images ls
```
OR
```
ctr -n k8s.io containers list
```

#### To list namespace specific containers command
```
$ sudo ctr -n k8s.io containers list
```
#### To list available namespaces run
```
$ sudo ctr ns ls
```

#### To do container inspect using ctr command
```
$ sudo ctr -n k8s.io containers info 16d5427d67f2031605aa538aa15114b6ecb6db2b0c0b21dfda0550481dc5c859
```
#### Other useful
```
1. get the container ID from list of containers
2. check if the container has a task associated with it (not all containers have a task associated. For such containers nerdctl or crictl might need to be used to exec)
ctr -n k8s.io tasks ls
3. Exec into container using ID
ctr -n k8s.io tasks exec--exec-id <arbitrary string to associate to this task> <container ID> /bin/sh
```

Reference:
- https://github.com/projectatomic/containerd/blob/master/docs/cli.md
- https://platform9.com/docs/kubernetes/containerd-commands-and-info

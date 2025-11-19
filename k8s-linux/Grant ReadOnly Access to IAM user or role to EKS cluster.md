## Grant ReadOnly Access to IAM user or role to EKS cluster

#### Create the ClusterRoleBinding with role reference to Kuberenetes default ClusterRole "view". 
```
$ kubectl apply -f - <<EOF
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: eksreadonlyrole
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: view
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: eksreadonlyrole
EOF
```
#### Next, create the IAM identity mapping for IAM user/role pointing out to ClusterRoleBinding username while using following eksctl command.
```
 $ eksctl create iamidentitymapping --cluster <clustername> --arn arn:aws:iam::<accountid>:user/iamuser --group a-pod-read-only --username eksreadonlyrole
```

#### To validate the IAM user access privileges we retrieved the IAM role access key, secret access key and token credentials and exported them on terminal.
- To assume IAM role
```
$ aws sts assume-role --role-arn "arn:aws:iam::123456789012:role/example-role" --role-session-name AWSCLI-Session
```

- Export the IAM role access key, secret access key and token credentials
```
$ export AWS_ACCESS_KEY_ID=RoleAccessKeyID
$ export AWS_SECRET_ACCESS_KEY=RoleSecretKey
$ export AWS_SESSION_TOKEN=RoleSessionToken
```
- To verify the configured IAM user credentials.
```
$ aws sts get-caller-identity
```

#### Moving a step ahead, we ran following kubectl command to validate the IAM user access privileges to EKS cluster resources.
```
$ kubectl auth can-i delete namespace      --> To verify the delete access to the kubernetes object

$ kubectl get pod -n <namespace>            --> To verify the read access to the kubernetes object
```

Note: For more information refer articles [rePost article](https://repost.aws/knowledge-center/iam-assume-role-cli), [How To give read only access of resources in EKS Cluster using RBAC Roles to IAM user](https://medium.com/@srminhas01/how-to-give-read-only-access-of-resources-in-eks-cluster-using-rbac-roles-to-iam-user-969acf9b619e) and [How to create Read Only User in EKS Cluster](https://sujitpatel.in/article/how-to-create-read-only-user-in-eks-cluster/).

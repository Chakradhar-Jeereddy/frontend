- We need to send frontend to ingress controller

Ingress
==
- ALB, Listener with certificate
- Created a rule
- Target group
- Added pod's IP to the target group.

***Instead providing access to the resources of EKS cluster for creating external resources,
We will create the loadbalancer, target group, rules & listeners, no need of ingress resource***

***Aws target group terraform***



browse https://dev.chakra86.shop/
curl -X POST -H 'Content-type: application/json' --data '{"text":"Hello, World!"}' 

Install eksctl
==
```
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=$(uname -m | sed 's/x86_64/amd64/' | sed 's/aarch64/arm64/')
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
```


***Create AWS Load Balancer Controller***

```
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster roboshop-dev \
    --approve
	
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.17.0/docs/install/iam_policy.json


aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
	
eksctl create iamserviceaccount \
--cluster=roboshop-dev \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::160885265516:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region us-east-1 \
--approve

helm repo add eks https://aws.github.io/eks-charts

helm upgrade --install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=roboshop-dev \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set vpcId=vpc-001cfbf13c0918f1a
```

Add target group binding (no ingress)
==



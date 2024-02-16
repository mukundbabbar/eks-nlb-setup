# eks-nlb-setup

PREREQ
----
EKSCTL, AWSCLI SETUP

UI
----
//Create VPC with 2 public and 2 private subnet, set auto assign public ip on public subnets

EKSCTL
-------
//Create K8s cluster using above networks
//update public and private subnets and region details in eks-cluster.yaml file
```
eksctl create cluster -f eks-cluster.yaml 
```

K8S ADD CONTROLLER
-----
//deploy sample app
```
k apply -f web-frontend.yaml
```

//deploy lb controller
```
helm install aws-load-balancer-controller \
  eks/aws-load-balancer-controller \
  --namespace kube-system \
  --set clusterName=eks-demo \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

MODIFY POLICY (Known issue)
------
//if error in nlb svc logs/describe command, modify the logged iam policy and remove extra permissions
///////NOTE - Policy bug fix - add policy rule conditions
```
 {
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:AddTags"
            ],
            "Resource": [
                "arn:aws:elasticloadbalancing:*:*:targetgroup/*/*",
                "arn:aws:elasticloadbalancing:*:*:loadbalancer/net/*/*",
                "arn:aws:elasticloadbalancing:*:*:loadbalancer/app/*/*"
            ],
            "Condition": {
                "StringEquals": {
                    "elasticloadbalancing:CreateAction": [
                        "CreateTargetGroup",
                        "CreateLoadBalancer"
                    ]
                },
                "Null": {
                    "aws:RequestTag/elbv2.k8s.aws/cluster": "false"
                }
            }
        }
```
///////


//modify loadbalancer config files with public subnets
```
k apply -f loadbalancer.yaml
k apply -f otel-loadbalancer.yaml
```
//check svc if external ip is assigned
```
k get svc
```

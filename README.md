# eks-nlb-setup


UI
----
//Create VPC with 2 public and 2 private subnet, set auto assign public ip on public subnets

PREREQ
----
EKSCTL, AWSCLI SETUP

EKSCTL
-------
//Create K8s cluster using above networks
eksctl create cluster -f eks-cluster.yaml 

K8S ADD CONTROLLER
-----
k apply -f web-frontend.yaml
helm install aws-load-balancer-controller \
  eks/aws-load-balancer-controller \
  --namespace kube-system \
  --set clusterName=eks-demo \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller

MODIFY POLICY
------
//modify iam policy and remove lines 
///////NOTE - Policy bug fix - add policy rule conditions
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
///////


//modify loadbalancer.yaml with public subnets
k apply -f loadbalancer.yaml
k apply -f otel-loadbalancer.yaml
//check svc if external ip is assigned
k get svc

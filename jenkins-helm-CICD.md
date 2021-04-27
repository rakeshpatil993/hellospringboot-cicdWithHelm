install eks cluster
eksctl create cluster --name=eks27apr \
                  --region=us-east-2 \
                  --zones=us-east-2a,us-east-2b \
                  --without-nodegroup 
add associate-iam-oidc-provider 				  
eksctl utils associate-iam-oidc-provider \
    --region us-east-2 \
    --cluster eks27apr \
    --approve

add nodes in eks
eksctl create nodegroup --cluster=eks27apr \
                   --region=us-east-2 \
                   --name=eks27apr-node \
                   --node-type=t2.xlarge \
                   --nodes=2 \
                   --nodes-min=2 \
                   --nodes-max=4 \
                   --node-volume-size=10 \
                   --ssh-access \
                   --ssh-public-key=shital-ohio \
                   --managed \
                   --asg-access \
                   --external-dns-access \
                   --full-ecr-access \
                   --appmesh-access \
                   --alb-ingress-access	

create .kube/config file in jenkins node:

aws eks update-kubeconfig --name eks27apr --region=us-east-2

install helm and kubectl on jenkins machine

configure iam user in jenkins machine

add iam user in the eks cluster
kubectl edit -n kube-system configmap/aws-auth
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  mapAccounts: |
    - "630433761117"
  mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::630433761117:role/eksctl-eks27apr-nodegroup-eks27ap-NodeInstanceRole-1BE4EUFXBY4ED
      username: system:node:{{EC2PrivateDNSName}}
  mapUsers: |
    - userarn: arn:aws:iam::630433761117:user/rakpatil
      username: rakpatil
      groups:
        - system:masters


deploy the job

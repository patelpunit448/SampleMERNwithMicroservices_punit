aws sso login --profile eks-admin

aws sts get-caller-identity --profile eks-admin

eksctl create cluster -f .\eks-cluster.yaml

aws eks update-kubeconfig `
  --region us-east-1 `
  --name my-eks-cluster `
  --profile eks-admin

kubectl get nodes

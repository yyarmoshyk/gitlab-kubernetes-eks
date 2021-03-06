## Description
This repository has a helm chart that can be used to provision Gitlab in EKS.
There are multiple points why I prepared custom chart instead of using the [official one](https://charts.gitlab.io/):
1. I want to use EFS as persistent volume for gitlab and pgsql
2. I want to use NLB or ELB as ingress controller instead of the Nginx
3. I had gitlab running in ECS in FARGATE mode and this is the target to achieve with this chart. This is why I need the PV to be EFS.
4. Gitlab didn't launch from the default chart and required 2 nodes to launch all pods while Community edition can be launced on the single EC2 instance.
## Provision EKS in AWS
The terraform instructions to provision eKs can be found in the [aws-kubernetes-setup-terraform repository](https://github.com/yyarmoshyk/aws-kubernetes-setup-terraform)

## Install aws EFS driver
### EC2+Fargate:
```bash
kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/ecr/?ref=release-1.0"
```
### Fargate:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-efs-csi-driver/master/deploy/kubernetes/base/csidriver.yaml
```
## Install ALB/NLB ingress
TBD
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.8/docs/examples/rbac-role.yaml

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-0.32.0/deploy/static/provider/aws/deploy.yaml



## Install gitlab
```bash
helm upgrade --install gitlab-server ./gitlab-server/ \
  --set efs_id=fs-2552f3a7 \
  --set hosts.domain=gitlab.example.com \
  --set hosts.aws_region=us-east-1\
  --set eks_cluster_name=sample-eks-eks-cluster\
  --set postgresql.install=true \
  --set redis.install=true
```

If you have PGSql and Redis running outside of your EKS cluster:
```bash
  --set postgresql.install=false \
  --set redis.install=false \
  --set postgresql.postgresqlHostName=pgsq-alskdjl.amazon.aws.com \
  --set postgresql.postgresqlUsername=gitlab_user \
  --set postgresql.postgresqlPostgresPassword=super_secret_password \
  --set postgresql.postgresqlDatabase=gitlab_db \
  --set redis.host_name=redis-elasticache-alskdjl.amazon.aws.com
```

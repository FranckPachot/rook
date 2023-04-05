
Note: the Rook operator is deprecated. 

This is just a test to make it work on Kubernetes > v1.22 but it still uses an old image of YugabyteDB and doesn't allow the same customisation as the [Yugabyte operator](https://github.com/yugabyte/yugabyte-operator) or [Helm Chart](https://dev.to/aws-heroes/yugabytedb-on-amazon-eks-3206)

Here is how to give it a try on AWS EKS:

## create an EKS cluster
```
eksctl create cluster --name rook-yugabytedb --version 1.22 --region eu-west-1 --zones eu-west-1a,eu-west-1b,eu-west-1c --nodegroup-name standard-workers --node-type c5.2xlarge --nodes 4 --nodes-min 0 --nodes-max 4 --managed
aws eks update-kubeconfig --region eu-west-1 --name rook-yugabytedb
```

## clone the repo

```
cd
git clone --single-branch --branch release-1.5 https://github.com/franckpachot/rook.git
cd rook/cluster/examples/kubernetes/yugabytedb

```

## Start the cluster
```
kubectl create -f operator.yaml
kubectl -n rook-yugabytedb-system get pods
kubectl create -f cluster.yaml 
kubectl -n rook-yugabytedb get ybclusters.yugabytedb.rook.io
kubectl -n rook-yugabytedb get pods
kubectl -n rook-yugabytedb-system logs -l app=rook-yugabytedb-operator
kubectl -n rook-yugabytedb logs -l app=yb-master-hello-ybdb-cluster
kubectl -n rook-yugabytedb logs -l app=yb-tserver-hello-ybdb-cluster
kubectl -n rook-yugabytedb describe pod yb-master-hello-ybdb-cluster-0
kubectl -n rook-yugabytedb describe sts
kubectl -n rook-yugabytedb exec -it yb-tserver-hello-ybdb-cluster-0 -- /home/yugabyte/bin/ysqlsh -h yb-tserver-hello-ybdb-cluster-0  --echo-queries -c "select * from version()"
```

## Cleanup
```
kubectl delete namespace rook-yugabytedb
kubectl delete namespace rook-yugabytedb-system
kubectl delete customresourcedefinitions.apiextensions.k8s.io ybclusters.yugabytedb.rook.io
kubectl delete clusterroles.rbac.authorization.k8s.io rook-yugabytedb-operator
kubectl delete clusterrolebindings.rbac.authorization.k8s.io rook-yugabytedb-operator
kubectl create -f operator.yaml
```


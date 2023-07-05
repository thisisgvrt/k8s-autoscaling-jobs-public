# k8s-autoscaling-jobs

**Requirement:**
This repo consists of an api-server and background job, which need to run on a horizonantally scalable k8s cluster. The api-server takes an incoming request and creates a pod of the background job which uses a single CPU core for "some" processing. We need to ensure that we have a highly available API for accepting the incoming request. We also need to scale up and scale down the cluster depending on the number of jobs we have pending/processing at any given moment.

Creating cluster
```
$ cd eks-infra
$ terraform plan
$ terraform apply
```

To configure local kubectl with the cluster created with above commands.
```
$ aws eks --region $(terraform output -raw region) update-kubeconfig --name $(terraform output -raw cluster_name)
```

To create the monitoring stack
```
helm install prometheus-community/kube-prometheus-stack \
--create-namespace --namespace prometheus \
--generate-name \
--set prometheus.service.type=LoadBalancer \
--set grafana.service.type=LoadBalancer \
```

Create secret for pulling docker images from github registry.
```
kubectl create secret docker-registry docker-registry-creds --docker-server=https://ghcr.io --docker-username=<username> --docker-password=<token> --docker-email=<email>
```

Install pod autoscaler.
```
$ cd k8s
$ jinja2 --format json autoscaler.yaml ../eks-infra/terraform.tfstate | kubectl apply -f -
```

To schedule pods

```
$ curl -X POST http://$(kubectl get svc --field-selector metadata.name="kubernetes-node-server-replicaset-service" --no-headers | awk '{ print $4 }')/api/job
```


Destroy the created cluster.
```
$ cd eks-infra
$ terraform destroy
```
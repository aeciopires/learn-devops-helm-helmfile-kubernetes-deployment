<!-- TOC -->

- [How to create HELMFILE specification for Grafana deployment by using HELM charts](#how-to-create-helmfile-specification-for-grafana-deployment-by-using-helm-charts)
  - [How to retrieve public IP addresses of our Kubernetes cluster from cmd](#how-to-retrieve-public-ip-addresses-of-our-kubernetes-cluster-from-cmd)
  - [Remove Prometheus and Grafana](#remove-prometheus-and-grafana)

<!-- TOC -->

# How to create HELMFILE specification for Grafana deployment by using HELM charts

The context key word can be retrieved by running  kubectl config view

```bash
kubectl config view

kubectl create serviceaccount --namespace kube-system tiller

kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

helm2 init --service-account tiller --upgrade
```

Please edit file: `~/udemy/learn-devops-helm-helmfile-kubernetes-deployment/local_charts/prometheus_grafana/grafana/values.yaml`

and

Please edit file: `~/udemy/learn-devops-helm-helmfile-kubernetes-deployment/local_charts/prometheus_grafana/prometheus/values.yaml`

and

Please edit file: `~/udemy/learn-devops-helm-helmfile-kubernetes-deployment/local_charts/prometheus_grafana/helmfile_specification_pg.yaml`

Process HELMFILE deployment:

```bash
cd ~/udemy/learn-devops-helm-helmfile-kubernetes-deployment/local_charts/prometheus_grafana/

helmfile -f helmfile_specification_pg.yaml sync

kubectl get pods -n prometheus
```


## How to retrieve public IP addresses of our Kubernetes cluster from cmd

```bash
aws ec2 describe-instances \
  --query "Reservations[*].Instances[*].PublicIpAddress" \
  --output=text
```


## Remove Prometheus and Grafana

Execute commands to remove Prometheus and Grafana with helmfile.

```bash
cd ~/udemy/learn-devops-helm-helmfile-kubernetes-deployment/local_charts/prometheus_grafana/

helmfile -f helmfile_specification_pg.yaml list

helmfile -f helmfile_specification_pg.yaml destroy
```
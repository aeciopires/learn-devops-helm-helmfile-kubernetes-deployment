<!-- TOC -->

- [Install Helm 2](#install-helm-2)
- [Install Helm 3](#install-helm-3)
- [Setup Tiller User Account in your Kubernetes Cluster in AWS](#setup-tiller-user-account-in-your-kubernetes-cluster-in-aws)
- [Run GOGS with Helm](#run-gogs-with-helm)
- [How to understand persistentVolumeClaim and persistentVolumes](#how-to-understand-persistentvolumeclaim-and-persistentvolumes)

<!-- TOC -->

# Install Helm 2

Simple shell function for kops installation in Linux 64 bits.

Helm documentation: https://helm.sh/docs

Copy and paste this code:

```bash
sudo su

HELM_TAR_FILE=helm-v2.16.1-linux-amd64.tar.gz
HELM_URL=https://get.helm.sh
HELM_BIN=helm2
TILLER_BIN=tiller

function install_helm2 {

if [ -z $(which $HELM_BIN) ]; then
    wget ${HELM_URL}/${HELM_TAR_FILE}
    tar -xvzf ${HELM_TAR_FILE}
    chmod +x linux-amd64/helm
    sudo cp linux-amd64/helm /usr/local/bin/$HELM_BIN
    chmod +x linux-amd64/$TILLER_BIN
    sudo cp linux-amd64/$TILLER_BIN /usr/local/bin/$TILLER_BIN
    rm -rf ${HELM_TAR_FILE} linux-amd64
    echo -e "\nwhich ${HELM_BIN}"
    which ${HELM_BIN}
else
    echo "Helm 2 is most likely installed"
fi
}

install_helm2

which helm2

helm2 version

tiller --version

exit
```


# Install Helm 3

Simple shell function for kops installation in Linux 64 bits.

Helm documentation: https://helm.sh/docs

Copy and paste this code:

```bash
sudo su

HELM_TAR_FILE=helm-v3.0.3-linux-amd64.tar.gz
HELM_URL=https://get.helm.sh
HELM_BIN=helm3

function install_helm3 {

if [ -z $(which $HELM_BIN) ]; then
    wget ${HELM_URL}/${HELM_TAR_FILE}
    tar -xvzf ${HELM_TAR_FILE}
    chmod +x linux-amd64/helm
    sudo cp linux-amd64/helm /usr/local/bin/$HELM_BIN
    sudo ln -sfn /usr/local/bin/$HELM_BIN /usr/local/bin/helm
    rm -rf ${HELM_TAR_FILE} linux-amd64
    echo -e "\nwhich ${HELM_BIN}"
    which ${HELM_BIN}
else
    echo "Helm 3 is most likely installed"
fi
}

install_helm3

which helm3

helm3 version

exit
```

# Setup Tiller User Account in your Kubernetes Cluster in AWS

```bash
kubectl create serviceaccount --namespace kube-system tiller

kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

helm2 init --service-account tiller --upgrade
```

# Run GOGS with Helm

Reference: https://github.com/helm/charts

Run GOGS helm deployment for the first time


helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/

`helm install --name <some_name> --values <values_you_edited>.yaml incubator/gogs`


```bash
cd ~/udemy

git clone https://github.com/helm/charts

cd learn-devops-helm-helmfile-kubernetes-deployment/local_installation/gogs

helm2 repo update

helm2 install --name udemy --values values.yaml gogs

kubectl get service,nodes,pods

kubectl describe NAME_POD
```

# How to understand persistentVolumeClaim and persistentVolumes

How to edit persistentVolumeClaim and persistentVolumes on the fly

Source: https://kubernetes.io/docs/tasks/administer-cluster/change-pv-reclaim-policy/


Choose one of your PersistentVolumes and change its reclaim policy:

```bash
kubectl patch pv <your-pv-name> -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
 where <your-pv-name> is the name of your chosen PersistentVolume.
```
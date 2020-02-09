<!-- TOC -->

- [Install Helm 2](#install-helm-2)
- [Install Helm 3](#install-helm-3)
- [Setup Tiller User Account in your Kubernetes Cluster in AWS](#setup-tiller-user-account-in-your-kubernetes-cluster-in-aws)
- [Run GOGS with Helm](#run-gogs-with-helm)
- [How to understand persistentVolumeClaim and persistentVolumes](#how-to-understand-persistentvolumeclaim-and-persistentvolumes)
- [Install Hellfile](#install-hellfile)
- [How to create HELMFILE specification for JENKINS deployment by using HELM charts](#how-to-create-helmfile-specification-for-jenkins-deployment-by-using-helm-charts)
  - [How to retrieve public IP addresses of our Kubernetes cluster from cmd](#how-to-retrieve-public-ip-addresses-of-our-kubernetes-cluster-from-cmd)
  - [Remove Jenkins](#remove-jenkins)

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

helm2 install --name udemy --values values.yaml .

kubectl get service,nodes,pods

kubectl describe NAME_POD

sudo kubectl port-forward -n NAMESPACE NAME_POD LOCALHOST_PORT:NODE_PORT

helm2 delete udemy

helm2 list
```

# How to understand persistentVolumeClaim and persistentVolumes

How to edit persistentVolumeClaim and persistentVolumes on the fly

Source: https://kubernetes.io/docs/tasks/administer-cluster/change-pv-reclaim-policy/


Choose one of your PersistentVolumes and change its reclaim policy:

```bash
kubectl patch pv <your-pv-name> -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
 where <your-pv-name> is the name of your chosen PersistentVolume.
```

# Install Hellfile

Simple shell function for kops installation in Linux 64 bits.

Helmfile documentation: https://github.com/roboll/helmfile
Kustomize documentation: https://github.com/kubernetes-sigs/kustomize

Copy and paste this code:

```bash

sudo su

HELMFILE_VERSION=v0.99.0
HELMFILE_DOWNLOADED_FILENAME=helmfile_linux_amd64
HURL=https://github.com/roboll/helmfile/releases/download
HELMFILE_URL=${HURL}/${HELMFILE_VERSION}/${HELMFILE_DOWNLOADED_FILENAME}
HELMFILE_BIN=helmfile

function install_helmfile {

if [ -z $(which $HELMFILE_BIN) ]; then
    wget ${HELMFILE_URL}
    chmod +x ${HELMFILE_DOWNLOADED_FILENAME}
    sudo mv ${HELMFILE_DOWNLOADED_FILENAME} /usr/local/bin/${HELMFILE_BIN}
    echo -e "\nexecuting: which ${HELMFILE_BIN}"
    which ${HELMFILE_BIN}
else
    echo "Helmfile is most likely installed"
fi
}

install_helmfile

which helmfile

helmfile --version

exit
```


# How to create HELMFILE specification for JENKINS deployment by using HELM charts

The context key word can be retrieved by running  kubectl config view

```bash
kubectl config view

kubectl create serviceaccount --namespace kube-system tiller

kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

helm2 init --service-account tiller --upgrade
```

Please edit file: `~/udemy/learn-devops-helm-helmfile-kubernetes-deployment/local_charts/jenkins/jenkins_udemy_helmfile.yaml`

    # kube-context (--kube-context)
    context: aecio.devopsinuse.com
     
    releases:
      # Published chart example
      # name of this release
      - name: jenkins-course-udemy
      # target namespace
        namespace: default
      # repository/chart` syntax
        chart: stable/jenkins
      # value files (--values)
        values:
          - jenkins/jenkins_custom_values.yaml
        # values (--set)
        set:
          - name: Master.ServiceType
            value: NodePort
          - name: Master.NodePort
            value: 30477
          - name: Persistence.Size
            value: 2Gi
          - name: Master.AdminPassword
            value: adminadmin
          - name: rbac.install
            value: true

Process HELMFILE deployment:

```bash
cd ~/udemy/learn-devops-helm-helmfile-kubernetes-deployment/local_charts/

helmfile -f jenkins_udemy_helmfile.yaml sync


sudo kubectl port-forward -n NAMESPACE NAME_POD LOCALHOST_PORT:NODE_PORT


printf $(kubectl get secret --namespace default jenkins-course-udemy -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo

export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services jenkins-course-udemy)

export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")

echo http://$NODE_IP:$NODE_PORT/login
```


## How to retrieve public IP addresses of our Kubernetes cluster from cmd

```bash
aws ec2 describe-instances \
  --query "Reservations[*].Instances[*].PublicIpAddress" \
  --output=text
```

Simple Jenkins job shell script:

```bash
HELM_CHARTS=$(ls -al stable | wc -l)

echo -e "You have just cloned ${HELM_CHARTS} number of HELM CHARTS from github.com"
```

## Remove Jenkins

Execute commands to remove Jenkins with helmfile.

```bash
cd ~/udemy/learn-devops-helm-helmfile-kubernetes-deployment/local_charts/

helmfile -f jenkins_udemy_helmfile.yaml list

helmfile -f jenkins_udemy_helmfile.yaml destroy
```
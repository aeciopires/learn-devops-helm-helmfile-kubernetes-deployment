<!-- TOC -->

- [AWS Regions and Availability Zones](#aws-regions-and-availability-zones)
- [Configure Access Account AWS](#configure-access-account-aws)
- [Note 1:](#note-1)
- [Install Kops](#install-kops)
- [Install Terraform](#install-terraform)
- [Install Kubectl](#install-kubectl)
- [Management Kubernetes Cluster](#management-kubernetes-cluster)


<!-- TOC -->

# Configure Access Account AWS


You will need to create an Amazon AWS account. Create a 'Free Tier' account at Amazon https://aws.amazon.com/ follow the instructions on the pages: https://docs.aws.amazon.com/chime/latest/ag/aws-account.html and https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier-limits.html. When creating the account you will need to register a credit card, but since you will create instances using the features offered by the 'Free Tier' plan, nothing will be charged if you do not exceed the limit for the use of the features and time offered and described in the previous link.

After creating the account in AWS, access the Amazon CLI interface at: https://aws.amazon.com/cli/

Click on the username (upper right corner) and choose the "Security Credentials" option. Then click on the "Access Key and Secret Access Key" option and click the "New Access Key" button to create and view the ID and Secret of the key, as shown below (https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html). The Access Key and Secret Key shown below are for illustration only. They are invalid and you need to exchange for the actual data generated for your account.

```bash
Access Key ID: YOUR_ACCESS_KEY_HERE
Secret Access Key: YOUR_SECRET_ACCESS_KEY_HERE
```
    
Create the directory below.

```bash
mkdir -p /home/USERNAME/.aws/
touch /home/USERNAME/.aws/credentials
```
    
Access `/home/USERNAME/.aws/credentials` file and add the following content. The Access Key and Secret Key shown below are for illustration only. They are invalid and you need to exchange for the actual data generated for your account.

```bash
[default]
aws_access_key_id = YOUR_ACCESS_KEY_HERE
aws_secret_access_key = YOUR_SECRET_ACCESS_KEY_HERE
```

# AWS Regions and Availability Zones

List of regions and availability zones in AWS.

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html

# Note 1:

I assume that if you are going through this course during several days - You always destroy all resources in AWS  It means that you stop you Kubernetes cluster every time you are not working on it. The easiest way is to do it via terraform 

```bash
mkdir -p ~/udemy

cd ~/udemy
git clone https://github.com/aeciopires/learn-devops-helm-helmfile-kubernetes-deployment

cd learn-devops-helm-helmfile-kubernetes-deployment

cd ~/udemy/terraform_code;
terraform destroy --auto-approve
```

Destroy/delete manually if terraform can't do that:

* VOLUMES
* LoadBalancer/s (if exists)
* RecordSet/s (custom RecordSet/s)
* EC2 instances
* Network resources
* Others

Except:

* S3 bucket   (delete once you do not want to use this free 1 YEAR account anymore, or you are done with this course.)
* Hosted Zone (delete once you do not want to use this free 1 YEAR account anymore, or you are done with this course.)

# Install Kops

Simple shell function for kops installation in Linux 64 bits.

Kubernetes documentation:

https://kubernetes.io/docs/getting-started-guides/kops/ 

Copy and paste this code:

```bash
sudo su

function install_kops {
    if [ -z $(which kops) ]
       then
           curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
           chmod +x kops-linux-amd64
           mv kops-linux-amd64 /usr/local/bin/kops
       else
           echo "kops is most likely installed"
       fi
}
 
install_kops

which kops

kops version

exit
```

# Install Terraform

Simple shell function for Terraform installation version 11 in Linux 64 bits.

Terraform documentation:

https://www.terraform.io/docs/index.html

Copy and paste this code:

```bash
sudo su

TERRAFORM_ZIP_FILE=terraform_0.11.14_linux_amd64.zip
TERRAFORM=https://releases.hashicorp.com/terraform/0.11.14
TERRAFORM_BIN=terraform
 
function install_terraform {
    if [ -z $(which $TERRAFORM_BIN) ]
       then
           wget ${TERRAFORM}/${TERRAFORM_ZIP_FILE}
           unzip ${TERRAFORM_ZIP_FILE}
           sudo mv ${TERRAFORM_BIN} /usr/local/bin/${TERRAFORM_BIN}
           rm -rf ${TERRAFORM_ZIP_FILE}
    else
       echo "Terraform is most likely installed"
    fi
 
}

install_terraform

which terraform

terraform version

exit
```

# Install Kubectl

Simple shell function for Kubectl installation in Linux 64 bits

Kubectl documentation:

https://kubernetes.io/docs/reference/kubectl/overview/

Copy and paste this code:

```bash
sudo su

KUBECTL_BIN=kubectl

function install_kubectl {
    if [ -z $(which $KUBECTL_BIN) ]
       then
           curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/$KUBECTL_BIN
           chmod +x ${KUBECTL_BIN}
           sudo mv ${KUBECTL_BIN} /usr/local/bin/${KUBECTL_BIN}
    else
       echo "Kubectl is most likely installed"
    fi
}

install_kubectl

which kubectl

kubectl version

exit
```

# Management Kubernetes Cluster

How to start Kubernetes cluster by using Kops and Terraform

First create a zone DNS private in Router53 service of AWS with name `devopsinuse.com` or other domain name.

Second create a bucket S3 public with name `aecio.devopsinuse.com` or other domain name.

```bash
SSH_KEYS=~/.ssh/udemy_devopsinuse
 
if [ ! -f "$SSH_KEYS" ]
then
    echo -e "\nCreating SSH keys ..."
    ssh-keygen -t rsa -C "udemy.course" -N '' -f ~/.ssh/udemy_devopsinuse
else
    echo -e "\nSSH keys are already in place!"
fi
 
echo -e "\nCreating kubernetes cluster ...\n"

cd ~/udemy

kops create cluster \
 --name=aecio.devopsinuse.com \
 --state=s3://aecio.devopsinuse.com \
 --authorization RBAC \
 --zones=us-east-2a \
 --node-count=2 \
 --node-size=t2.micro \
 --master-size=t2.micro \
 --master-count=1 \
 --dns private \
 --out=terraform_code \
 --target=terraform \
 --ssh-public-key=~/.ssh/udemy_devopsinuse.pub

cd ~/udemy/terraform_code

terraform init
terraform validate
terraform plan
terraform apply
terraform show
```

Get the public IP of instace `master.aecio.devopsinuse.com` and create a entry in `/etc/hosts` file for public IP with name `api.aecio.devopsinuse.com` and execute the commands to validate cluster and list nodes.

```bash
kops validate cluster --state=s3://aecio.devopsinuse.com

kubectl get nodes --show-labels

ssh -i ~/.ssh/udemy_devopsinuse admin@api.aecio.devopsinuse.com

```

Delete cluster:

```bash
cd ~/udemy/terraform_code

terraform destroy --auto-approve

kops delete cluster \
 --name=aecio.devopsinuse.com \
 --state=s3://aecio.devopsinuse.com \
 --yes
```

Update cluster:

```bash
kops update cluster \
 --name=aecio.devopsinuse.com \
 --state=s3://aecio.devopsinuse.com \
 --out=terraform_code --target=terraform \
 --ssh-public-key=~/.ssh/udemy_devopsinuse.pub
```




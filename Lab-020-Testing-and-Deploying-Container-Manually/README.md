# Lab # 2 : Testing and Deploying the Containerized application manually
In this Lab, we will deploy and test the application manually step by step. This will help to figure out any issues. 

* **Clone Repo**
```
git clone https://github.com/vijay-khanna/Container-tracing-Nodejs-application
```

* **Test and Deploy Docker Image**
edit the account name and container registry variable.
** Need to give the SSM Role to the Container Nodes. to be able to fetch API Secret. 



This lab documentation is made for N.Virginia region (us-east-1). Please make note of this, and change accordingly for your deployment.
Login to the AWS EC2 Console, go to Cloud9 Services. Create a new environment e.g. "Tracing Containerized Nodejs application" <br/>
>Select Environment type : "Create a new instance for environment (EC2)<br/>
>Instance type : t2.micro (1 GiB RAM + 1 vCPU)  <br/>
>Platform : Amazon Linux <br/>
>Cost-saving setting: after 30 minutes <br/>
>Network settings : Select an existing vpc and subnet, or create a new one <br/>
>Review, click Create <br/>

* **IAM AdministratorAccess Role for Cloud9 Instance :**
>Go to IAM Service, create a role <br/>
>Type of trusted entity : **AWS Service** <br/>
>Choose the service that will use this role : **EC2**, 'Click Next:permissions' <br/>
>Select from existing policies: **AdministratorAccess**, 'Next:Tags'  <br/>
>Add-tags (optional) <br/>
>Role Name: **Admin-Role_for_Cloud9_Instance** <br/>
>Open EC2 Service console, select the Cloud9 Instance <br/>
> Actions => Instance Settings => Attach/Replace IAM Role => Select **Admin-Role_for_Cloud9_Instance** => Apply<br/>

> In Cloud9 console => Preferences => AWS Settings => Disable "AWS Managed Temporary Credentials <br/>


* **Install kubectl and ancilliary tools:**
Inside the cloud9 console, 
```
sudo curl --silent --location -o /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/v1.13.7/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl

sudo yum -y install jq gettext

```
Verify the binaries are in path.
```
for command in kubectl jq envsubst
  do
    which $command &>/dev/null && echo "$command in path" || echo "$command NOT FOUND"
  done

```
Remove temporary credendials, and setup environment variables.

```
rm -vf ${HOME}/.aws/credentials

export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')

echo "export ACCOUNT_ID=${ACCOUNT_ID}" >> ~/.bash_profile
echo "export AWS_REGION=${AWS_REGION}" >> ~/.bash_profile
aws configure set default.region ${AWS_REGION}
aws configure get default.region

```
Validate the IAM role. the Output Arn should contain the IAM role created and attached to instance, and the Instance ID
e.g.  "Arn": "arn:aws:sts::1122334455:assumed-role/Admin-Role_for_Cloud9_Instance/i-0abcd1122334455<br/>
```
aws sts get-caller-identity
```

* **Generating ssh key to login to the kubernetes worker node:**
Press enter three times to accept defaults. 
```
ssh-keygen
```
Delete old-key-pair in case if existing with same name
```
aws ec2 delete-key-pair --key-name "eksworkshop"
aws ec2 import-key-pair --key-name "nodejs-container-istio-key" --public-key-material file://~/.ssh/id_rsa.pub
```


* **Download eksctl and deploy EKS Cluster:**
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/latest_release/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv -v /tmp/eksctl /usr/local/bin
```
Confirm the eksctl command works:
```
eksctl version
```
Modify the parameters if required to change the Region, Zone, Worker Node Instance Type, Max/Min Nodes, Public key to ssh to Nodes.
```
eksctl create cluster --version=1.13 --name=nodejs-istio-cluster --nodes=2 --node-ami=auto --region=${AWS_REGION} --zones=${AWS_REGION}a,${AWS_REGION}b  --ssh-public-key nodejs-container-istio-key --nodes-min 2 --nodes-max 3 --node-type m5.large --node-volume-size 50
```

* **Test the cluster:**
```
kubectl get nodes
```
Export the Worker Role Name for use throughout
```
STACK_NAME=$(eksctl get nodegroup --cluster nodejs-istio-cluster -o json | jq -r '.[].StackName')
INSTANCE_PROFILE_ARN=$(aws cloudformation describe-stacks --stack-name $STACK_NAME | jq -r '.Stacks[].Outputs[] | select(.OutputKey=="InstanceProfileARN") | .OutputValue')
ROLE_NAME=$(aws cloudformation describe-stacks --stack-name $STACK_NAME | jq -r '.Stacks[].Outputs[] | select(.OutputKey=="InstanceRoleARN") | .OutputValue' | cut -f2 -d/)
echo "export ROLE_NAME=${ROLE_NAME}" >> ~/.bash_profile
echo "export INSTANCE_PROFILE_ARN=${INSTANCE_PROFILE_ARN}" >> ~/.bash_profile
```
